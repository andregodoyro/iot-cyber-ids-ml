# iot-cyber-ids-ml

## Overview
**iot-cyber-ids-ml** is an end-to-end Machine Learning Intrusion Detection System (IDS) engineered for network telemetry across Internet of Things (IoT) and Industrial IoT (IIoT) ecosystems. The project demonstrates reproducible medium-data handling, memory-conscious data engineering, and multi-class cyber threat classification.

## Key Technical Highlights
* **Medium-Data Engineering:** Implements memory-efficient ETL strategies utilizing columnar Parquet storage, data-type downcasting, and chunked ingestion to process large-scale telemetry within constrained compute environments.
* **Modular Pipeline Architecture:** Built with `scikit-learn` pipelines that enforce strict isolation between training and evaluation phases, guaranteeing zero data leakage during preprocessing and feature scaling.
* **Threat Taxonomy & Domain Alignment:** Evaluates multi-class attack vectors—including Denial of Service (DoS/DDoS), Ransomware, SQL Injection, Scanning, and Password Cracking—aligned with modern cybersecurity frameworks.

## Dataset & Academic Attribution
This research repository utilizes the **ToN_IoT Dataset**, developed by the UNSW Canberra Cyber research team (Cyber Range and IoT Labs).

* **Principal Investigator:** Dr. Nour Moustafa
* **Institution:** UNSW Canberra Cyber, Australia
* **Official Source:** [UNSW Canberra ToN_IoT Datasets](https://research.unsw.edu.au/projects/toniot-datasets)
* **Compliance:** Utilized strictly for non-commercial educational and research purposes in alignment with the original authors' terms.

### Recommended Citation
When referencing the underlying telemetry data, please cite the foundational paper:
> Moustafa, N. (2021). *New Generations of Internet of Things Datasets for Cybersecurity Applications Based on Artificial Intelligence Pipeline*. IEEE Transactions on Big Data.