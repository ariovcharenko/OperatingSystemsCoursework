# Week 4 – Initial System Configuration & Security Implementation

## 1. Introduction
This week I started configuring the Ubuntu Server for secure remote administration.  
The main tasks were:
- Enabling and testing SSH
- Creating a non-root admin user
- Preparing for key-based authentication
- Installing and configuring the firewall (`ufw`)

## 2. SSH Service Setup
<img width="869" height="250" alt="image" src="https://github.com/user-attachments/assets/2a611c26-9e4a-419a-b7c5-c14d424089e3" />
<img width="599" height="115" alt="image" src="https://github.com/user-attachments/assets/c21a9701-3f71-442f-906b-83cf254d67ee" />
<img width="897" height="72" alt="image" src="https://github.com/user-attachments/assets/0ff35b47-e44d-453b-87e5-c7a3dfae176b" />
<img width="898" height="317" alt="image" src="https://github.com/user-attachments/assets/a469dbed-4e7b-4b7b-affe-4e68b7956fb1" />
Commands used:

```bash
sudo apt update
sudo apt install
openssh-server -y
sudo systemctl enable ssh
sudo systemctl status ssh
```



## 3. Non-root Administrative User
<img width="1062" height="397" alt="image" src="https://github.com/user-attachments/assets/4e99387e-1810-41d0-a277-086269e2d70e" />
Commands used:

```bash
sudo adduser adminarina
sudo usermod -aG sudo adminarina
id adminarina
```


## 4. Firewall Configuration (`ufw`)
<img width="1280" height="832" alt="image" src="https://github.com/user-attachments/assets/2311d9d8-a399-45f4-8a73-c5ad143e53bd" />


Commands:

```bash
sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Remove overly-broad SSH rule if it already exists
sudo ufw delete allow ssh 2>/dev/null

# Allow SSH ONLY from workstation IP
sudo ufw allow from 192.168.65.4 to any port 22 proto tcp

sudo ufw enable
sudo ufw status verbose
```


## 5. Remote Administration Evidence (SSH from Workstation)

SSH command used:

```bash
ssh adminarina@192.168.65.3
```
<img width="1280" height="666" alt="image" src="https://github.com/user-attachments/assets/7beef605-542a-406b-a0f9-417b1a6de323" />



## 6. Configuration File Changes (Before/After)
### Before Changes (`/etc/ssh/sshd_config`)
```text
#PermitRootLogin prohibit-password
#PasswordAuthentication yes
#PubkeyAuthentication yes
#ChallengeResponseAuthentication no
#UsePAM yes
```

After Changes (Hardened Configuration)
```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
UsePAM yes
AllowUsers dminarina
```
Explanation of Changes
PermitRootLogin no → disables direct login as root, reducing attack surface.
PasswordAuthentication no → enforces key-based authentication (stronger, not brute-forceable).
AllowUsers dminarina → restricts SSH access to only the new admin user.
PubkeyAuthentication yes → ensures SSH keys are required for login.
UsePAM yes → keeps Pluggable Authentication Module active for session management.


## 7. Reflection for Week 4

This week focused on establishing secure remote administration for my Linux server. I installed and configured the SSH service, enabled the firewall, and added a dedicated non-root administrator account to improve security. One of the key lessons was understanding how different network modes affect IP addressing in virtual machines, since I had to identify the correct server IP before connecting from the workstation.

I also verified service status using systemd tools and ensured SSH was enabled and persistent across reboots. Another important skill was troubleshooting connection issues such as “connection refused” and “broken pipe,” which helped me understand how authentication, user accounts, and service configuration interact. Finally, I reviewed the SSH configuration file and documented how a hardened version improves security by disabling root login, controlling authentication methods, and allowing only specific users to connect.

Overall, this week strengthened my ability to manage remote access securely and improved my confidence working with Linux services, firewall rules, and network diagnostics.

