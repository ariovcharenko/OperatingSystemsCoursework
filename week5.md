Week 5 – Advanced Security & Monitoring Infrastructure
This week I implemented advanced security controls and monitoring scripts for my Ubuntu Server.
Building on the SSH + firewall setup from Week 4, I:
Verified and documented mandatory access control (AppArmor)
Configured automatic security updates
Installed and configured fail2ban for intrusion detection
Wrote a security baseline verification script (security-baseline.sh) on the server
Wrote a remote monitoring script (monitor-server.sh) on the workstation that collects metrics over SSH

5.1 Access Control with AppArmor
Ubuntu uses AppArmor as its mandatory access control system. This adds another layer of security on top of traditional Unix permissions.
5.1.1 Checking AppArmor status
Commands run on the server (via SSH):
# Check if AppArmor service is running
sudo systemctl status apparmor

# List loaded profiles and modes (enforce/complain)
sudo aa-status
<img width="1041" height="286" alt="image" src="https://github.com/user-attachments/assets/21e0fd53-a4f1-4636-a4e9-8a71cf8f0d23" />
<img width="1165" height="826" alt="image" src="https://github.com/user-attachments/assets/0af48988-081d-4c23-9103-f337a592a3e3" />
<img width="1058" height="411" alt="image" src="https://github.com/user-attachments/assets/54918294-6f63-46e3-b950-07f14012d9ca" />

5.1.2 Verifying a profile (example: OpenSSH server)
On Ubuntu, OpenSSH usually has an AppArmor profile:
# Look for sshd AppArmor profile files
ls /etc/apparmor.d/ | grep ssh
<img width="1023" height="138" alt="image" src="https://github.com/user-attachments/assets/5ad64dc7-f3ea-4612-b757-960f8df63902" />

# Show the contents of the sshd profile (read-only for documentation)
sudo sed -n '1,80p' /etc/apparmor.d/usr.sbin.sshd
<img width="676" height="48" alt="image" src="https://github.com/user-attachments/assets/555f9aaf-3d31-430f-91ae-9eb10519fc00" />

Note: My Ubuntu Server image does not include an AppArmor profile for OpenSSH (/etc/apparmor.d/usr.sbin.sshd). This is normal for some minimal Ubuntu builds. AppArmor is still enabled and enforcing, as shown by sudo aa-status, but only rsyslogd has a loaded profile on this VM.

Reflection:
AppArmor adds fine-grained control over what each service can access (files, sockets, etc.).
Even if an attacker exploits sshd, AppArmor limits what that compromised process can do.
5.2 Automatic Security Updates
To reduce the window of exposure for known vulnerabilities, I enabled automatic security updates using unattended-upgrades.
5.2.1 Installing and enabling unattended upgrades
On the server:
# Install unattended-upgrades and its dependencies
sudo apt update
sudo apt install unattended-upgrades -y

# Enable automatic security updates
sudo dpkg-reconfigure --priority=low unattended-upgrades
[Screenshot 5.5 – Installing unattended-upgrades]
[Screenshot 5.6 – dpkg-reconfigure screen or confirmation]
5.2.2 Verifying configuration
# Check the main unattended-upgrades configuration
sudo sed -n '1,80p' /etc/apt/apt.conf.d/50unattended-upgrades

# Check that the timer/service are enabled
systemctl status unattended-upgrades
systemctl status apt-daily.timer
[Screenshot 5.7 – Snippet of 50unattended-upgrades showing security origin enabled]
[Screenshot 5.8 – systemctl status unattended-upgrades]
Note:
Enabling automatic security updates helps keep the system patched without manual intervention, which is important for long-running servers.
5.3 Configuring fail2ban for Intrusion Detection
fail2ban monitors log files and temporarily bans IPs that show suspicious behaviour (e.g. repeated failed SSH logins).
5.3.1 Installing fail2ban
On the server:
sudo apt update
sudo apt install fail2ban -y
[Screenshot 5.9 – Installing fail2ban]
5.3.2 Basic SSH jail configuration
I created a local configuration file to enable the SSH jail. This keeps changes separate from the default config.
# Create or edit the local jail configuration
sudo nano /etc/fail2ban/jail.local
Example minimal content (adapt to your system):
[sshd]
enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 5
bantime  = 600
findtime = 600
Then restart and enable the service:
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban

# Check global status
sudo fail2ban-client status

# Check SSH jail status specifically
sudo fail2ban-client status sshd
[Screenshot 5.10 – Contents of /etc/fail2ban/jail.local]
[Screenshot 5.11 – fail2ban-client status showing sshd jail]
Reflection:
This setup protects the SSH service from brute-force attacks by automatically banning IPs that repeatedly fail to authenticate.
5.4 Security Baseline Verification Script (security-baseline.sh)
The goal of this script is to verify all key security controls from Weeks 4 and 5:
SSH hardening (PasswordAuthentication no, PermitRootLogin no)
Firewall enabled with expected rules
Automatic security updates configured
fail2ban running with sshd jail
AppArmor loaded
The script lives on the server, but I execute it remotely over SSH from the workstation.
5.4.1 Creating the script on the server
On the server (via SSH), create the script:
sudo nano /usr/local/bin/security-baseline.sh
Paste the following (adjust paths/values if needed):
#!/bin/bash
# security-baseline.sh
# Verifies key security settings on the server.

echo "===== Security Baseline Check ====="
echo "Date: $(date)"
echo

# Simple helper function to label each check
run_check() {
  local description="$1"
  local command="$2"

  echo "---- $description ----"
  echo "Command: $command"
  eval "$command"
  echo
}

