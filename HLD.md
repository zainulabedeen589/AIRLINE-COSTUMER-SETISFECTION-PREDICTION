# ✈️ High-Level Design (HLD)
## Airline Customer Satisfaction Prediction System

---

<p align="center">
  <img src="https://img.shields.io/badge/Design-High%20Level-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Version-1.0-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Status-Production%20Ready-brightgreen?style=for-the-badge" />
</p>

---

## 1. 📌 Purpose & Scope

### 1.1 Purpose
Yeh document **Airline Customer Satisfaction Prediction System** ka High-Level Design present karta hai. Iska goal hai:
- System ka overall architecture define karna
- Major components aur unke interactions explain karna
- Data flow describe karna source se prediction tak
- Technology choices justify karna

### 1.2 Scope
Is system ka scope hai:
- Airline passenger survey data ko ingest karna (CSV ya MySQL)
- Data ko clean, process, aur feature-engineer karna
- Best ML model select karke train karna (LightGBM)
- Experiments track karna (MLflow + TensorBoard)
- Real-time web interface provide karna (Flask)

### 1.3 Target Audience
- Data Scientists
- ML Engineers
- System Architects
- Hiring Managers / Reviewers

---

## 2. 🏗️ System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AIRLINE SATISFACTION PREDICTION SYSTEM            │
│                                                                       │
│  ┌─────────────┐    ┌──────────────────────────────────────────┐     │
│  │  DATA LAYER │    │            ML PIPELINE LAYER             │     │
│  │             │    │                                          │     │
│  │  ┌────────┐ │    │  ┌─────────┐  ┌──────────┐  ┌────────┐ │     │
│  │  │  CSV   │─┼────┼─▶│Ingestion│─▶│Processing│─▶│Feature │ │     │
│  │  │ Files  │ │    │  └─────────┘  └──────────┘  │Engineer│ │     │
│  │  └────────┘ │    │                              └───┬────┘ │     │
│  │  ┌────────┐ │    │                                  │      │     │
│  │  │ MySQL  │─┼────┘                          ┌───────▼────┐ │     │
│  │  │   DB   │ │                               │   Model    │ │     │
│  │  └────────┘ │                               │  Training  │ │     │
│  └─────────────┘                               └───┬────────┘ │     │
│                                                    │           │     │
│  ┌─────────────────────────────────────────────────▼─────────┐│     │
│  │                   SERVING LAYER                            ││     │
│  │                                                            ││     │
│  │   ┌──────────────┐        ┌───────────────────────────┐   ││     │
│  │   │  Flask App   │◀───────│   Trained Model (.pkl)    │   ││     │
│  │   │  (Port 5001) │        └───────────────────────────┘   ││     │
│  │   └──────┬───────┘                                        ││     │
│  │          │ HTTP                                           ││     │
│  │   ┌──────▼───────┐                                        ││     │
│  │   │   Browser /  │                                        ││     │
│  │   │   End User   │                                        ││     │
│  │   └──────────────┘                                        ││     │
│  └────────────────────────────────────────────────────────────┘│     │
│                                                                 │     │
│  ┌──────────────────────────────────────────────────────────┐  │     │
│  │                MONITORING & TRACKING LAYER               │  │     │
│  │   ┌─────────────┐          ┌──────────────────────────┐  │  │     │
│  │   │   MLflow    │          │       TensorBoard        │  │  │     │
│  │   │  (Port 5000)│          │       (Port 6006)        │  │  │     │
│  │   └─────────────┘          └──────────────────────────┘  │  │     │
│  └──────────────────────────────────────────────────────────┘  │     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. 🔄 High-Level Data Flow

```
                        ┌─────────────────┐
                        │   Data Source   │
                        │  (CSV / MySQL)  │
                        └────────┬────────┘
                                 │ Raw Data
                                 ▼
                        ┌─────────────────┐
                        │  Data Ingestion │
                        │  (80% Train /   │
                        │   20% Test)     │
                        └────────┬────────┘
                                 │ Split Data
                                 ▼
                        ┌─────────────────┐
                        │ Data Processing │
                        │ • Drop Cols     │
                        │ • Outlier Fix   │
                        │ • Null Handling │
                        └────────┬────────┘
                                 │ Clean Data
                                 ▼
                        ┌─────────────────┐
                        │   Feature       │
                        │   Engineering   │
                        │ • Construction  │
                        │ • Age Binning   │
                        │ • Encoding      │
                        │ • MI Selection  │
                        └────────┬────────┘
                                 │ Feature-Ready Data (Top 12)
                                 ▼
                   ┌─────────────────────────┐
                   │     Model Selection     │
                   │  (10 Models Compared)   │
                   │  TensorBoard Logging    │
                   └────────────┬────────────┘
                                │ Best: LightGBM
                                ▼
                   ┌─────────────────────────┐
                   │     Model Training      │
                   │  LightGBM + GridSearchCV│
                   │  MLflow Experiment Log  │
                   └────────────┬────────────┘
                                │ trained_model.pkl
                                ▼
                   ┌─────────────────────────┐
                   │     Flask Web App       │
                   │  Real-time Prediction   │
                   │  (Satisfied / Not)      │
                   └─────────────────────────┘
```

---

## 4. 🧩 Major Components

### 4.1 Data Layer

| Component | Description | Technology |
|-----------|-------------|------------|
| **CSV Data Source** | Raw airline survey data (train.csv / test.csv) | Pandas CSV I/O |
| **MySQL Database** | Optional production data source | mysql-connector-python |
| **Artifact Store** | Processed data files at each pipeline stage | Local Filesystem |

### 4.2 ML Pipeline Layer

