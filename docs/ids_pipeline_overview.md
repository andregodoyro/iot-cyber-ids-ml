# IDS Pipeline Overview: End-to-End Execution Plan
This modular execution plan for the `notebooks/` directory of the `iot-cyber-ids-ml` project is strictly structured into 4 sequential steps. It guides the development of the intrusion detection pipeline, from exploratory inspection to the advanced Deep Transfer Learning (DTL-GRU) model, guaranteeing reproducibility and data leakage prevention.

## 1.0-eda.ipynb — Exploratory Data Analysis and Statistical Selection

* **Main Objective:** Inspect traffic/telemetry distributions, compare the behavior of the source device (`IoT_Garage_Door`) with the target device (`IoT_Thermostat`), and statistically select the features with the highest discriminatory power.
* **Statistical Methods and Analyses:**
* **Descriptive Statistics and Class Profiling:** Calculation of counts, means, medians, standard deviations, quartiles, and skewness for continuous traffic features, aggregated by attack class (`label`) and category (`type`).
* **Temporal Behavioral Analysis ($\Delta t$):** Calculation of inter-arrival time ($\Delta t = ts_i - ts_{i-1}$) per IP address. Application of non-parametric Mann-Whitney U tests (to compare temporal medians) and Kolmogorov-Smirnov (K-S) tests to prove divergence between regular traffic (heartbeat) and burst traffic from volumetric attacks (DoS, DDoS, Scanning).
* **Correlation Matrices:**
* **Pearson ($r$):** Evaluation of direct linear association between continuous numerical volume features (bytes and packets).
* **Spearman ($\rho$):** Evaluation of non-linear monotonic relationships immune to network outliers.


* **Categorical Association (Cramér's V):** Chi-Square test ($\chi^2$) and Cramér's V ($V$) calculation to measure the degree of dependence between categorical protocol/service variables (`proto`, `service`, `conn_state`) and the target variable `label`.
* **Mutual Information (Information Gain):** Application of the $k$-NN estimator (`mutual_info_classif`) to measure the reduction in Shannon entropy of class $Y$ given feature $X$, quantifying complex and non-linear dependencies.


* **EDA Evaluation Metrics:**
* $p$-values of hypothesis tests ($p < 0.05$).
* Correlation coefficients ($r, \rho$) and Cramér's V ($V \in$).
* Ranked Mutual Information score ($I(X; Y)$).



## 2.0-feature-engineering.ipynb — Pipeline and Standardization

* **Main Objective:** Execute raw data sanitization, select the universal feature subset, apply stabilizing transformations, and structure time sequences.
* **Data Transformations and Pipeline:**
* **Sanitization and Imputation:**
* Replacement of Zeek/Bro missing character indicators ('-') with standard nulls (`NaN`).
* Numerical imputation using the median to protect the pipeline against the impact of outliers.
* Categorical imputation using the mode or explicit category (e.g., `'missing'`) in functional attributes such as `door_state` and `service`.


* **Selection of the 6 Universal Variables:** Filtering down to a reduced set of 6 network flow attributes (`proto`, `dst_port`, `src_ip_bytes`, `src_pkts`, `dst_ip_bytes`, `conn_state`), guaranteeing low-overhead cross-device generalization.
* **Logarithmic Transformation:** Application of $\log(1 + x)$ on heavy-tailed variables with extreme right-skewness (such as $\Delta t$, `src_ip_bytes`, and `dst_ip_bytes`) to stabilize variance and approximate a normal distribution.
* **Normalization:** Feature scaling via `MinMaxScaler` within the range, preventing volumetric attributes from dominating gradient learning.
* **Time Sequence Structuring:** Construction of 3D sliding windows formatted as `(samples, timesteps, features)` to feed the recurrent architecture of the GRU model.


* **Generated Artifacts:**
* `.parquet` or `.csv` file containing the cleaned and standardized dataset.
* Pickle files of fitted scalar transformers and encoders trained solely on the source device's training set.



## 3.0-baseline-decision-tree.ipynb — Comparison Benchmark Model

* **Main Objective:** Train a simple supervised Decision Tree classifier (CART) using the 6 universal variables, establishing a lightweight, interpretable baseline with extremely fast inference speed.
* **Algorithm and Configuration:**
* **Algorithm:** Decision Tree (Classification and Regression Trees - CART) with splitting criteria based on Entropy / Information Gain or Gini Impurity.
* **Input:** Reduced set of 6 universal variables.
* **Validation:** Stratified split (80% train / 20% test) or 10-fold cross-validation.


* **Evaluation Metrics and Diagnostics:**
* **Confusion Matrix:** Calculation of True Positives (TP), True Negatives (TN), False Positives (FP), and False Negatives (FN).
* **Performance Metrics:** Accuracy, Precision, Recall (Detection Rate), and F1-Score.
* **ROC Curve and AUC-ROC:** Evaluation of binary separability between normal traffic and intrusions.
* **Computational Efficiency:** Total training time in seconds and average inference time per flow in microseconds ($\mu$s/flow).



## 4.0-dtl-gru-model.ipynb — Deep Transfer Learning with GRU

* **Main Objective:** Build and train a Recurrent Neural Network based on GRU (Gated Recurrent Unit) with Deep Transfer Learning (DTL) architecture, transferring knowledge from the source device (Garage) to the target device (Thermostat) without requiring prior target labels.
* **Architecture and Transfer Learning Flow:**
1. **Source Base Network (`GRUbasemodel`):**


* **Input:** Neurons corresponding to the selected input features.
* **5-Layer Architecture:** 3 consecutive GRU layers with 256 units each, followed by a 4th GRU layer with 64 units and ReLU activation.
* **Optimization:** RMSprop optimizer, binary cross-entropy loss function, 10 epochs, and batch size of 1000.
* **Training:** Full training from scratch on the labeled dataset of the source device (`IoT_Garage_Door`).


2. **Target Transfer Model (`GRUtargetmodel`):**


* Freezing/transferring learned layers and weights from `GRUbasemodel`.
* Addition of a final Dense layer with Sigmoid activation function on top of the model.
* Fine-tuning and inference on the unlabeled target device (`IoT_Thermostat`).
* **Comparative Evaluation Metrics against Baseline:**
* **Performance Gain Matrix:** Evaluation of accuracy jump on the target device (evolving from ~69.20% without DTL up to 99.76% with DTL).
* **Direct Comparison Table:**
* Accuracy, Precision, Recall, and F1-Score of DTL-GRU vs. Baseline Decision Tree.
* Time Comparison: Training Time (s) and Inference/Prediction Time (s).
