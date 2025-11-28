# Week 5 – Advanced Security & Monitoring Infrastructure

This week I implemented advanced security controls and monitoring capabilities on my Ubuntu Server. Building on the SSH hardening and firewall configuration from Week 4, I:

* Verified and documented mandatory access control using AppArmor
* Configured automatic security updates
* Installed and configured **fail2ban** for intrusion detection
* Wrote a security baseline verification script (`security-baseline.sh`) that runs on the server
* Wrote a remote monitoring script (`monitor-server.sh`) that runs on my workstation and collects performance metrics over SSH

---

## 5.1 Access Control with AppArmor

Ubuntu uses **AppArmor** as its Mandatory Access Control (MAC) system. It restricts what applications can access (files, directories, network capabilities) even if they are exploited.

### 5.1.1 Checking AppArmor status

Commands run on the server:

```bash
# Check AppArmor service status
sudo systemctl status apparmor

# List loaded AppArmor profiles and modes
sudo aa-status
```

<img width="900" height="293" alt="image" src="https://github.com/user-attachments/assets/70c3a128-94f5-4ebd-a777-bbfc2e971b37" />
<img width="614" height="258" alt="image" src="https://github.com/user-attachments/assets/ce314533-063d-44a8-b76d-0aa061b25ecd" />


### 5.1.2 Listing available AppArmor profiles

My Ubuntu Server image uses a minimal AppArmor configuration. Some profiles — including the one for `sshd` — are not included on this image, which is normal.

To list all available profiles:

```bash
ls /etc/apparmor.d/
```

<img width="527" height="83" alt="image" src="https://github.com/user-attachments/assets/5cb42c95-da47-46ba-ac95-1ca8745d0c61" />


### Note

My Ubuntu Server image does not include an AppArmor profile for OpenSSH (`/etc/apparmor.d/usr.sbin.sshd`). This is typical for minimal VM builds. AppArmor is still enabled and enforcing, as shown by:

```bash
sudo aa-status
sudo systemctl status apparmor
```

The only active profile on this VM is for `rsyslogd`.

### Reflection

AppArmor provides fine‑grained access control for system services, limiting what they can do even if compromised. Even without an sshd profile, AppArmor contributes to an overall defence‑in‑depth security strategy.

---

## 5.2 Automatic Security Updates

Automatic security updates reduce exposure to known vulnerabilities by keeping the system patched without manual intervention.

### 5.2.1 Installing and enabling automatic updates

Run on the server:

```bash
sudo apt update
sudo apt install unattended-upgrades -y

# Enable automatic security updates
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

(*Screenshot: installation of unattended-upgrades*)
(*Screenshot: dpkg-reconfigure confirmation*)

### 5.2.2 Verifying configuration

```bash
# View main configuration file
sudo sed -n '1,80p' /etc/apt/apt.conf.d/50unattended-upgrades

# Verify services
systemctl status unattended-upgrades
systemctl status apt-daily.timer
```

<img width="563" height="151" alt="image" src="https://github.com/user-attachments/assets/a2345ba4-bf65-47ce-afc6-08446290b1db" />
<img width="1280" height="832" alt="image" src="https://github.com/user-attachments/assets/c96b4272-5db7-4ee7-b8e0-176e1f297484" />


---

## 5.3 fail2ban Intrusion Detection

`fail2ban` monitors log files (such as `/var/log/auth.log`) and bans IPs that repeatedly fail authentication, helping prevent brute‑force SSH attacks.

### 5.3.1 Installing fail2ban

```bash
sudo apt update
sudo apt install fail2ban -y
```

<img width="866" height="455" alt="image" src="https://github.com/user-attachments/assets/608e38a3-c24d-4899-bafe-44842ea97696" />
<img width="1280" height="832" alt="image" src="https://github.com/user-attachments/assets/13984dd4-30e8-4fc5-8e8f-728139411b8e" />



### 5.3.2 Configuring the SSH jail

```bash
sudo nano /etc/fail2ban/jail.local
```

Add:

```ini
[sshd]
enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 5
bantime  = 600
findtime = 600
```

Restart and verify:

```bash
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban

# View fail2ban status
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

<img width="1276" height="827" alt="image" src="https://github.com/user-attachments/assets/39882d3c-0713-442a-b54e-961977a351af" />
<img width="1133" height="154" alt="image" src="https://github.com/user-attachments/assets/bf38a8c3-d109-4be8-be34-48dca5db4ea1" />
<img width="667" height="190" alt="image" src="https://github.com/user-attachments/assets/cbceb08b-556e-41d8-bb31-2cd9a5734b35" />


### Reflection

This configuration automatically bans malicious IPs that repeatedly attempt to log in, reducing exposure to brute-force attacks.

---

## 5.4 Security Baseline Verification Script (security-baseline.sh)

