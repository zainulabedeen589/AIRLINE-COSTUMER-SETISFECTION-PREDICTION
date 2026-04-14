# 🔬 Low-Level Design (LLD)
## Airline Customer Satisfaction Prediction System

---

<p align="center">
  <img src="https://img.shields.io/badge/Design-Low%20Level-purple?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Version-1.0-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Language-Python%203.9%2B-blue?style=for-the-badge" />
</p>

---

## 1. 📌 Purpose

Yeh document system ke **internal implementation details** describe karta hai — classes, methods, logic, configurations, aur module interactions. Yeh HLD ka complement hai jo architecture ke andar ki mechanics explain karta hai.

---

## 2. 📁 Module & Class Design

---

### 2.1 `src/paths_config.py` — Centralized Path Registry

**Purpose:** Saari file paths ko ek jagah define karna taaki koi bhi module hardcoded paths use na kare.

```
paths_config.py
│
├── ARTIFACTS_DIR        = "./artifact"
├── RAW_DATA_PATH        = artifact/raw/data.csv
├── INGESTED_DATA_DIR    = artifact/ingested_data/
├── TRAIN_DATA_PATH      = artifact/ingested_data/train.csv
├── TEST_DATA_PATH       = artifact/ingested_data/test.csv
├── PROCESSED_DIR        = artifact/processed_data/
├── PROCESSED_DATA_PATH  = artifact/processed_data/processed_train.csv
├── ENGINEERED_DIR       = artifact/engineered_data/
├── ENGINEERED_DATA_PATH = artifact/engineered_data/final_df.csv
├── PARAMS_PATH          = config/params.json
└── MODEL_SAVE_PATH      = artifact/models/trained_model.pkl
```

**Design Decision:** `*` wildcard import se har module automatically yeh paths inherit karta hai bina duplication ke.

---

### 2.2 `src/logger.py` — Custom Logger

**Purpose:** Sabhi modules ke liye consistent, formatted logging provide karna.

#### Class / Function Design:
```
get_logger(name: str) -> logging.Logger
│
├── INPUT  : module ka __name__
├── OUTPUT : configured Logger instance
│
├── Handler    : StreamHandler (console output)
├── Log Level  : INFO
└── Format     : "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
```

#### Usage Pattern:
```python
from src.logger import get_logger
logger = get_logger(__name__)
logger.info("Pipeline started")
logger.error("Something failed")
```

---

### 2.3 `src/custom_exception.py` — Exception Handler

**Purpose:** Traceback ke saath meaningful error messages generate karna.

#### Class Design:
```
CustomException(Exception)
│
├── __init__(error_message: str, error_detail: sys)
│       └── Extracts: filename, line number from traceback
│
└── __str__() → formatted error string
        Format: "Error in [file] at line [N]: [message]"
```

---

### 2.4 `src/database_extraction.py` — MySQL Extractor

**Purpose:** MySQL database se data extract karke CSV mein save karna.

#### Class Design:
```
MySQLDataExtractor
│
├── __init__(db_config: dict)
│   ├── self.host         ← db_config["host"]
│   ├── self.user         ← db_config["user"]
│   ├── self.password     ← db_config["password"]
│   ├── self.database     ← db_config["database"]
│   ├── self.table_name   ← db_config["table_name"]
│   └── self.connection   = None
│
├── connect()
│   └── mysql.connector.connect(host, user, password, database)
│
├── dissconnect()
│   └── self.connection.close()
│
└── extract_to_csv(output_folder="./artifacts")
    ├── connect() if not connected
    ├── cursor.execute(f"SELECT * FROM {table_name}")
    ├── fetchall() → rows
    ├── csv.writer → write header + rows
    └── dissconnect()
```

#### Config Schema (`config/db_config.py`):
```python
DB_CONFIG = {
    "host"       : "localhost",
    "user"       : "root",
    "password"   : "your_password",
    "database"   : "airline_db",
    "table_name" : "passengers"
}
```

---

### 2.5 `src/data_ingestion.py` — Data Ingestion Module

**Purpose:** Raw CSV data load karna aur train/test split karke artifact directory mein save karna.

#### Class Design:
```
DataIngestion
│
├── __init__(raw_data_path: str, ingested_data_dir: str)
│   ├── self.raw_data_path     ← RAW_DATA_PATH
│   └── self.ingested_data_dir ← INGESTED_DATA_DIR
│
├── create_ingested_data_dir()
│   └── os.makedirs(ingested_data_dir, exist_ok=True)
│
└── split_data(train_path, test_path, test_size=0.2, random_state=42)
    ├── pd.read_csv(raw_data_path) → df
    ├── train_test_split(df, test_size=0.2, random_state=42)
    ├── train_data.to_csv(train_path, index=False)
    └── test_data.to_csv(test_path, index=False)
```

