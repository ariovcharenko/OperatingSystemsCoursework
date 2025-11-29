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
| Baseline         | 0.9      | 1342.6           | 0.34               | 0             | X            | Idle state     |
| CPU stress       | 380     | 1401.9          | 0.00              | 0            | X            | `stress-ng`    |
| Redis idle       | 11.2       | 965          | 0.00               | 0            | X            | RAM baseline   |
| Redis benchmark  | 5.6      | 984         | 0.00               | 0           | X            | 100k ops       |
| Disk test (`dd`) | 4.8     | 1159          | 1400              | 0           | X            | Write perf     |
| iperf3           | 22      | 900        |0.00               | 5100            | X            | Throughput     |
| HTTP idle        | 1.5      | 1012       | 0.00            | 0,5          | X            | Python server  |
| HTTP load        | 48       | 1381         | 0.00            | 70            | X            | `ab` load test |

*(Values will be filled after testing)*

---

## **5. Performance Charts**

Include four graphs:

* CPU usage comparison
* Memory usage comparison
* Disk I/O comparison
* Network throughput comparison

<img width="579" height="422" alt="image" src="https://github.com/user-attachments/assets/a1384ffd-5ddf-4c5b-b515-5a6983f5f856" />
<img width="623" height="454" alt="image" src="https://github.com/user-attachments/assets/3899425b-c3ae-4bcb-b0aa-89e6fe8be44d" />
<img width="552" height="403" alt="image" src="https://github.com/user-attachments/assets/90b093c0-7f83-4000-b46d-2223b48a9862" />
<img width="564" height="422" alt="image" src="https://github.com/user-attachments/assets/5f0a379f-25b0-435d-81c9-7f1ce4328108" />

---

## **6. Application Testing Evidence**

### **6.1 CPU Stress Test**

Command:

```bash
stress-ng --cpu 4 --timeout 30s
```

https://chatgpt.com/backend-api/estuary/content?id=file_000000005e6472088661d30a9537424c&ts=490095&p=fs&cid=1&sig=2b2f4d33baae5f505e94f3166ca8e4e65d1d90e05353f16ba8fc7e96e882d7f8&v=0<img width="2048" height="1331" alt="image" src="https://github.com/user-attachments/assets/d448e459-02ac-4c75-a033-0cf9759813d1" />

<img width="1280" height="832" alt="image" src="https://github.com/user-attachments/assets/5f18013c-2a56-424c-bde6-a901b0209df1" />


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

<img width="1280" height="832" alt="image" src="https://github.com/user-attachments/assets/155af35b-599f-49bf-bb57-002d88e065c6" />

<img width="1023" height="480" alt="image" src="https://github.com/user-attachments/assets/8856d299-f7e8-4b6f-a56d-f45bdd4571d6" />


---

### **6.3 Disk I/O Test**

Command:

```bash
dd if=/dev/zero of=testfile bs=1G count=1 oflag=direct
```

<img width="1246" height="160" alt="image" src="https://github.com/user-attachments/assets/67ccc902-6faf-4dc0-9d16-2f016bd98510" />


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


<img width="1118" height="547" alt="image" src="https://github.com/user-attachments/assets/e62b8360-4ded-4b9d-bc29-30ef150f131c" />


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

<img width="1103" height="114" alt="image" src="https://github.com/user-attachments/assets/6249bcae-50f9-48d1-b260-89d70aae4782" />

<img width="1065" height="422" alt="image" src="https://github.com/user-attachments/assets/40410a90-987e-4216-9d25-7f929c3cbd42" />


---

## **7. Network Performance Analysis**

### **Latency**

```bash
ping -c 10 <server-ip>
```

<img width="1007" height="405" alt="image" src="https://github.com/user-attachments/assets/7f21ec93-331a-4a58-897d-1027c29874e8" />


### **Throughput (iperf3)**

<img width="1092" height="503" alt="image" src="https://github.com/user-attachments/assets/c705f9af-f6b6-4466-a187-748d8831c60e" />

---

## **8. Bottleneck Identification**

| Component | Bottleneck                  | Evidence         |
| --------- | --------------------------- | ---------------- |
| CPU       | Full utilisation under load | stress-ng output |
| Disk      | Low write throughput        | dd results       |
| Network   | Virtual NIC ceiling         | iperf results    |

<img width="1280" height="832" alt="image" src="https://github.com/user-attachments/assets/ebc37dba-a6a5-43d4-a799-9e6197372a15" />

<img width="822" height="193" alt="image" src="https://github.com/user-attachments/assets/0e9d6fa8-2f4f-4e2b-836b-1a5a66cf54f3" />


---

## **9. Optimisation Testing & Improvements**

### **Optimisation 1 — CPU Governor: performance**

```bash
sudo cpupower frequency-set -g performance
```

> Result: higher sustained CPU frequency
> ~15–25% improvement in stress-ng throughput

<img width="1280" height="832" alt="image" src="https://github.com/user-attachments/assets/acf6d7a2-22e4-41c3-aec3-c1448737e60c" />

<img width="1280" height="832" alt="image" src="https://github.com/user-attachments/assets/6b0c8e1b-3c49-42d8-a1e8-e96533e1a094" />


---

### **Optimisation 2 — Disk Scheduler: deadline**

```bash
echo deadline | sudo tee /sys/block/sda/queue/scheduler
```

> Result: lower latency and faster sequential writes
> ~20–30% improvement in disk performance

<img width="1259" height="190" alt="image" src="https://github.com/user-attachments/assets/909cd0ca-913f-4872-99e6-79f4cc9e7ee2" />
<img width="1231" height="153" alt="image" src="https://github.com/user-attachments/assets/a5d1ecdf-9ceb-4d01-815f-6fdbf408f3ff" />


---

## **10. Summary of Findings**

* CPU tests showed thermal throttling before optimisation
* Redis performed well but spiked under heavy operations
* Disk I/O was the slowest subsystem and benefitted most from tuning
* Network throughput was limited by virtual NIC capacity