This script verifies all key security controls from Weeks 4 and 5:

* SSH hardening
* Firewall rules
* Automatic security updates
* AppArmor status
* fail2ban jail status

### 5.4.1 Creating the script on the server

```bash
sudo nano /usr/local/bin/security-baseline.sh
```

Paste:

```bash
#!/bin/bash
# security-baseline.sh
# Verifies key security settings on the server.

echo "===== Security Baseline Check ====="
echo "Date: $(date)"
echo

run_check() {
  local description="$1"
  local command="$2"
  echo "---- $description ----"
  echo "Command: $command"
  eval "$command"
  echo
}

# 1) SSH PasswordAuthentication\ run_check "SSH: PasswordAuthentication is disabled" \
  "grep -E '^[[:space:]]*PasswordAuthentication[[:space:]]+no' /etc/ssh/sshd_config || echo 'WARNING: PasswordAuthentication not set to no'"

# 2) SSH PermitRootLogin
run_check "SSH: PermitRootLogin is restricted" \
  "grep -E '^[[:space:]]*PermitRootLogin[[:space:]]+(no|prohibit-password)' /etc/ssh/sshd_config || echo 'WARNING: PermitRootLogin not configured correctly'"

# 3) Firewall
run_check "Firewall (ufw) status" 'sudo ufw status verbose'

# 4) AppArmor
run_check "AppArmor status" 'sudo aa-status'

# 5) Automatic updates
run_check "unattended-upgrades service status" \
  'systemctl is-enabled unattended-upgrades; systemctl status unattended-upgrades | head -n 10'

# 6) fail2ban
run_check "fail2ban service status" 'systemctl status fail2ban | head -n 10'
run_check "fail2ban sshd jail status" 'sudo fail2ban-client status sshd'

echo "===== Security Baseline Check Complete ====="
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/security-baseline.sh
```

<img width="1280" height="823" alt="image" src="https://github.com/user-attachments/assets/1cb02706-5bb0-4cd0-871d-291e19b57430" />
<img width="1218" height="426" alt="image" src="https://github.com/user-attachments/assets/2b35573f-3573-432a-a652-634154d72dfc" />
<img width="1215" height="475" alt="image" src="https://github.com/user-attachments/assets/d4d3366b-8c68-4dce-8101-e529d2b3276d" />




### 5.4.2 Running the script from the workstation

```bash
ssh <your_user>@<your_server_ip> 'sudo /usr/local/bin/security-baseline.sh'
```

<img width="1280" height="832" alt="image" src="https://github.com/user-attachments/assets/8b49a614-a581-4ddc-8663-3e39fc388473" />


---

## 5.5 Remote Monitoring Script (monitor-server.sh)

This script runs on my workstation and connects to the server via SSH to collect performance metrics.

### 5.5.1 Creating the script on the workstation

```bash
nano ~/monitor-server.sh
```

Paste:

```bash
#!/bin/bash
# monitor-server.sh
# Collects key server performance metrics over SSH.

SERVER_USER="<your_user>"
SERVER_HOST="<your_server_ip>"
LOG_DIR="$HOME/server-monitor-logs"
mkdir -p "$LOG_DIR"

TIMESTAMP="$(date +%Y%m%d-%H%M%S)"
LOG_FILE="$LOG_DIR/monitor-$TIMESTAMP.txt"

echo "Writing server metrics to: $LOG_FILE" > "$LOG_FILE"
ssh "$SERVER_USER@$SERVER_HOST" 'bash -s' << "EOF" >> "$LOG_FILE"
echo "---- Hostname ----"
hostname

echo "---- Uptime ----"
uptime

echo "---- Top CPU processes ----"
ps aux --sort=-%cpu | head -n 5

echo "---- Memory usage ----"
free -h

echo "---- Disk usage ----"
df -h /

echo "---- Network sockets ----"
ss -tuna | head -n 10
EOF
```

Make executable:

```bash
chmod +x ~/monitor-server.sh
```

<img width="1280" height="832" alt="image" src="https://github.com/user-attachments/assets/fd92a814-068d-4a8f-b5ad-20807d45d1be" />


<img width="887" height="74" alt="image" src="https://github.com/user-attachments/assets/ae5da3b6-7a19-4a77-8b1d-df2aa99fa45c" />


### 5.5.2 Running the script

```bash
./monitor-server.sh
ls ~/server-monitor-logs
cat ~/server-monitor-logs/monitor-<timestamp>.txt
```

(*Screenshot: script execution and log file*)

---

## 5.6 Reflection

This week, I implemented multiple layers of advanced security and monitoring. AppArmor provides access control, automatic updates ensure timely patching, and fail2ban helps mitigate brute-force attacks. The custom scripts provide repeatable security verification and remote monitoring, which are essential practices in system administration and DevOps workflows.
