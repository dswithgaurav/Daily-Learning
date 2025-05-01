# 🚀 Deploying Machine Learning Models with ModelBit

This guide demonstrates how to train a simple Logistic Regression model in Python and deploy it using [ModelBit](https://modelbit.com/). ModelBit enables you to deploy Python functions and ML models to live REST APIs in just a few lines of code.

---

## 📦 Prerequisites

Before you begin, make sure you have:

- Python 3.11 installed
- Pip for installing packages
- A [ModelBit account](https://modelbit.com/)
- Your workspace set up and connected via `modelbit.login()`

Install the ModelBit SDK:

```bash
pip install modelbit
```

---

## 🧠 Step 1: Training the ML Model

We use a simple dataset with `Age` and `Salary` to predict whether a user bought a product or not (`Bought` column).

```python
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score

# Sample dataset
data = {
    'Age': [22, 25, 47, 52, 46, 56, 55, 60],
    'Salary': [20000, 25000, 47000, 52000, 48000, 60000, 58000, 62000],
    'Bought': [0, 0, 1, 1, 1, 1, 1, 1]
}
df = pd.DataFrame(data)

# Feature and label separation
X = df[['Age', 'Salary']]
y = df['Bought']

# Split the data
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Model training
model = LogisticRegression()
model.fit(X_train, y_train)

# Evaluate
y_pred = model.predict(X_test)
print("Predictions:", y_pred)
print("Accuracy:", accuracy_score(y_test, y_pred))
```

---

## 🧩 Step 2: Define the Inference Function

Wrap your model's prediction logic inside a function that accepts input in the required format.

```python
def my_lr_deployement(input_x: list):
    if isinstance(input_x, (int, float)):
        return model.predict([input_x])[0]
    else:
        return None
```

> Note: This logic currently expects a single list of numeric values. You may want to enhance it to handle batch inputs or validate shapes.

---

## 🚀 Step 3: Deploy the Model with ModelBit

```python
import modelbit
mb = modelbit.login()

mb.deploy(
    my_lr_deployement,
    python_packages=["scikit-learn==1.1.3"],
    python_version="3.11"
)
```

This will deploy the function as a REST API and return an endpoint you can use.

---

## 🌐 Step 4: Get the Endpoint and Test

You can test the endpoint via ModelBit’s SDK or with `curl`.

### Via SDK:
```python
modelbit.get_inference(
    region="us-east-1.aws",
    workspace="gauravkandel",
    deployment="my_lr_deployement",
    version=4,
    data=[[10, 20]]
)
```

### Via `curl`:
```bash
curl -s -XPOST "https://gauravkandel.us-east-1.aws.modelbit.com/v1/my_lr_deployement/latest" \
  -d '{"data": [[20, 20000]]}' | json_pp
```

> Ensure the input format matches the expected structure in your inference function.

---

## 💾 Step 5: Add and Use the Model Object

ModelBit also allows you to store the model itself:

```python
mb.add_model("first_model", model)
mod = mb.get_model('first_model')

# Prediction
mod.predict([[10, 10000]])[0]
```

---

## 🛠 Improvements and Best Practices

- **Input Validation**: Add checks for array shape, missing values, etc.
- **Batch Predictions**: Modify the inference function to handle multiple rows.
- **Logging**: Log inputs and outputs for traceability.
- **Version Control**: Track model and deployment versions properly using ModelBit's features.
- **Monitoring**: Integrate with observability tools to track model drift and API usage.

---

## 📎 Summary

| Step              | Description                                      |
|-------------------|--------------------------------------------------|
| Data Preparation  | Create a simple dataset with `Age` and `Salary` |
| Model Training    | Train a Logistic Regression classifier          |
| Define Inference  | Wrap logic in a callable function               |
| Deploy to ModelBit| Expose model via REST API                       |
| Call Inference API| Use SDK or `curl` to call live endpoint         |

---
