# ToN_IoT Domain Analysis & Feature Strategy

The ToN_IoT dataset simulates real-world IoT/IIoT cyberattacks by combining a physical and virtualized multilayer testbed with direct execution of authentic hacking tools [1].

---

## 1. Multilayer Testbed Architecture & Data Collection

Developed at the UNSW Canberra Cyber Range and IoT Labs (ADFA), the testbed leverages VMware NSX vCloud NFV virtualization and Software-Defined Networking (SDN) across three operational layers [2]:

* **Edge Layer:** Physical and simulated IoT/IIoT sensors and actuators (smart fridge, garage door, GPS tracker, thermostat, weather station, motion sensor, and Modbus registers), alongside mobile devices and Smart TVs [1].
* **Fog Layer:** Gateway servers running MQTT brokers, Node-RED platforms, and core network services (DNS, HTTP, DHCP, FTP, Email, Kerberos) [6, 9].
* **Cloud Layer:** Public cloud APIs (AWS Lambda, Azure IoT Hub) and centralized MQTT brokers [6, 10].

### Offensive Security Nodes
Attacks were launched from Kali Linux virtual machines attached directly to the testbed's virtual switches [6]. Attacking nodes were restricted to a dedicated IP subnet range (`192.168.159.30–39`) to isolate offensive traffic [6, 11].

### Multimodal Data Collection & Ground Truth Labeling
Data capture occurred simultaneously across three parallel perspectives [1]:
* **Network Level:** Raw `.pcap` files processed into structured network flows using **Zeek (Bro)** [17, 19].
* **Operating System Level:** Internal audit logs recorded via `atop` on Linux and Performance Monitor (`.blg`) on Windows [16].
* **Device Telemetry:** Physical and operational readings from IoT/IIoT sensors and actuators [16].

Microsecond-precision attack timestamps cross-referenced with originating attacker IP addresses established dataset ground truth [11].

---

## 2. Threat Categories & Attack Vectors

ToN_IoT executes industry-standard security utilities and penetration testing scripts across 9 threat categories [1, 12]:

* **Scanning & Reconnaissance:** Network and service discovery via Nmap, Nessus, and Xprobe2 [12, 13].
* **Denial of Service (DoS & DDoS):** Volumetric floods (TCP, UDP, SYN, HTTP flood) via Hping3, Scapy, ufonet, and Golden-Eye [12, 13].
* **Brute-Force Attacks:** Authentication cracking against SSH, FTP, and Telnet using Hydra and CeWL wordlists [12, 13].
* **Man-in-the-Middle (MitM):** ARP table poisoning and eavesdropping using Ettercap [12, 13].
* **Data Injection:** Intercepting MQTT queues and Node-RED scripts to manipulate actuator commands [12].
* **Web Application Attacks:** Cross-Site Scripting (XSS) and SQL injections against DVWA and OWASP Shepherd [9].
* **Exploits, Backdoors & Ransomware:** Metasploitable3 vulnerability exploits for reverse shells, alongside native ransomware execution on Windows 7/10 VMs [9, 12].

### Deep Dive: Ransomware Execution & OS Telemetry
Ransomware attack simulations take place directly in a virtualized environment hosting Windows 7 and Windows 10 target systems [1]:

* **Real Binary Execution & Privilege Escalation:** Launched from offensive Kali Linux nodes [4], attackers execute actual ransomware binaries alongside privilege escalation scripts directly on Windows virtual machines [1, 3]. The malware performs real-world encryption of local files and user profiles [1, 3].
* **OS-Level Metric Capture (Windows PerfMon):** While ransomware operates, host behavior is recorded in real time by the native **Performance Monitor (PerfMon)** tool [7]. PerfMon collectors log activity into raw `.blg` binary files, later converted into structured CSV files with a 133-attribute OS schema [8]. Encryption activity induces distinct behavioral spikes across processor usage (CPU), active thread scheduling, physical/virtual memory allocation, disk I/O requests, and network interface throughput [7].
* **Multimodal Collection & Ground Truth Labeling:** Ransomware data capture occurs simultaneously across OS logs (PerfMon), network captures (`.pcap` via Zeek), and device telemetry [7]. Accurate labeling cross-references microsecond attack timestamps with the Kali Linux attacker IP range (`192.168.159.30–39`), assigning binary attack labels (`label = 1`) and class category tags (`type = Ransomware`) [4].

---