#### Data Flow:
```
RAW_DATA_PATH (data.csv)
       │
       ▼ pd.read_csv()
   DataFrame (shape: ~129,000 rows)
       │
       ▼ train_test_split(test_size=0.2, random_state=42)
   ┌───┴────────────────┐
   ▼                    ▼
train.csv (80%)     test.csv (20%)
~103,000 rows       ~25,000 rows
```

---

### 2.6 `src/data_processing.py` — Data Processor

**Purpose:** Raw ingested data ko clean karna.

#### Class Design:
```
DataProcessor
│
├── __init__()
│   ├── self.train_path          ← TRAIN_DATA_PATH
│   └── self.processed_data_path ← PROCESSED_DATA_PATH
│
├── load_data() → DataFrame
│   ├── pd.read_csv(train_path)
│   └── Force numeric conversion per column
│
├── drop_unnecessary_columns(df, columns: list) → DataFrame
│   └── df.drop(columns=["index", "id"])
│
├── handle_outliers(df, columns: list) → DataFrame
│   └── For each column:
│         Q1 = df[col].quantile(0.25)
│         Q3 = df[col].quantile(0.75)
│         IQR = Q3 - Q1
│         lower = Q1 - 1.5 * IQR
│         upper = Q3 + 1.5 * IQR
│         df[col] = df[col].clip(lower, upper)  ← IQR Clipping
│
├── handle_null_values(df, columns) → DataFrame
│   └── df[columns].fillna(df[columns].median())
│
├── save_data(df)
│   └── df.to_csv(processed_data_path, index=False)
│
└── run()  ← Orchestrator
    ├── load_data()
    ├── drop_unnecessary_columns(["index", "id"])
    ├── handle_outliers(['Flight Distance',
    │                    'Departure Delay in Minutes',
    │                    'Arrival Delay in Minutes',
    │                    'Checkin service'])
    ├── handle_null_values('Arrival Delay in Minutes')
    └── save_data()
```

#### Outlier Handling Logic (IQR Method):
```
Original Value:  ████████████████████████████████ [outlier: 2000 min]
After Clip:      ████████████ [upper_bound: ~350 min]

Lower Bound = Q1 - 1.5 * IQR
Upper Bound = Q3 + 1.5 * IQR
Values outside bounds are CLIPPED (not removed) to preserve row count.
```

---

### 2.7 `src/feature_engineering.py` — Feature Engineer

**Purpose:** Predictive features construct karna, encode karna, aur mutual information se select karna.

#### Class Design:
```
FeatureEngineer
│
├── __init__()
│   ├── self.data_path    ← PROCESSED_DATA_PATH
│   ├── self.df           = None
│   └── self.label_mapping = {}
│
├── load_data()
│   └── pd.read_csv(processed_data_path)
│
├── feature_construction()
│   ├── df['Total Delay'] = 'Departure Delay' + 'Arrival Delay'
│   └── df['Delay Ratio'] = 'Total Delay' / ('Flight Distance' + 1)
│           (+1 to avoid division-by-zero)
│
├── bin_age()
│   └── pd.cut(df['Age'],
│              bins=[0, 18, 30, 50, 100],
│              labels=['Child','Youngster','Adult','Senior'])
│
├── label_encoding()
│   ├── columns = ['Gender', 'Customer Type', 'Type of Travel',
│   │              'Class', 'satisfaction', 'Age Group']
│   └── label_encode(df, columns) → (df, label_mapping dict)
│
├── feature_selection() → Top 12 Features
│   ├── X = df.drop('satisfaction'), y = df['satisfaction']
│   ├── train_test_split(X, y, test_size=0.2)
│   ├── mutual_info_classif(X_train, y_train)
│   ├── Sort features by Mutual Information score
│   ├── top_features = top 12 features
│   └── df = df[top_features + ['satisfaction']]
│
├── save_data()
│   └── df.to_csv(ENGINEERED_DATA_PATH, index=False)
│
└── run()  ← Orchestrator
    ├── load_data()
    ├── feature_construction()
    ├── bin_age()
    ├── label_encoding()
    ├── feature_selection()
    └── save_data()
```

