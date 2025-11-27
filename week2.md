# Week 2 – Security Planning and Testing Methodology

## 1. Introduction

This week focuses on **planning** security and performance testing for the Ubuntu Server.  
I am not changing any configuration yet – only deciding how I will harden the system and how I will measure performance later (in Weeks 4–6).

---

## 2. Performance Testing Plan

### 2.1 Overall approach

- All tests will be run **remotely** from the Ubuntu Desktop workstation using **SSH** into the Ubuntu Server.
- I will run different workloads on the server (chosen in Week 3) and observe how CPU, memory, disk and network behave.
- Results will be stored later in a simple table in Week 6.

### 2.2 Tools I plan to use

From the **workstation** (over SSH):

- `top` – live CPU and memory usage.
- `free -h` – quick memory summary.
- `df -h` – disk usage overview.
- `ping` – latency between workstation and server.
- (Optionally) `vmstat` or `iostat` for more detailed CPU/disk stats if needed.

### 2.3 Metrics to collect

For each workload I will note:

- CPU usage (%)
- Memory usage (used / total)
- Disk usage / obvious disk bottlenecks
- Network latency using `ping`
- Any visible performance issues (high load, swapping, etc.)

These metrics will later be turned into a structured table and graphs in Week 6.

---

## 3. Security Configuration Checklist (Plan)

This is my **planned** security baseline for Weeks 4–5:

1. **SSH hardening**
   - Disable password authentication.
   - Use key-based login only.
   - Change default SSH config to limit root login.
   - Restrict which users can SSH into the server.

2. **Firewall configuration**
   - Enable `ufw` (Uncomplicated Firewall).
   - Default policy: deny all incoming, allow all outgoing.
   - Allow **only** SSH from the workstation IP.

3. **Mandatory access control**
   - Use **AppArmor** profiles for critical services (for example, SSH and any server apps I deploy).
   - Ensure profiles are in **enforce** mode, not complain mode.

4. **Automatic security updates**
   - Enable unattended security updates.
   - Make sure important security patches install automatically.

5. **User privilege management**
   - Create and use a **non-root admin user** with `sudo` access.
   - Avoid direct root login over SSH.
   - Use strong passwords for any local logins.

6. **Network security**
   - Keep only necessary services running.
   - Use `nmap` (later) from the workstation to confirm only expected ports are open.
   - Document which ports are allowed and why.

This checklist will be used later in Week 5 to build a verification script (`security-baseline.sh`).

---

## 4. Threat Model

Below are three concrete threats and how I plan to mitigate them.

### Threat 1 – SSH brute-force attacks

- **Risk:** An attacker repeatedly guesses passwords over SSH.
- **Impact:** Possible unauthorized access to the server.
- **Mitigations:**
  - Disable password authentication, use **SSH keys only**.
  - Use `fail2ban` in Week 5 to block IP addresses after repeated failed logins.
  - Restrict SSH to the workstation IP via firewall.

---

### Threat 2 – Exploiting outdated packages

- **Risk:** Vulnerabilities in old software versions could be exploited.
- **Impact:** Privilege escalation or remote code execution.
- **Mitigations:**
  - Enable **automatic security updates**.
  - Run `sudo apt update && sudo apt upgrade` regularly.
  - Before Week 7 security audit, ensure all packages are up to date.

---

### Threat 3 – Misconfigured firewall / open services

- **Risk:** Extra services or ports left open to the network.
- **Impact:** Larger attack surface, easier lateral movement for an attacker.
- **Mitigations:**
  - Use `ufw` with “deny incoming by default”.
  - Explicitly allow only SSH (and later any required service ports).
  - Use `nmap` from the workstation in Week 7 to confirm the final exposed surface.

---

## 5. Reflection for Week 2

This week I planned how I will **measure performance** and **harden security** before actually changing the system configuration.  
Having a clear checklist and threat model should make Weeks 4–5 simpler, because I already know which tools and controls I need (SSH keys, firewall, AppArmor, fail2ban, automatic updates). It also aligns my work with the module requirement to consider both security and performance when configuring the OS.
