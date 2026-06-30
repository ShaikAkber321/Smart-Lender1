# Smart Lender — Loan Eligibility Prediction

[![Live Demo](https://img.shields.io/badge/Live-Demo-brightgreen)](https://smart-lender-loan-eligibility-tester.onrender.com/)

Smart Lender is a machine-learning web application that predicts loan
approval likelihood from applicant data (income, employment, credit
history, education, property area, etc.), so credit officers and analysts
can fast-track low-risk applications and flag high-risk ones for review.

## 🌐 Live Demo
Try it here: **https://smart-lender-loan-eligibility-tester.onrender.com/**

> Note: the app is hosted on Render's free tier, so the first request after
> a period of inactivity may take 30–60 seconds to spin up.

This implementation follows the project's architecture end-to-end:

```
Dataset Collection
   -> Data Preprocessing (missing values, encoding, outliers, SMOTE, scaling)
   -> Train-Test Split
   -> Model Training (Decision Tree, Random Forest, KNN, XGBoost)
   -> Model Evaluation (accuracy, confusion matrix, classification report, CV)
   -> Best Model Selection
   -> Model Storage (model/rdf.pkl)
   -> Flask Web App (Frontend templates -> routing -> prediction engine)
```

## 📂 GitHub Repository
https://github.com/ShaikAkber321/smart-lender1

## Project structure

```
smart-lender/
├── app.py                      # Flask application (routes + prediction logic)
├── train_model.py              # Full ML training pipeline
├── requirements.txt
├── data/
│   ├── generate_dataset.py     # Generates the synthetic applicant dataset
│   └── loan_data.csv           # The dataset used for training (generated)
├── model/
│   └── rdf.pkl                 # Saved best model bundle (model + scaler + encoders)
├── reports/                    # Auto-generated evaluation plots & metrics
│   ├── model_comparison.png
│   ├── confusion_matrix_best_model.png
│   ├── feature_importance.png
│   ├── model_evaluation_report.txt
│   └── metrics_summary.json
├── templates/
│   ├── layout.html
│   ├── home.html
│   ├── predict.html
│   └── submit.html
└── static/
    └── css/style.css
```

## About the dataset

No real dataset was supplied with this project, so `data/generate_dataset.py`
creates a **realistic synthetic** loan-applicant dataset (900 rows) with the
same schema as the well-known public loan-eligibility dataset: `Gender`,
`Married`, `Dependents`, `Education`, `Self_Employed`, `ApplicantIncome`,
`CoapplicantIncome`, `LoanAmount`, `Loan_Amount_Term`, `Credit_History`,
`Property_Area`, `Loan_Status`.

It deliberately includes the messiness real bank data has — missing values,
income/loan outliers, and class imbalance — so the preprocessing pipeline
(missing-value handling, outlier capping, encoding, SMOTE, scaling) has
genuine work to do, and a learnable signal (credit history is the dominant
predictor, as in real credit data) so the trained models are meaningful.

**If you have a real loan dataset** (e.g. a CSV with the same columns), just
replace `data/loan_data.csv` with it and re-run `train_model.py` — the
pipeline doesn't need any code changes as long as the column names match.

## Model results (this run)

| Model         | Train Accuracy | Test Accuracy | 5-Fold CV Accuracy |
|---------------|:--:|:--:|:--:|
| Decision Tree | ~82% | ~78% | ~78% |
| Random Forest | ~90% | ~81% | ~85% |
| KNN           | 100% | ~71% | ~81% |
| **XGBoost**   | **~96%** | **~76%** | **~86%** |

XGBoost is selected as the best model based on **5-fold cross-validation
accuracy** (not the held-out test score) — this is standard practice: it
keeps the test set strictly for final, unbiased reporting rather than for
choosing the model, avoiding test-set leakage into the selection decision.
Exact numbers will vary slightly run to run and dataset to dataset; see
`reports/metrics_summary.json` after training for this run's exact figures.

## Setup & run in VS Code

### 1. Prerequisites
- Python 3.10+ installed (3.12 recommended) — check with `python --version`
- VS Code with the Python extension installed

### 2. Open the project
Open the `smart-lender` folder in VS Code (`File > Open Folder...`).

### 3. Create a virtual environment
Open a terminal in VS Code (`` Ctrl+` ``) and run:

**Windows (PowerShell):**
```powershell
python -m venv venv
venv\Scripts\Activate.ps1
```

**macOS / Linux:**
```bash
python3 -m venv venv
source venv/bin/activate
```

In VS Code, select this environment as the interpreter:
`Ctrl+Shift+P` → "Python: Select Interpreter" → choose the `venv` one.

### 4. Install dependencies
```bash
pip install -r requirements.txt
```

### 5. (Re)generate the dataset — optional, already included
A dataset is already provided at `data/loan_data.csv`. To regenerate it:
```bash
python data/generate_dataset.py
```

### 6. Train the model
```bash
python train_model.py
```
This prints progress to the terminal, saves the best model to
`model/rdf.pkl`, and writes evaluation plots/reports to `reports/`.
A trained model is already included, so this step is optional unless you
change the dataset or pipeline.

### 7. Run the web app
```bash
python app.py
```
You should see:
```
 * Running on http://127.0.0.1:5000
```
Open that address in your browser (Chrome/Edge recommended).

### 8. Use the app
- **Home** (`/`) — overview and model stats
- **Check Eligibility** (`/predict`) — fill in applicant details
- Submitting the form runs the saved model and shows **Approved/Declined**
  with a confidence score on the results page.

## Troubleshooting

**`ModuleNotFoundError: No module named 'flask'` (or similar)**
Your virtual environment isn't activated, or dependencies aren't installed.
Re-run steps 3–4 above. Make sure VS Code's selected interpreter is the
`venv` one (bottom-right of the VS Code window shows the active interpreter).

**`FileNotFoundError: Could not find .../model/rdf.pkl`**
Run `python train_model.py` first — the Flask app loads this file at startup.

**`OSError: [Errno 98] Address already in use` (port 5000 busy)**
Another process is already using port 5000. Either stop it, or run the app
on a different port:
```python
# at the bottom of app.py
app.run(debug=True, host="127.0.0.1", port=5001)
```

**`pip install xgboost` fails / wheel build errors**
Make sure pip itself is up to date first: `python -m pip install --upgrade pip`,
then retry `pip install -r requirements.txt`. XGBoost ships pre-built wheels
for all major platforms, so a source build should not normally be required.

**Predictions look wrong / KeyError when submitting the form**
This usually means `model/rdf.pkl` is out of sync with `app.py` (e.g. you
edited the feature columns in `train_model.py` but didn't retrain). Re-run
`python train_model.py` to regenerate a matching model bundle.

**Changes to templates/CSS not showing up**
Hard-refresh the browser (Ctrl+Shift+R) — static files can be cached.

## Notes on the pipeline design

- **Missing values**: categorical columns filled with the mode, numeric
  columns filled with the median.
- **Outlier handling**: income and loan-amount columns are capped (not
  dropped) using the IQR rule, so no applicants are discarded.
- **Encoding**: all categorical fields are label-encoded; the exact encoders
  fitted during training are saved and reused at prediction time, so the
  web form and the model always agree on what each category means.
- **SMOTE balancing**: applied only to the training split (never the test
  split) to avoid leaking information into evaluation.
- **Feature scaling**: a `StandardScaler` fitted on the training data is
  reused for both the test set and live predictions.
- **Model storage**: `model/rdf.pkl` is a single pickle file containing the
  trained model, the scaler, all encoders, and the exact feature-column
  order — so `app.py` only needs to load one file to make consistent
  predictions.

## Tech stack
Python · Flask · scikit-learn · XGBoost · imbalanced-learn (SMOTE) ·
pandas · NumPy · Matplotlib · Seaborn