# 1) Check SSH hardening: PasswordAuthentication
run_check "SSH: PasswordAuthentication is disabled" \
  "grep -E '^[[:space:]]*PasswordAuthentication[[:space:]]+no' /etc/ssh/sshd_config || echo 'WARNING: PasswordAuthentication is not set to no'"

# 2) Check SSH hardening: PermitRootLogin
run_check 'SSH: PermitRootLogin is set to no or prohibit-password' \
  "grep -E '^[[:space:]]*PermitRootLogin[[:space:]]+(no|prohibit-password)' /etc/ssh/sshd_config || echo 'WARNING: PermitRootLogin not correctly restricted'"

# 3) Check firewall status (ufw)
run_check 'Firewall (ufw) status' \
  'sudo ufw status verbose'

# 4) Check AppArmor status
run_check 'AppArmor status' \
  'sudo aa-status || echo \"AppArmor not available\"'

# 5) Check unattended-upgrades
run_check 'unattended-upgrades service status' \
  'systemctl is-enabled unattended-upgrades; systemctl status unattended-upgrades | head -n 10'

# 6) Check fail2ban + sshd jail
run_check 'fail2ban service status' \
  'systemctl status fail2ban | head -n 10'

run_check 'fail2ban sshd jail status' \
  'sudo fail2ban-client status sshd || echo \"sshd jail not found\"'

echo "===== Security Baseline Check Complete ====="
Make the script executable:
sudo chmod +x /usr/local/bin/security-baseline.sh
[Screenshot 5.12 – Editing security-baseline.sh in nano]
[Screenshot 5.13 – ls -l /usr/local/bin/security-baseline.sh showing executable bit]
5.4.2 Running the script via SSH from the workstation
On the workstation (Mac host terminal):
ssh <your_server_user>@<your_server_ip> 'sudo /usr/local/bin/security-baseline.sh'
[Screenshot 5.14 – Output of security-baseline.sh showing all checks]
Reflection:
Having a scripted baseline makes it easy to re-check my configuration after any change or OS update and is also useful for the final video demo.
5.5 Remote Monitoring Script (monitor-server.sh)
This script runs on my workstation and connects to the server over SSH to collect performance metrics. It stores each run in a timestamped log file.
Metrics collected (on the server):
Hostname and current date/time
uptime (load averages)
Top CPU processes
Memory usage (free -h)
Disk usage (df -h /)
Network sockets (ss -tuna)
5.5.1 Creating the script on the workstation
On the workstation:
nano ~/monitor-server.sh
Paste and adjust SERVER_USER and SERVER_HOST:
#!/bin/bash
# monitor-server.sh
# Runs basic performance checks on the remote server over SSH
# and saves the output to a timestamped log file.

# --- Configuration: set your SSH user and server IP/hostname ---
SERVER_USER="<your_server_user>"
SERVER_HOST="<your_server_ip>"

# Directory where logs will be stored on the workstation
LOG_DIR="$HOME/server-monitor-logs"

# Create log directory if it doesn't exist
mkdir -p "$LOG_DIR"

# Generate timestamped log file name
TIMESTAMP="$(date +%Y%m%d-%H%M%S)"
LOG_FILE="$LOG_DIR/monitor-$TIMESTAMP.txt"

echo "Writing server metrics to: $LOG_FILE"
echo "===== Server Monitor Run ====="        > "$LOG_FILE"
echo "Date: $(date)"                         >> "$LOG_FILE"
echo "Target: $SERVER_USER@$SERVER_HOST"    >> "$LOG_FILE"
echo                                       >> "$LOG_FILE"

# --- Remote commands executed on the server over SSH ---
ssh "$SERVER_USER@$SERVER_HOST" 'bash -s' << "EOF" >> "$LOG_FILE"
echo "---- Hostname ----"
hostname
echo

echo "---- Uptime ----"
uptime
echo

echo "---- Top CPU processes (top 5) ----"
ps aux --sort=-%cpu | head -n 5
echo

echo "---- Memory usage (free -h) ----"
free -h
echo

echo "---- Disk usage for / ----"
df -h /
echo

echo "---- Network sockets (first 10) ----"
ss -tuna | head -n 10
echo
EOF

echo "Done. Log saved at: $LOG_FILE"
Make the script executable:
chmod +x ~/monitor-server.sh
[Screenshot 5.15 – monitor-server.sh in nano on the workstation]
[Screenshot 5.16 – ls -l ~/monitor-server.sh]
5.5.2 Running the monitoring script
On the workstation:
./monitor-server.sh
Then view a sample log:
ls ~/server-monitor-logs
cat ~/server-monitor-logs/monitor-<timestamp>.txt
[Screenshot 5.17 – Terminal showing script run and created log file]
[Screenshot 5.18 – Excerpt of a monitoring log file]
Reflection:
This script gives a quick snapshot of server health without logging in manually.
In a real environment, similar scripts could be scheduled with cron or replaced by more advanced monitoring tools.
5.6 Weekly Summary & Reflection
I implemented advanced security layers: AppArmor, automatic updates, and fail2ban.
I created two reusable scripts:
security-baseline.sh to verify the full security configuration on the server
monitor-server.sh to collect remote performance metrics from the workstation
These steps prepare the system for Week 6 performance testing and Week 7 security audit, and match the Phase 5 deliverables in the brief.
What I learned:
How defence-in-depth works in practice (firewall + SSH hardening + AppArmor + fail2ban).
How Bash scripts and SSH can automate both security checks and monitoring, which is closer to real DevOps workflows.
