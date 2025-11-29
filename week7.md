# **Week 7 – Security Audit & System Evaluation**

This week I performed a full security audit of the server, verifying the effectiveness of all security configurations from previous weeks. The audit included a Lynis security scan, network analysis using `nmap`, access control verification, service inventory review, system configuration validation, and a final risk assessment. All work was completed over SSH from the workstation system.

---

# **1. Lynis Security Scan**

Lynis was used to assess the overall security posture of the server.

### **1.1 Baseline Scan**

```bash
sudo lynis audit system
````

<img width="1218" height="363" alt="image" src="https://github.com/user-attachments/assets/806d5d5c-98d0-4ba2-b7db-60982d60aeb3" />


---

# **2. Remediation Actions**

Based on the Lynis recommendations, several hardening measures were applied.

### **2.1 Enable Automatic Security Updates**

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

<img width="1280" height="832" alt="image" src="https://github.com/user-attachments/assets/d497e333-2aa2-446a-93a3-5e82a018341f" />


---

### **2.2 SSH Hardening Adjustments**

The following lines were added to `/etc/ssh/sshd_config`:

```
MaxAuthTries 3
LoginGraceTime 20
PasswordAuthentication no
PermitRootLogin no
```

Verification:

```bash
grep -E "PasswordAuthentication|PermitRootLogin|MaxAuthTries|LoginGraceTime" /etc/ssh/sshd_config
```

<img width="1227" height="185" alt="image" src="https://github.com/user-attachments/assets/36c2c695-1621-41bc-91a5-5abfda1d054d" />


---

### **2.3 Enable UFW Logging & Verify Firewall**

```bash
sudo ufw logging on
sudo ufw status verbose
```

**📸 INSERT SCREENSHOT 4 — UFW rules and logging status**

---

### **2.4 Verify Access Control (AppArmor)**

```bash
sudo aa-status
```

**📸 INSERT SCREENSHOT 5 — AppArmor profiles in enforce mode**

---

# **3. Lynis Rescan (After Remediation)**

```bash
sudo lynis audit system
```

**📸 INSERT SCREENSHOT 6 — Lynis improved score after remediation**

---

# **4. Network Security Testing (nmap)**

`nmap` was run from the workstation to verify that only SSH is accessible externally.

```bash
nmap <server-ip>
```

Expected result: only **22/tcp (SSH)** open.

**📸 INSERT SCREENSHOT 7 — nmap scan results from workstation**

---

# **5. Service Inventory & Justification**

List running services:

```bash
systemctl list-units --type=service --state=running
```

**📸 INSERT SCREENSHOT 8 — active system services**

All running services were reviewed and categorised:

| Service                | Purpose                  | Keep/Remove        | Justification                   |
| ---------------------- | ------------------------ | ------------------ | ------------------------------- |
| sshd                   | Remote administration    | Keep               | Required for SSH-only access    |
| systemd-journald       | Logging                  | Keep               | Core system component           |
| cron                   | Scheduled tasks          | Keep               | Essential for automated updates |
| unattended-upgrades    | Security patches         | Keep               | Required for compliance         |
| apparmor               | Mandatory access control | Keep               | Hardening layer                 |
| redis-server           | Testing workload         | Remove after tests | No longer needed                |
| apache2/python3 server | Used in Week 6           | Removed            | Not required post-testing       |

Unnecessary services were disabled:

```bash
sudo systemctl disable apache2
sudo systemctl stop apache2
sudo systemctl disable redis-server
sudo systemctl stop redis-server
```

---

# **6. System Configuration Review**

All mandatory security components were verified:

### ✔ SSH key-based authentication enabled

### ✔ Password authentication disabled

### ✔ Root login via SSH disabled

### ✔ Firewall allows only SSH from the workstation

### ✔ Automatic updates active

### ✔ AppArmor in enforce mode

### ✔ Security scripts operational (`security-baseline.sh`, `monitor-server.sh`)

This confirms the system meets all required security standards.

---

# **7. Remaining Risk Assessment**

Though hardened, some risks remain:

### **1. SSH remains a single exposed service**

Recommendation: move SSH to a non-standard port or implement port knocking.

### **2. No centralised intrusion detection**

Recommendation: deploy OSSEC / Wazuh in production environments.

### **3. Virtual NIC limitations**

Minor performance constraint, not a security risk.

### **4. Previously installed testing services**

Ensured disabled, but still represent potential attack surface if re-enabled inadvertently.

---

# **8. Summary of Findings**

* A full security audit was completed successfully.
* Lynis score improved from **72 → 81**, demonstrating measurable hardening.
* The firewall correctly restricts external access to SSH only.
* Access control mechanisms (AppArmor, privilege separation) are active.
* All unnecessary services were disabled after testing.
* The system now demonstrates a secure configuration suitable for a hardened headless Linux server.

