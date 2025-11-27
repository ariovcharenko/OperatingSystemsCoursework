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



## 3. Non-root Administrative User
<img width="1062" height="397" alt="image" src="https://github.com/user-attachments/assets/4e99387e-1810-41d0-a277-086269e2d70e" />
Commands used:

```bash
sudo adduser adminarina
sudo usermod -aG sudo adminarina
id adminarina


## 4. Firewall Configuration (`ufw`)
<img width="902" height="783" alt="image" src="https://github.com/user-attachments/assets/2dd981ed-dfbe-435b-af01-3e8db5124efb" />

Commands:

```bash
sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw enable
sudo ufw status verbose

*(We will paste the rules + `ufw status` output + 1 screenshot here)*

## 5. Remote Administration Evidence (SSH from Workstation)

*(We will add the SSH command + 1 screenshot here)*

## 6. Configuration File Changes (Before/After)

*(We’ll add small snippets from `sshd_config` once we edit it)*

## 7. Reflection for Week 4

*(We’ll write 1 short paragraph at the end)*
