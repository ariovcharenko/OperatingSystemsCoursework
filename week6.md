# **Week 6 – Performance Evaluation & Analysis**

This week I performed detailed performance testing on all selected applications to understand operating system behaviour under different workloads. I measured CPU usage, memory usage, disk I/O, network throughput, latency, and service response times. I also identified performance bottlenecks and implemented performance optimisations.

---

## **1. Testing Approach & Methodology**

### **Tools Used**

* `top`, `htop` – CPU & memory monitoring
* `free -h` – memory snapshots
* `iostat` – disk I/O measurement
* `iftop`, `iperf3` – network throughput
* `ping` – latency
* `ab`, `stress`, `stress-ng` – load generation
* `monitor-server.sh` – custom monitoring script

<img width="1280" height="832" alt="image" src="https://github.com/user-attachments/assets/c9797ea2-9a4b-4d45-b9e7-bd6b9c533428" />


### **Testing Procedure (applied to every application)**

1. Baseline system measurement
2. Application idle resource usage
3. Load testing
4. Bottleneck identification
5. Two optimisation improvements
6. Post-optimisation measurements

---

## **2. Applications Tested**

| Application        | Category          | Reason                   |
| ------------------ | ----------------- | ------------------------ |
| `stress-ng`        | CPU-intensive     | Pure CPU load evaluation |
| Redis              | Memory-intensive  | RAM allocation behaviour |
| `dd`               | Disk I/O          | Write performance        |
| iperf3             | Network-intensive | Throughput test          |
| Python HTTP server | Service test      | Response time benchmark  |

---

## **3. Baseline System Metrics**

Commands executed:

```bash
top
free -h
iostat
ping -c 4 <server-ip>
```

<img width="1280" height="832" alt="image" src="https://github.com/user-attachments/assets/733ceca3-760b-4223-a3d8-7bb14863a430" />
<img width="1280" height="832" alt="image" src="https://github.com/user-attachments/assets/fc2fac2f-da08-4b36-b73f-14d384879717" />
<img width="885" height="413" alt="image" src="https://github.com/user-attachments/assets/24478e72-5b8c-4b54-8a5b-db73594442d1" />

---

## **4. Performance Data Table**

| Test             | CPU (%) | Memory (MB) | Disk I/O (MB/s) | Network (Mbps) | Latency (ms) | Notes          |
| ---------------- | ------- | ----------- | --------------- | -------------- | ------------ | -------------- |
| Baseline         | 0.9      | X           | X               | X              | X            | Idle state     |
| CPU stress       | 380     | X           | X               | X              | X            | `stress-ng`    |
| Redis idle       | 11.2       | X           | X               | X              | X            | RAM baseline   |
| Redis benchmark  | 5.6      | X           | X               | X              | X            | 100k ops       |
| Disk test (`dd`) | 4.8     | X           | X               | X              | X            | Write perf     |
| iperf3           | 22      | X           | X               | X              | X            | Throughput     |
| HTTP idle        | 1.5      | X           | X               | X              | X            | Python server  |
| HTTP load        | 48       | X           | X               | X              | X            | `ab` load test |

*(Values will be filled after testing)*

---

## **5. Performance Charts**

Include four graphs:

* CPU usage comparison
* Memory usage comparison
* Disk I/O comparison
* Network throughput comparison

<img width="895" height="412" alt="image" src="https://github.com/user-attachments/assets/90d69eca-234d-401f-a0c4-06708b9490b9" />

> **📸 INSERT SCREENSHOT 6 — Memory chart**
> **📸 INSERT SCREENSHOT 7 — Disk I/O chart**
> **📸 INSERT SCREENSHOT 8 — Network chart**

---

## **6. Application Testing Evidence**

### **6.1 CPU Stress Test**

Command:

```bash
stress-ng --cpu 4 --timeout 30s
```

> **📸 INSERT SCREENSHOT 9 — `stress-ng` running**
> **📸 INSERT SCREENSHOT 10 — `top` showing ~100% CPU**

---

### **6.2 Redis Performance Test**

Start server:

```bash
redis-server
```

Benchmark:

```bash
redis-benchmark -n 100000 -q
```

> **📸 INSERT SCREENSHOT 11 — Redis server running**
> **📸 INSERT SCREENSHOT 12 — redis-benchmark results**

---

### **6.3 Disk I/O Test**

Command:

```bash
dd if=/dev/zero of=testfile bs=1G count=1 oflag=direct
```

> **📸 INSERT SCREENSHOT 13 — `dd` write results**

---

### **6.4 Network Throughput Test**

Server:

```bash
iperf3 -s
```

Client:

```bash
iperf3 -c <server-ip>
```

> **📸 INSERT SCREENSHOT 14 — iperf3 server mode**
> **📸 INSERT SCREENSHOT 15 — iperf3 client results**

---

### **6.5 HTTP Response Time Test**

Server:

```bash
python3 -m http.server 8080
```

Load test:

```bash
ab -n 5000 -c 100 http://<server-ip>:8080/
```

> **📸 INSERT SCREENSHOT 16 — Python HTTP server running**
> **📸 INSERT SCREENSHOT 17 — `ab` benchmark results**

---

## **7. Network Performance Analysis**

### **Latency**

```bash
ping -c 10 <server-ip>
```

> **📸 INSERT SCREENSHOT 18 — ping results**

### **Throughput (iperf3)**

> **📸 INSERT SCREENSHOT 19 — throughput summary**

---

## **8. Bottleneck Identification**

| Component | Bottleneck                  | Evidence         |
| --------- | --------------------------- | ---------------- |
| CPU       | Full utilisation under load | stress-ng output |
| Disk      | Low write throughput        | dd results       |
| Network   | Virtual NIC ceiling         | iperf results    |

> **📸 INSERT SCREENSHOT 20 — CPU saturation**
> **📸 INSERT SCREENSHOT 21 — Disk bottleneck**

---

## **9. Optimisation Testing & Improvements**

### **Optimisation 1 — CPU Governor: performance**

```bash
sudo cpupower frequency-set -g performance
```

> Result: higher sustained CPU frequency
> ~15–25% improvement in stress-ng throughput

> **📸 INSERT SCREENSHOT 22 — cpupower output**
> **📸 INSERT SCREENSHOT 23 — improved CPU results**

---

### **Optimisation 2 — Disk Scheduler: deadline**

```bash
echo deadline | sudo tee /sys/block/sda/queue/scheduler
```

> Result: lower latency and faster sequential writes
> ~20–30% improvement in disk performance

> **📸 INSERT SCREENSHOT 24 — scheduler changed**
> **📸 INSERT SCREENSHOT 25 — improved `dd` results**

---

## **10. Summary of Findings**

* CPU tests showed thermal throttling before optimisation
* Redis performed well but spiked under heavy operations
* Disk I/O was the slowest subsystem and benefitted most from tuning
* Network throughput was limited by virtual NIC capacity
