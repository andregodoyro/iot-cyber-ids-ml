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

## 5. Deep Transfer Learning (DTL) & Recurrent Architectures (GRU)

Deep Transfer Learning (DTL) combined with Gated Recurrent Units (GRU) addresses one of the primary operational bottlenecks in IoT/IIoT intrusion detection: the severe shortage of labeled malicious telemetry across heterogeneous edge endpoints and extreme class imbalance [1, 2, 3].

### 5.1. Problem Statement: Data Scarcity & Heterogeneous IoT Edge
In production IoT deployments, individual smart sensors (e.g., thermostats, motion detectors) rarely possess sufficient labeled attack samples to train high-capacity deep learning networks from scratch [1, 2]. DTL bypasses full retraining by extracting non-linear temporal dynamics from a telemetry-rich source device (source domain) and transferring frozen layer representations to an unlabelled target device (target domain) [1, 4].

### 5.2. 5-Layer DTL-GRU Architecture Specifications
The architecture proposed by Poonkuzhali et al. consists of a 5-layer deep recurrent structure engineered specifically for sequential telemetry processing [1, 3]:

* **Input Layer:** Formed by 9 neurons with Linear/Identity activation, corresponding directly to the 9 extracted sensor telemetry attributes [3, 6].
* **Recurrent Layer 1 (GRU 1):** First recurrent layer containing 256 GRU neurons using Tanh/Sigmoid gating to capture macro temporal dependencies and sequential patterns [3, 6].
* **Recurrent Layer 2 (GRU 2):** Second recurrent layer containing 256 GRU neurons with Tanh/Sigmoid gating to extract higher-level temporal features from the initial recurrent output [3, 6].
* **Recurrent Layer 3 (GRU 3):** Third recurrent layer containing 256 GRU neurons with Tanh/Sigmoid gating to encode complex non-linear temporal state transitions [3, 6].
* **Recurrent Layer 4 (GRU 4):** Fourth recurrent layer containing 64 GRU neurons utilizing the ReLU activation function to compress hidden representations while preventing vanishing gradients [3, 6].
* **Output Dense Layer:** Final classification layer containing 1 neuron. The base model utilizes ReLU activation, whereas the target model attaches an additional Sigmoid activation layer to map latent representations directly into binary decisions (`0 = Normal`, `1 = Attack`) [3, 7].

### 5.3. Hyperparameters & Optimization Pipeline
Model convergence and optimization are governed by the following hyperparameter settings [3]:

* **Loss Function:** Binary Cross-Entropy, calculating logarithmic loss for binary decision outputs [3].
* **Optimizer:** `RMSprop` (Root Mean Square Propagation), providing adaptive learning rates well-suited for non-stationary recurrent gradient dynamics [3].
* **Batch Size:** 1000 samples per mini-batch, balancing gradient stability with processing throughput [3].
* **Training Epochs:** 10 epochs, sufficient for loss stabilization and convergence without overfitting [3].

### 5.4. Knowledge Transfer Mechanism (`GRUbasemodel` → `GRUtargetmodel`)
The transfer learning pipeline is executed in two distinct stages [4, 5]:

1. **Source Training (`GRUbasemodel`):** The base model is trained from scratch using full labeled telemetry from a source device (e.g., smart garage door or smart fridge) [1, 5].
2. **Knowledge Extraction & Transfer:** Learned weights, hidden layer parameters, and gating mechanisms are saved and frozen [4, 5].
3. **Target Adaptation (`GRUtargetmodel`):** The frozen network is transferred to the target device domain (e.g., smart thermostat) [1, 5]. An additional Sigmoid activation layer is attached to the output to fine-tune classification without retraining the underlying GRU layers from scratch [4, 5].

### 5.5. Hybrid GRU Architectures in ToN_IoT Benchmarking
Beyond standalone DTL models, GRU units are utilized in hybrid deep learning architectures across ToN_IoT research to benchmark performance against lightweight tree-based models [2, 3]:

* **LSTM + DENSE + GRU Hybrid:** Integrates Long Short-Term Memory (LSTM) blocks, dense layers, and GRU units to fuse long-term sequence tracking with rapid gated representations, supporting both binary and multi-class attack classification [2, 11].
* **GRU-BiLSTM Baseline:** Combines Bidirectional LSTM (evaluating past-to-future and future-to-past contexts) with GRU layers [3]. Serves as a deep recurrent benchmark to evaluate computational overhead and inference latency against lightweight models such as Hybrid Random Forest [3, 12].

### 5.6. Empirical Performance & Accuracy Leap
Experimental evaluations using ToN_IoT telemetry confirm significant gains achieved through knowledge transfer [1]:

* **Accuracy Leap:** Applying DTL-GRU elevated target device classification accuracy from **69.20% up to 99.76%** (specifically when transferring from a smart garage door/GPS tracker source to a smart thermostat target) [1].
* **Outperforming Non-Transferred Models:** The transferred DTL-GRU architecture consistently outperformed conventional, non-transferred deep learning architectures (including standard CNN, RNN, and DNN baselines), demonstrating robust cross-device generalization across heterogeneous edge environments [1].
