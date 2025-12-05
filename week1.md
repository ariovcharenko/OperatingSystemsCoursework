# Week 1 – System Planning and Distribution Selection

## 1. Introduction
This week focuses on planning the deployment of a dual-system Linux environment required for the coursework. The goal is to choose the server distribution, define the workstation configuration, plan VirtualBox networking, and collect baseline system information using command-line tools.

## 2. Distribution Selection and Justification
For the server, I selected Ubuntu Server 24.04 LTS (ARM64).
Justification:
Long-term support (5 years), stable for coursework
Lightweight headless installation suitable for server tasks
Compatible with UTM on Apple Silicon (ARM64 architecture)
Strong documentation and straightforward package management (APT)
Alternatives considered:
Debian – also stable, but fewer built-in conveniences for beginners
CentOS/AlmaLinux – more enterprise-oriented and unnecessary complexity for this module
Ubuntu Server provides the right balance of stability, simplicity, and ARM support, making it the best fit for my setup.

## 3. Workstation Configuration
My workstation VM uses Ubuntu Desktop (UTM).
Reasoning:
Has a graphical interface, making it easier to manage GitHub, screenshots, and documentation
Includes built-in Terminal — works perfectly as the SSH client
Recommended by the module for students who prefer a Linux GUI for administrative tasks
This matches the module requirement for a separate workstation system used only to remotely administer the server.

## 4. Network Configuration:
Both VMs are connected using UTM’s Shared Network (NAT) interface.  
The server obtained IP 192.168.65.X, and the workstation obtained 192.168.65.Y.  
This allows SSH connectivity while keeping the VMs isolated from the host network.

## 5. System Information Commands and Evidence
Below is the required system information gathered from the server using the commands:
uname -a
free -h
df -h
ip addr
lsb_release -a
System Information Screenshot:
<img width="1280" height="832" alt="image" src="https://github.com/user-attachments/assets/4c3ffa10-e35b-4e36-a068-2ef6210d6c10" />


## 6. System Architecture Diagram
<img width="1200" height="500" alt="system-architecture" src="https://github.com/user-attachments/assets/9a9d7364-df1a-4f2b-9824-589718cd502d" />


## 7. Reflection for Week 1
This week I successfully installed both VMs, fixed ARM64 compatibility issues, and prepared the server for SSH-only administration. I also practiced collecting system information via CLI and documented the necessary setup steps for the technical journal. The main challenge was resolving the ISO architecture mismatch, but after switching to the ARM64 server image, the installation worked as expected.