#### Feature Construction Detail:
```
Input Columns:
  Departure Delay in Minutes = 15
  Arrival Delay in Minutes   = 20
  Flight Distance            = 500

Computed:
  Total Delay  = 15 + 20 = 35
  Delay Ratio  = 35 / (500 + 1) = 0.0699
```

#### Label Encoding Mapping (Example):
```
Gender:          {'Male': 0, 'Female': 1}
Class:           {'Eco': 0, 'Eco Plus': 1, 'Business': 2}
Type of Travel:  {'Personal Travel': 0, 'Business travel': 1}
satisfaction:    {'neutral or dissatisfied': 0, 'satisfied': 1}
Age Group:       {'Child': 0, 'Youngster': 1, 'Adult': 2, 'Senior': 3}
```

#### Mutual Information Feature Selection:
```
All Features → Ranked by MI Score → Top 12 Selected

Top Features (Approximate Ranking):
  1. Online Boarding
  2. Inflight wifi service
  3. Class
  4. Type of Travel
  5. Inflight entertainment
  6. Seat comfort
  7. Leg room service
  8. On-board service
  9. Cleanliness
  10. Ease of Online Booking
  11. Flight Distance
  12. Delay Ratio
```

---

### 2.8 `src/model_selection.py` — Model Benchmarking

**Purpose:** 10 different ML models ko train aur compare karna taaki best algorithm identify ho.

#### Class Design:
```
ModelSelection
│
├── __init__(data_path: str)
│   ├── SummaryWriter(log_dir=f"tensorboard_logs/run_{timestamp}")
│   └── self.models = {
│         'Logistic Regression'      : LogisticRegression(),
│         'Random Forest'            : RandomForestClassifier(n_estimators=50),
│         'Gradient Boosting'        : GradientBoostingClassifier(n_estimators=50),
│         'AdaBoost'                 : AdaBoostClassifier(n_estimators=50),
│         'Support Vector Classifier': SVC(),
│         'K-Nearest Neighbors'      : KNeighborsClassifier(),
│         'Naive Bayes'              : GaussianNB(),
│         'Decision Tree'           : DecisionTreeClassifier(),
│         'LightGBM'                : LGBMClassifier(),
│         'XGBoost'                 : XGBClassifier(eval_metric='mlogloss')
│       }
│
├── load_data() → (X, y)
│   ├── pd.read_csv(data_path)
│   ├── df_sample = df.sample(frac=0.1)  ← 10% for speed
│   ├── X = df_sample.drop('satisfaction')
│   └── y = df_sample['satisfaction']
│
├── split_data(X, y) → (X_train, X_test, y_train, y_test)
│   └── train_test_split(test_size=0.2, random_state=42)
│
├── log_confusion_matrix(y_true, y_pred, step, model_name)
│   ├── confusion_matrix(y_true, y_pred)
│   ├── plt.matshow(cm, cmap=Blues)
│   └── writer.add_figure(f"Confusion_Matrix/{model_name}")
│
├── train_and_evaluate(X_train, X_test, y_train, y_test)
│   └── for each model:
│         model.fit(X_train, y_train)
│         y_pred = model.predict(X_test)
│         metrics = {accuracy, precision, recall, f1}
│         writer.add_scalar(f'Accuracy/{name}', accuracy, idx)
│         writer.add_scalar(f'Precision/{name}', precision, idx)
│         writer.add_scalar(f'Recall/{name}', recall, idx)
│         writer.add_scalar(f'F1_score/{name}', f1, idx)
│         log_confusion_matrix(...)
│
└── run()
    ├── load_data()
    ├── split_data()
    └── train_and_evaluate()
```

#### TensorBoard Metrics Logged:
| Metric | Tag Format |
|--------|-----------|
| Accuracy | `Accuracy/{ModelName}` |
| Precision | `Precision/{ModelName}` |
| Recall | `Recall/{ModelName}` |
| F1-Score | `F1_score/{ModelName}` |
| Confusion Matrix | `Confusion_Matrix/{ModelName}` |

---

### 2.9 `src/model_training.py` — Final Model Trainer

**Purpose:** LightGBM ko GridSearchCV se tune karna aur MLflow mein track karna.

