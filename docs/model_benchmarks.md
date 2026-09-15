# Literature Benchmarks & Comparative Analysis (ToN_IoT)

The comparative analysis of Machine Learning (ML) and Deep Learning (DL) models applied to the ToN_IoT dataset encompasses traditional algorithms, ensemble methods, transfer learning networks, and hybrid architectures optimized for edge devices[1].

## 1. Ensemble Models vs. Traditional Algorithms
* **Tree-Based Models (Ensemble):** Algorithms such as Random Forest (RF), Decision Tree (DT), XGBoost, and LightGBM exhibit the best overall performance in binary and multiclass classification tasks[2]. In the Combined ToN_IoT Dataset, LightGBM achieved 100% accuracy and XGBoost reached **99%**[7][8].
* **Outperforming Classical Models:** These algorithms consistently outperform traditional methods such as Logistic Regression (LR), Naive Bayes (NB), K-Nearest Neighbors (KNN), and Support Vector Machines (SVM)[8], which frequently display lower accuracies or higher false alarm rates in complex IoT flows[10][11].

## 2. Deep Learning and Transfer Learning
* **Gated Recurrent Unit (GRU) with Knowledge Transfer:** The application of Deep Transfer Learning (TL) to GRU models allows training the network on a source device (such as a smart garage door) and classifying malicious traffic on an unlabeled target device (such as a thermostat)[12][13]. This approach increased accuracy from **69.20% up to 99.76%**[12], outperforming traditional non-transferred DL models (CNN, RNN, and DNN)[14][15].
* **Recurrent and Sequential Networks (LSTM / CNN):** Models such as LSTM and CNN excel at capturing temporal and multiclass patterns[2][16]. However, they require significantly more training time and higher computational overhead during validation compared to decision tree models[17][18].

## 3. Hybrid Architectures and Edge Detection
* **Compressed CNN-LSTM:** To bypass CPU and memory constraints at edge nodes, a hybrid CNN-LSTM classifier was developed using pruning and post-training quantization[19]. This model achieved 98.6% accuracy on ToN_IoT, becoming 71% smaller and 58% faster in inference than its uncompressed version[19].
* **Hybrid Random Forest (HRF):** Integrated into the Wazuh platform and combined with Pearson correlation feature selection, the HRF model reached 99.65% accuracy on ToN_IoT[3]. It outperformed hybrid deep baselines (such as GRU-BiLSTM) in speed and computational efficiency, reducing mean time to detect (MTTD) and mean time to respond (MTTR) by up to 75% and 65%, respectively[3].

## 4. Feature Selection and Explainability (XAI)
* **Reduced Feature Sets:** The application of Recursive Feature Elimination (RFE) enabled the selection of a universal set of only 6 network flow features (`dst_port`, `proto`, `conn_state`, `src_pkts`, `src_ip_bytes`, `dst_ip_bytes`)[20].
* **Performance and Speed:** Training a Decision Tree (DT) model with these 6 features achieved 99.62% accuracy[21][22] and an inference time per flow of only 0.45 µs (a reduction of up to 70% in prediction time)[22].
* **Explainability via SHAP:** The use of SHAP values proved that feature importance aligns with actual attack behaviors (for example, most scans and exploits utilize the TCP protocol and low destination ports)[25][26].