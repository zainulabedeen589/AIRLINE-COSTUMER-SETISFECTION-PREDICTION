# ✈️ Airline Customer Satisfaction Prediction

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.9%2B-blue?style=for-the-badge&logo=python" />
  <img src="https://img.shields.io/badge/LightGBM-Classifier-brightgreen?style=for-the-badge" />
  <img src="https://img.shields.io/badge/MLflow-Tracking-orange?style=for-the-badge&logo=mlflow" />
  <img src="https://img.shields.io/badge/Flask-Web%20App-red?style=for-the-badge&logo=flask" />
  <img src="https://img.shields.io/badge/TensorBoard-Visualization-yellow?style=for-the-badge&logo=tensorflow" />
</p>

---

## 📌 Project Overview

Yeh project ek **end-to-end Machine Learning pipeline** hai jo airline customers ki satisfaction predict karta hai — yani yeh classify karta hai ki koi customer **satisfied** hai ya **neutral/dissatisfied**.

Isme complete MLOps workflow implement kiya gaya hai, jisme shaamil hai:
- Structured data ingestion
- Automated data processing & feature engineering
- Multi-model selection with TensorBoard logging
- Hyperparameter tuning with GridSearchCV
- MLflow experiment tracking
- Flask-based web application for real-time prediction

---

## 🗂️ Project Structure

```
AIRLINE COSTUMER SETISFECTION PREDICTION/
│
├── app.py                    # Flask web application (real-time prediction)
├── main.py                   # Master pipeline runner
├── setup.py                  # Package setup
├── requirements.txt          # Python dependencies
│
├── src/                      # Core source modules
│   ├── __init__.py
│   ├── paths_config.py       # Centralized path management
│   ├── logger.py             # Custom logging setup
│   ├── custom_exception.py   # Custom exception handler
│   ├── database_extraction.py# MySQL data extraction utility
│   ├── data_ingestion.py     # Data loading & train/test split
│   ├── data_processing.py    # Data cleaning & preprocessing
│   ├── feature_engineering.py# Feature construction & selection
│   ├── model_selection.py    # Multi-model benchmarking
│   └── model_training.py     # Final model training with MLflow
│
├── config/
│   ├── db_config.py          # MySQL database configuration
│   └── params.json           # LightGBM hyperparameter grid
│
├── data/
│   ├── train.csv             # Raw training data
│   └── test.csv              # Raw test data
│
├── artifact/
│   ├── raw/                  # Raw data copy
│   ├── ingested_data/        # Ingested train/test splits
│   ├── processed_data/       # Cleaned & processed data
│   ├── engineered_data/      # Feature-engineered final dataset
│   └── models/               # Saved trained model (.pkl)
│
├── templates/                # HTML templates for Flask app
├── static/                   # CSS/JS static files for Flask
├── logs/                     # Application logs
├── mlruns/                   # MLflow experiment tracking data
├── tensorboard_logs/         # TensorBoard run logs
└── NOTEBOOK.ipynb            # Exploratory Data Analysis notebook
```

---

## 📊 Dataset

| Property       | Details                                  |
|----------------|------------------------------------------|
| **Source**     | Airline Passenger Satisfaction Survey    |
| **Train Size** | ~103,000+ rows                           |
| **Test Size**  | ~25,000+ rows                            |
| **Target**     | `satisfaction` (satisfied / neutral or dissatisfied) |

### Key Features Used (Top 12 by Mutual Information):

| Feature                   | Type       |
|---------------------------|------------|
| Online Boarding           | Categorical |
| Inflight wifi service     | Categorical |
| Class                     | Categorical |
| Type of Travel            | Categorical |
| Inflight entertainment    | Categorical |
| Seat comfort              | Categorical |
| Leg room service          | Categorical |
| On-board service          | Categorical |
| Cleanliness               | Categorical |
| Ease of Online Booking    | Categorical |
| Flight Distance           | Numerical  |
| Delay Ratio               | Engineered |

---

## 🔁 ML Pipeline — Step by Step

### 1️⃣ Data Ingestion (`data_ingestion.py`)
- Raw CSV data load karta hai
- 80/20 train-test split karta hai
- `artifact/ingested_data/` mein save karta hai

### 2️⃣ Data Processing (`data_processing.py`)
- Unnecessary columns drop karta hai (`id`, `index`)
- Outlier handling IQR method se karta hai in-place clipping ke saath
- Missing values fill karta hai median se (`Arrival Delay in Minutes`)
- Processed data `artifact/processed_data/` mein save karta hai

### 3️⃣ Feature Engineering (`feature_engineering.py`)
- **New Features Construct karta hai:**
  - `Total Delay` = Departure Delay + Arrival Delay
  - `Delay Ratio` = Total Delay / (Flight Distance + 1)
- **Age Binning:** Child, Youngster, Adult, Senior
- **Label Encoding:** Gender, Customer Type, Type of Travel, Class, Age Group, satisfaction
- **Feature Selection:** Top 12 features Mutual Information se select karta hai
- Final dataset `artifact/engineered_data/final_df.csv` mein save karta hai

### 4️⃣ Model Selection (`model_selection.py`)
10 classic ML models ko benchmark karta hai:

| Model                    |
|--------------------------|
| Logistic Regression      |
| Random Forest            |
| Gradient Boosting        |
| AdaBoost                 |
| Support Vector Classifier|
| K-Nearest Neighbors      |
| Naive Bayes              |
| Decision Tree            |
| **LightGBM** ✅ (Best)   |
| XGBoost                  |

Har model ka **Accuracy, Precision, Recall, F1-Score** aur **Confusion Matrix** TensorBoard mein log hota hai.