#### Class Design:
```
ModelTraining
│
├── __init__(data_path, params_path, model_save_path,
│            experiment_name="Airline_Satisfaction_Prediction")
│   ├── self.best_model = None
│   ├── self.metrics = None
│   ├── tracking_uri = f"file://{project_root}/mlruns"
│   └── mlflow.set_tracking_uri(tracking_uri)
│
├── load_data() → DataFrame
│   └── pd.read_csv(data_path)
│
├── split_data(data) → (X_train, X_test, y_train, y_test)
│   ├── X = data.drop('satisfaction')
│   ├── y = data['satisfaction']
│   └── train_test_split(test_size=0.2, random_state=42)
│
├── train_model(X_train, y_train, params) → best_params dict
│   ├── lgbm = LGBMClassifier()
│   ├── GridSearchCV(lgbm, param_grid=params, cv=3, scoring='accuracy')
│   ├── grid_search.fit(X_train, y_train)
│   └── self.best_model = grid_search.best_estimator_
│
├── evaluate_model(X_test, y_test) → metrics dict
│   ├── y_pred = best_model.predict(X_test)
│   └── self.metrics = {
│           accuracy, precision, recall,
│           f1_score, confusion_matrix (list)
│         }
│
├── save_model()
│   ├── os.makedirs(dirname(model_save_path))
│   └── joblib.dump(best_model, model_save_path)
│
└── run()  ← Main MLflow Run
    ├── mlflow.set_experiment(experiment_name)
    ├── with mlflow.start_run():
    │     ├── load_data()
    │     ├── split_data()
    │     ├── json.load(params_path) → params
    │     ├── mlflow.log_params(grid_params)
    │     ├── train_model() → best_params
    │     ├── mlflow.log_params(best_params)
    │     ├── evaluate_model() → metrics
    │     ├── mlflow.log_metric(accuracy, precision, recall, f1)
    │     ├── save confusion_matrix.json → mlflow.log_artifact()
    │     ├── save_model() → .pkl
    │     └── mlflow.sklearn.log_model(best_model, "model")
    └── on failure: mlflow.end_run(status="FAILED")
```

#### Hyperparameter Grid (`config/params.json`):
```json
{
    "learning_rate" : [0.01, 0.05, 0.1],
    "n_estimators"  : [100, 200, 300],
    "max_depth"     : [5, 10, 15]
}
```

#### GridSearchCV Details:
```
Total Combinations = 3 × 3 × 3 = 27 combinations
CV Folds           = 3
Total Fits         = 27 × 3 = 81 model fits
Scoring Metric     = accuracy
```

#### MLflow Logged Items:
| Type | Key | Example Value |
|------|-----|--------------|
| Param | `grid_learning_rate` | `[0.01, 0.05, 0.1]` |
| Param | `best_learning_rate` | `0.05` |
| Param | `best_n_estimators` | `200` |
| Param | `best_max_depth` | `10` |
| Metric | `accuracy` | `0.9542` |
| Metric | `precision` | `0.9538` |
| Metric | `recall` | `0.9542` |
| Metric | `f1_score` | `0.9539` |
| Artifact | `confusion_matrix.json` | `[[TN, FP],[FN, TP]]` |
| Model | `model/` | Serialized LGBM model |

---

### 2.10 `app.py` — Flask Web Application

**Purpose:** Trained model ko expose karna real-time prediction ke liye via HTTP.

#### Application Design:
```
Flask App (Port: 5001)
│
├── Startup:
│   └── model = joblib.load(MODEL_SAVE_PATH)  ← loaded once
│
└── Route: GET/POST "/"   → home()
    │
    ├── GET  → render index.html (empty form)
    │
    └── POST → extract form fields:
            ├── departure_delay = float(form["Departure Delay"])
            ├── arrival_delay   = float(form["Arrival Delay"])
            ├── flight_distance = float(form["Flight Distance"])
            │
            ├── delay_ratio = (departure_delay + arrival_delay)
            │                 / (flight_distance + 1)
            │
            ├── data = [
            │     Online Boarding,     delay_ratio,
            │     Inflight wifi,       Class,
            │     Type of Travel,      Inflight entertainment,
            │     flight_distance,     Seat comfort,
            │     Leg room service,    On-board service,
            │     Cleanliness,         Ease of Online Booking
            │   ]  ← 12 features (matches model input)
            │
            ├── prediction = model.predict([data])
            └── render_template("index.html", prediction=output)
```