| Component | Responsibility | Input → Output |
|-----------|---------------|----------------|
| **Data Ingestion** | Load raw data, create train/test split | Raw CSV → Ingested CSVs |
| **Data Processing** | Clean, fix outliers, handle nulls | Ingested → Processed CSV |
| **Feature Engineering** | Construct, encode, select features | Processed → Engineered CSV |
| **Model Selection** | Benchmark 10 ML models | Engineered → Comparison Report |
| **Model Training** | Train LightGBM with tuning | Engineered → `.pkl` Model |

### 4.3 Serving Layer

| Component | Responsibility | Technology |
|-----------|---------------|------------|
| **Flask App** | REST web server for predictions | Flask (Python) |
| **HTML Templates** | Frontend user interface | Jinja2 Templates |
| **Model Loader** | Load saved model at startup | joblib |

### 4.4 Monitoring & Tracking Layer

| Component | Purpose | Access |
|-----------|---------|--------|
| **MLflow** | Track experiments, metrics, params, artifacts | `http://localhost:5000` |
| **TensorBoard** | Visualize model comparison & confusion matrices | `http://localhost:6006` |
| **Custom Logger** | File-based structured logging across modules | `logs/` directory |

---

## 5. 🛠️ Technology Stack

### 5.1 Core Technologies

| Category | Technology | Justification |
|----------|-----------|---------------|
| **Language** | Python 3.9+ | Industry standard for ML/AI |
| **ML Framework** | Scikit-learn | Consistent API across models |
| **Boosting Model** | LightGBM | Fast, accurate on tabular data |
| **Alternative Models** | XGBoost, RandomForest, etc. | Comparative benchmarking |
| **Web Framework** | Flask | Lightweight Python web server |
| **Experiment Tracking** | MLflow | Industry-standard ML tracking |
| **Visualization** | TensorBoard | Deep model comparison visuals |

### 5.2 Data & Storage Technologies

| Category | Technology | Purpose |
|----------|-----------|---------|
| **Data Manipulation** | Pandas, NumPy | Data transformation |
| **Serialization** | joblib | Model save/load |
| **Database** | MySQL | Optional production data source |
| **File Format** | CSV, JSON, PKL | Data & config storage |

### 5.3 Deep Learning Stack (For Future Extension)

| Technology | Purpose |
|-----------|---------|
| PyTorch | Neural Network models |
| TorchVision / TorchAudio | Multimodal future support |
| TensorFlow | Alternative DL framework |

---

## 6. 🔒 Non-Functional Requirements

### 6.1 Scalability
- Pipeline is modular — each step replaceable independently
- MySQL support allows scaling data source beyond CSV files
- LightGBM scales well to millions of rows

### 6.2 Maintainability
- Central `paths_config.py` — change paths in one place
- Custom exception system with traceback details
- Structured logging across all modules

### 6.3 Reproducibility
- MLflow tracks every experiment with params + metrics + model
- `random_state=42` used consistently for deterministic splits
- `params.json` externalizes hyperparameter configuration

### 6.4 Observability
- MLflow UI for experiment history
- TensorBoard for visual model comparison
- File-based logs in `logs/` for audit trail

---

## 7. 📦 Deployment Architecture

```
┌──────────────────────────────────────┐
│          Local Development           │
│                                      │
│  python main.py  ──▶  Training       │
│  python app.py   ──▶  Flask (5001)  │
│  mlflow ui       ──▶  MLflow (5000) │
│  tensorboard     ──▶  TB (6006)     │
└──────────────────────────────────────┘

Future Production Deployment:
┌──────────────────────────────────────┐
│         Docker / Cloud               │
│                                      │
│  ┌──────────┐   ┌────────────────┐  │
│  │  Flask   │   │  MLflow Server │  │
│  │ Container│   │   (Remote)     │  │
│  └────┬─────┘   └────────────────┘  │
│       │ NGINX / Load Balancer        │
│  ┌────▼─────────────────────────┐   │
│  │         End Users            │   │
│  └──────────────────────────────┘   │
└──────────────────────────────────────┘
```

---

## 8. 🔁 Pipeline Execution Flow

```
main.py
   │
   ├── 1. DataIngestion.create_ingested_data_dir()
   ├── 2. DataIngestion.split_data() → train.csv, test.csv
   │
   ├── 3. DataProcessor.run()
   │       ├── load_data()
   │       ├── drop_unnecessary_columns()
   │       ├── handle_outliers()
   │       ├── handle_null_values()
   │       └── save_data()
   │
   ├── 4. FeatureEngineer.run()
   │       ├── load_data()
   │       ├── feature_construction()
   │       ├── bin_age()
   │       ├── label_encoding()
   │       ├── feature_selection()
   │       └── save_data()
   │
   └── 5. ModelTraining.run()
           ├── load_data()
           ├── split_data()
           ├── train_model() → GridSearchCV + LightGBM
           ├── evaluate_model()
           ├── save_model() → trained_model.pkl
           └── MLflow logging
```

---

## 9. 📊 Summary Table

| Aspect | Detail |
|--------|--------|
| **Problem Type** | Binary Classification |
| **Target Variable** | `satisfaction` (2 classes) |
| **Best Model** | LightGBM Classifier |
| **Tuning Method** | GridSearchCV (3-fold CV) |
| **Feature Count** | 12 (top by Mutual Information) |
| **Tracking Tool** | MLflow + TensorBoard |
| **Serving** | Flask REST API (Port 5001) |
| **Experiment DB** | mlflow.db (SQLite) |
| **Model Format** | `.pkl` (joblib) |

---

*Created by Zainul Abedeen*
