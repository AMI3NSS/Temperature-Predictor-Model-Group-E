# 🌡️ Classroom Temperature Predictor (Random Forest Regressor)

## 📖 Project Overview
This project uses Machine Learning to predict the localized indoor temperature of a classroom. It utilizes a **Random Forest Regressor** to analyze various environmental, temporal, and spatial factors (such as the number of people, class duration, outside temperature, and grid location) to accurately forecast the indoor temperature.

The pipeline is fully automated—handling everything from raw data cleaning and feature engineering to hyperparameter tuning and model evaluation.

---

## ⚙️ Model Architecture & Pipeline

### 1. Data Preprocessing & Feature Engineering
Before training, the script processes the raw CSV data using Scikit-Learn's `Pipeline` and `ColumnTransformer`:
* **Time Extraction:** Converts `Start Time` into a continuous `Start Hour` to capture time-of-day trends.
* **Duration Calculation:** Calculates `Class Duration (mins)` by finding the difference between `Start Time` and `Stop Time`.
* **Binary Mapping:** Converts the text-based `Window Open?` feature into standard binary (1 or 0).
* **Handling Missing Data:** Automatically imputes missing numeric data using the `median` and missing categorical data using the `most_frequent` value.
* **Scaling & Encoding:** Standardizes numeric features (`StandardScaler`) and One-Hot Encodes categorical features (`OneHotEncoder`).

### 2. The Machine Learning Algorithm
* **Algorithm:** `RandomForestRegressor`
* **Why Random Forest?** Random Forests excel at tabular data with complex, non-linear physical rules (e.g., "If windows are open AND it is the afternoon, temp changes by X"). 
* **Hyperparameter Tuning:** The script uses `GridSearchCV` with 3-fold Cross-Validation to automatically find the optimal number of trees (`n_estimators`), tree depth (`max_depth`), and split criteria (`min_samples_split`).

---

## 📊 Input Data Requirements

The script expects a dataset (CSV) containing the following target and feature columns:

**Target Variable:**
* `Indoor Temp (°C)`

**Input Features:**
* `Day` (Categorical)
* `Session` (Categorical)
* `Start Time` & `Stop Time` (Used to calculate Hour and Duration)
* `Outside Temp (°C)` (Numeric)
* `Total People in Classroom` (Numeric)
* `Grid` (Categorical - specific zone in the classroom)
* `Corner` (Categorical - specific corner in the grid)
* `Humidity (%)` (Numeric)
* `Window Open?` (Categorical: Yes/No)

---

## 🚀 How to Run the Code

### Prerequisites
Make sure you have the required Python libraries installed. You can install them via pip:
```bash
pip install pandas numpy scikit-learn

### Target Variable (What the model is predicting):
* **`Indoor Temp (°C)`**: The actual recorded temperature inside the classroom (Float).

### Input Features (What the model uses to make predictions):
The dataset must contain the following columns:

| Column Name | Data Type | Description |
| :--- | :--- | :--- |
| **Day** | Text | Day of the week (e.g., Monday, Tuesday). |
| **Session** | Text | Time of day session (e.g., Morning, Afternoon). |
| **Start Time** | Time | The time the class or recording session started (HH:MM:SS). |
| **Stop Time** | Time | The time the class or recording session ended (HH:MM:SS). |
| **Outside Temp (°C)**| Number | The recorded temperature outside the building. |
| **Total People in Classroom** | Number | The number of students/staff inside the classroom. |
| **Grid** | Text | The specific zone in the classroom (e.g., A, B, C, D). |
| **Corner** | Number | The specific corner in the grid (e.g., 1, 2, 3, 4). |
| **Humidity (%)** | Number | The humidity level inside the classroom. |
| **Window Open?** | Text | Whether the windows near the grid were open (Yes/No). |

---

## 3. Data Processing & Feature Engineering

Before training, the script automatically transforms the raw inputs via a Scikit-Learn `Pipeline`. It performs the following steps:

1. **Time Extraction:** Converts `Start Time` into a numeric **Start Hour** (e.g., 14:35:00 becomes 14).
2. **Duration Calculation:** Subtracts `Start Time` from `Stop Time` to calculate **Class Duration (mins)**.
3. **Binary Mapping:** Converts `Window Open?` from "Yes/No" text into `1` and `0`.
4. **Handling Missing Data:** Fills missing numbers with the *median* and missing text with the *mode*.
5. **One-Hot Encoding:** Converts categorical text columns (`Day`, `Session`, `Grid`, `Corner`) into separate binary columns.
6. **Scaling:** Standardizes numeric values so features are on the same mathematical scale.

---

## 4. Outputs

When the script finishes running, it produces:

### A. Console Output (Metrics)
* **Best Model Parameters:** The optimal settings found during GridSearch tuning.
* **Mean Absolute Error (MAE):** The average margin of error in degrees Celsius.
* **R-squared Score (R2):** A percentage representing how much of the temperature variance the model can explain.

### B. Exported File (The Model)
* **`classroom_temperature_model.joblib`**: This file contains the fully trained model and the entire preprocessing pipeline. 

---

## 5. How to Use the `.joblib` Model File

To use the saved model later to make a prediction, use the following Python code:

```python
import pandas as pd
import joblib

# 1. Load the saved model file
model = joblib.load('classroom_temperature_model.joblib')

# 2. Provide new input values
new_data = pd.DataFrame({
    'Day': ['Wednesday'],
    'Session': ['Morning'],
    'Start Hour': [9],
    'Class Duration (mins)': [60],
    'Outside Temp (°C)': [28.5],
    'Total People in Classroom': [30],
    'Grid': ['B'],
    'Corner': [2],
    'Humidity (%)': [55],
    'Window Open?': [1] # 1 for Yes, 0 for No
})

# 3. Get the prediction
prediction = model.predict(new_data)
print(f"Predicted Indoor Temperature: {prediction[0]:.2f} °C")
