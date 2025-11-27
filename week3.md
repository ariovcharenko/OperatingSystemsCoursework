# Week 3 – Application Selection for Performance Testing

## 1. Introduction
This week I selected the applications I will use later for CPU, memory, disk I/O, and network performance testing.  
No installation or configuration is done yet — only planning.

---

## 2. Application Selection Matrix

The goal is to cover different workload categories: CPU-intensive, RAM-intensive, Disk-intensive, Network-intensive, and a lightweight server application.

| Category | Application | Why I Chose It |
|----------|-------------|----------------|
| **CPU-intensive** | `stress-ng` | Simple tool for generating CPU load using multiple worker threads. Easy to control load level. |
| **Memory-intensive** | `stress-ng --vm` | Generates predictable RAM pressure by allocating memory blocks. Very useful for Week 6 testing. |
| **Disk I/O-intensive** | `fio` | Industry-standard storage benchmark. Allows sequential, random, and write/read patterns. |
| **Network-intensive** | `iperf3` | Best tool for measuring network throughput and latency between workstation and server. |
| **Server application** | `nginx` | Lightweight web server; good for measuring request latency, concurrency, and CPU/network usage. |

---

## 3. Installation Commands (SSH-based)

These commands will be executed later from the workstation via SSH:

sudo apt update
sudo apt install stress-ng -y
sudo apt install fio -y
sudo apt install iperf3 -y
sudo apt install nginx -y


---

## 4. Expected Resource Profiles

| Application | Expected CPU | Expected RAM | Expected Disk | Expected Network |
|-------------|--------------|--------------|---------------|------------------|
| `stress-ng` | Very high | Low–Medium | Low | None |
| `stress-ng --vm` | Low | Very high (allocates memory) | Low | None |
| `fio` | Medium | Low | Very high | None |
| `iperf3` | Low–Medium | Low | Low | High (throughput testing) |
| `nginx` | Low | Low | Low | Medium (HTTP requests) |

This provides a wide range of behaviors to analyze in Week 6.

---

## 5. Monitoring Strategy for Each Application

**CPU + RAM**  
- `top`  
- `htop` (optional later)  
- `free -h`

**Disk I/O**  
- `iostat` or `vmstat` (optional)  
- `df -h` (disk usage before/after)

**Network**  
- `ping` (latency)  
- `iperf3` (throughput testing)

**Server performance (`nginx`)**  
- `ab` (Apache Benchmark) or a simple browser request loop  
- Monitor with `top` during load

---

## 6. Reflection for Week 3
This week helped me plan realistic performance scenarios ahead of time. By choosing simple but effective tools like `stress-ng`, `fio`, and `iperf3`, I can easily measure how different workloads affect CPU, memory, disk and network behavior in Week 6. I also included a real server application (`nginx`) to observe performance under actual HTTP traffic.