#### Input → Feature Vector Mapping:
```
Form Field                  →  Feature Index  →  Feature Name
─────────────────────────────────────────────────────────────
Online Boarding             →  [0]            →  Online Boarding
Departure+Arrival / Dist    →  [1]            →  Delay Ratio (engineered)
Inflight wifi service       →  [2]            →  Inflight wifi service
Class                       →  [3]            →  Class (encoded)
Type of Travel              →  [4]            →  Type of Travel (encoded)
Inflight entertainment      →  [5]            →  Inflight entertainment
Flight Distance             →  [6]            →  Flight Distance
Seat comfort                →  [7]            →  Seat comfort
Leg room service            →  [8]            →  Leg room service
On-board service            →  [9]            →  On-board service
Cleanliness                 →  [10]           →  Cleanliness
Ease of Online Booking      →  [11]           →  Ease of Online Booking
```

---

## 3. 🗄️ Artifact Directory Schema

```
artifact/
│
├── raw/
│   └── data.csv                      ← Original full dataset
│
├── ingested_data/
│   ├── train.csv                     ← 80% split (raw)
│   └── test.csv                      ← 20% split (raw)
│
├── processed_data/
│   └── processed_train.csv           ← After cleaning pipeline
│
├── engineered_data/
│   └── final_df.csv                  ← After FE (12 features + target)
│
└── models/
    └── trained_model.pkl             ← Final LightGBM model (joblib)
```

---

## 4. ⚙️ Configuration Files

### 4.1 `config/params.json`
```json
{
    "learning_rate" : [0.01, 0.05, 0.1],
    "n_estimators"  : [100, 200, 300],
    "max_depth"     : [5, 10, 15]
}
```

### 4.2 `config/db_config.py`
```python
DB_CONFIG = {
    "host"       : "localhost",
    "user"       : "root",
    "password"   : "password",
    "database"   : "airline_db",
    "table_name" : "passengers"
}
```

---

## 5. 🧬 Class Interaction Diagram

```
main.py
  │
  ├──▶ DataIngestion
  │        └── uses: RAW_DATA_PATH, INGESTED_DATA_DIR,
  │                  TRAIN_DATA_PATH, TEST_DATA_PATH
  │
  ├──▶ DataProcessor
  │        └── uses: TRAIN_DATA_PATH, PROCESSED_DATA_PATH
  │
  ├──▶ FeatureEngineer
  │        ├── uses: PROCESSED_DATA_PATH, ENGINEERED_DATA_PATH
  │        └── calls: label_encode() from utils/helpers.py
  │
  └──▶ ModelTraining
           ├── uses: ENGINEERED_DATA_PATH, PARAMS_PATH, MODEL_SAVE_PATH
           └── calls: mlflow (tracking), GridSearchCV, LGBMClassifier

app.py
  └──▶ loads trained_model.pkl
       └── serves predictions via Flask HTTP routes
```

---

## 6. 🔍 Error Handling Design

```
Each method follows this pattern:

try:
    logger.info("Starting [step]...")
    # ... core logic ...
    logger.info("[step] completed successfully.")
    return result

except Exception as e:
    logger.error(f"Error in [step]: {e}")
    raise CustomException(f"Descriptive error message", sys)
```

**CustomException** captures:
- Source filename
- Line number
- Original error message

**Logger** captures:
- Timestamp
- Module name
- Log level (INFO / ERROR)
- Message

---

## 7. 📋 Method Summary Table

| Module | Class | Key Methods | Returns |
|--------|-------|-------------|---------|
| `data_ingestion` | `DataIngestion` | `split_data()` | None (saves CSVs) |
| `data_processing` | `DataProcessor` | `handle_outliers()`, `handle_null_values()` | DataFrame |
| `feature_engineering` | `FeatureEngineer` | `feature_construction()`, `feature_selection()` | None (modifies df) |
| `model_selection` | `ModelSelection` | `train_and_evaluate()` | None (logs to TB) |
| `model_training` | `ModelTraining` | `train_model()`, `evaluate_model()` | dict (metrics) |
| `database_extraction` | `MySQLDataExtractor` | `extract_to_csv()` | None (saves CSV) |
| `app` | Flask | `home()` | HTML response |

---

## 8. 🚀 Execution Commands Reference

| Task | Command |
|------|---------|
| Run full ML pipeline | `python main.py` |
| Run only ingestion | `python -m src.data_ingestion` |
| Run only processing | `python -m src.data_processing` |
| Run only feature engineering | `python -m src.feature_engineering` |
| Run model selection (benchmark) | `python -m src.model_selection` |
| Run model training | `python -m src.model_training` |
| Start Flask app | `python app.py` |
| Open MLflow UI | `mlflow ui` → http://localhost:5000 |
| Open TensorBoard | `tensorboard --logdir=tensorboard_logs` → http://localhost:6006 |

---

*Created by Zainul Abedeen*