## 3. Universal 6-Feature Subset & SHAP Explainability

To optimize the dataset for resource-constrained edge deployment, Alani & Miri (2022) applied Recursive Feature Elimination (RFE) with Random Forest feature importance, reducing raw dimensionality from 44 features down to 6 core network attributes [1]. This subset maintains **99.62% accuracy** with an ultra-low inference latency of **0.45 µs per flow** [1].

SHAP (Shapley Additive Explanations) analysis reveals feature contributions to model decisions [6]:

* **`proto` (Transport Protocol)**
  * **Measurement:** Identifies the transport layer protocol used in the flow (TCP, UDP, ICMP) [9].
  * **Model Impact:** Highest absolute impact [10]. Malicious traffic in ToN_IoT relies overwhelmingly on TCP, whereas benign traffic exhibits higher protocol variance [10].
* **`dst_port` (Destination Port)**
  * **Measurement:** Logical destination port [11, 12].
  * **Model Impact:** Second most relevant feature [12]. Low or well-known ports indicate reconnaissance scans and brute-force attacks; dynamic high ports signal normal traffic [12]. Extractable from the initial packet [11].
* **`src_ip_bytes` (Source IP Bytes)**
  * **Measurement:** Total bytes sent by source IP across headers and payloads [11, 13].
  * **Model Impact:** High transmission volumes signal volumetric floods (DoS/DDoS) [13]. Evaluated alongside `src_pkts` to identify low-byte attacks [13].
* **`src_pkts` (Source Packets)**
  * **Measurement:** Total packet count transmitted by source device [11, 13].
  * **Model Impact:** High packet emission rates confirm resource exhaustion attempts [13].
* **`dst_ip_bytes` (Destination IP Bytes)**
  * **Measurement:** Response bytes returned by target device [11, 14].
  * **Model Impact:** Legitimate flows return complete payloads; null or minimal bytes reflect rejected connections or failed exploits [14].
* **`conn_state` (Connection State)**
  * **Measurement:** TCP connection termination state (e.g., `S0`, `S1`, `REJ`) [11, 15].
  * **Model Impact:** Tracks handshake completion [15]. Incomplete states highlight port scanning or SYN flooding [15].

---

## 4. Practical Edge Deployment Advantages

* **Fast Extraction:** Features like `proto` and `dst_port` are parsed immediately from initial packet headers before flow termination [11].
* **Computational Efficiency:** Reduces model training time by up to 81% and testing/inference latency by approximately 70% [5, 16].
* **Low False Alarm Rate:** Achieves a False Positive Rate (FPR) of 0.27% and a **False Negative Rate (FNR) of 0.46%** [17].

---

## 5. Deep Transfer Learning (DTL) & Cross-Device Generalization

Deep Transfer Learning (DTL) improves cyberattack detection accuracy on target IoT devices by addressing one of Deep Learning's primary bottlenecks: the requirement for large volumes of labeled data specific to each type of equipment [1, 2].

### 5.1. Overcoming Data Scarcity & Class Imbalance
In real-world IoT and IIoT networks, the availability of labeled malicious traffic data for each individual device is very low, accompanied by severe class imbalance [2, 3]. DTL enables reusing knowledge extracted from a device with well-structured data (source domain) to classify traffic on a target device that lacks labeled data (target domain) [1].

### 5.2. Knowledge Transfer Mechanism (GRU-Based Architecture)
In the GRU-based (Gated Recurrent Unit) model, the network is initially trained on a source device (e.g., smart garage door or GPS tracker) [1]. The parameters, layers, and weights learned by this trained network are saved and transferred to a new target model (e.g., smart thermostat) [1]. Consequently, the target device inherits the capability to recognize complex attack patterns (such as DoS, DDoS, and Backdoors) without requiring full training from scratch [1].

### 5.3. Empirical Performance & Accuracy Leap
Experiments conducted with ToN_IoT telemetry demonstrated that applying Transfer Learning to GRU networks raised classification accuracy on target devices from **69.20% up to 99.76%** [1]:

* **Source-to-Target Generalization:** When using the smart GPS tracker or smart garage door as the source domain and the smart thermostat as the target domain, classification accuracy peaked at **99.76%** [1].
* **Comparison with Non-Transferred DL Models:** DTL significantly outperformed non-transferred traditional Deep Learning algorithms (conventional CNN, RNN, and DNN architectures), establishing a high-performance common baseline across heterogeneous edge devices [1].