### 5️⃣ Model Training (`model_training.py`)
- **Algorithm:** LightGBM Classifier
- **Hyperparameter Tuning:** GridSearchCV (3-fold CV)
- **Hyperparameter Grid** (`config/params.json`):
  ```json
  {
    "learning_rate": [0.01, 0.05, 0.1],
    "n_estimators": [100, 200, 300],
    "max_depth": [5, 10, 15]
  }
  ```
- **MLflow Tracking:**
  - Grid params & best params log hote hain
  - Accuracy, Precision, Recall, F1-Score metrics log hoti hain
  - Confusion Matrix JSON artifact ke roop mein save hoti hai
  - Trained model `artifact/models/trained_model.pkl` mein save hota hai

---

## 🌐 Flask Web Application (`app.py`)

Real-time prediction ke liye ek web interface provide karta hai.

**Input Fields:**
- Online Boarding rating
- Inflight Wifi Service rating
- Class (Economy / Business / Eco Plus)
- Type of Travel (Business / Personal)
- Inflight Entertainment rating
- Seat Comfort rating
- Leg Room Service rating
- On-board Service rating
- Cleanliness rating
- Ease of Online Booking rating
- Flight Distance
- Departure Delay (minutes)
- Arrival Delay (minutes)

**Output:** Satisfied ✅ ya Neutral/Dissatisfied ❌

---

## ⚙️ Installation & Setup

### Prerequisites
- Python 3.9+
- MySQL (optional — database extraction ke liye)

### Step 1: Repository Clone Karein
```bash
git clone <your-repo-url>
cd "AIRLINE COSTUMER SETISFECTION PREDICTION"
```

### Step 2: Virtual Environment Banayein
```bash
python -m venv .venv
source .venv/bin/activate       # Mac/Linux
.venv\Scripts\activate          # Windows
```

### Step 3: Dependencies Install Karein
```bash
pip install -r requirements.txt
```

### Step 4: Package Install Karein (Editable Mode)
```bash
pip install -e .
```

---

## 🚀 Running the Project

### Full ML Pipeline Run Karein
```bash
python main.py
```
Yeh in steps ko sequentially execute karega:
1. Data Ingestion
2. Data Processing
3. Feature Engineering
4. Model Training

### Flask Web App Start Karein
```bash
python app.py
```
Browser mein open karein: **http://localhost:5001**

### MLflow UI Dekhein
```bash
mlflow ui
```
Browser mein open karein: **http://localhost:5000**

### TensorBoard Dekhein (Model Selection Comparison)
```bash
tensorboard --logdir=tensorboard_logs
```
Browser mein open karein: **http://localhost:6006**

---

## 🔬 Model Performance

LightGBM classifier ko GridSearchCV ke saath tune kiya gaya hai.  
Final metrics MLflow mein track kiye gaye:

| Metric    | Value (approx.) |
|-----------|-----------------|
| Accuracy  | ~95%+           |
| Precision | ~95%+           |
| Recall    | ~95%+           |
| F1-Score  | ~95%+           |

> 💡 Exact metrics dekhne ke liye `mlflow ui` command run karein.

---

## 📦 Dependencies

```
numpy
pandas
setuptools
mysql-connector-python
scikit-learn
matplotlib
seaborn
lightgbm
xgboost
tensorflow
torch
torchvision
torchaudio
tensorboard
mlflow
flask
joblib
```

---

## 🔧 Configuration

### Database Configuration (`config/db_config.py`)
Apni MySQL credentials yahan set karein agar database se data extract karna ho:
```python
DB_CONFIG = {
    "host": "localhost",
    "user": "your_username",
    "password": "your_password",
    "database": "your_database_name",
    "table_name": "your_table_name"
}
```

### Hyperparameter Grid (`config/params.json`)
LightGBM ke liye tuning parameters modify karein:
```json
{
    "learning_rate": [0.01, 0.05, 0.1],
    "n_estimators": [100, 200, 300],
    "max_depth": [5, 10, 15]
}
```

---

## 📁 Key Path Configurations (`src/paths_config.py`)

| Constant              | Path                                           |
|-----------------------|------------------------------------------------|
| `RAW_DATA_PATH`       | `artifact/raw/data.csv`                        |
| `TRAIN_DATA_PATH`     | `artifact/ingested_data/train.csv`             |
| `TEST_DATA_PATH`      | `artifact/ingested_data/test.csv`              |
| `PROCESSED_DATA_PATH` | `artifact/processed_data/processed_train.csv`  |
| `ENGINEERED_DATA_PATH`| `artifact/engineered_data/final_df.csv`        |
| `MODEL_SAVE_PATH`     | `artifact/models/trained_model.pkl`            |
| `PARAMS_PATH`         | `config/params.json`                           |

---

## 🏗️ Architecture Overview

```
Raw Data (CSV / MySQL)
        │
        ▼
  Data Ingestion
  (Train/Test Split 80/20)
        │
        ▼
  Data Processing
  (Drop Cols → Outlier Handling → Null Handling)
        │
        ▼
  Feature Engineering
  (Construction → Binning → Encoding → Selection)
        │
        ▼
  Model Selection (TensorBoard)
  (10 Models Benchmarked)
        │
        ▼
  Model Training (MLflow)
  (LightGBM + GridSearchCV)
        │
        ▼
  Trained Model (.pkl)
        │
        ▼
  Flask Web App
  (Real-time Prediction)
```

---

## 👨‍💻 Author

**Zainul**  
Data Science & Machine Learning Project  
📅 2024

---

## 📄 License

This project is for educational and portfolio purposes.



---

*Created by Zainul Abedeen*
