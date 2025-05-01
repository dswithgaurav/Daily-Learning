

# 🧪 Testing ML Models in Production

This project demonstrates various strategies to test and safely roll out machine learning models in a production environment using **[Modelbit](https://www.modelbit.com/)**.

We implement **A/B Testing**, **Canary Deployment**, **Interleaved Deployment**, and **Shadow Deployment** techniques using `modelbit` with Scikit-learn models.



## 📦 Setup

Install Modelbit and dependencies:
```bash
pip install modelbit scikit-learn==1.1.1
```

Initialize Modelbit:
```python
import modelbit
mb = modelbit.login()
```



## ⚖️ A/B Testing

Randomly assign incoming requests to either the current or updated model.

> 🔸 Not recommended to split traffic equally. Start with a smaller percentage for the new model.

```python
mb.add_models({
    "LinearModel": model_lr,
    "ForestModel": model_rf
})

def ABTesting(a):
    if np.random.binomial(1, 0.1):  # 10% traffic to ForestModel
        model = mb.get_model("ForestModel")
        return model.predict([[a]])[0], "RF"
    
    model = mb.get_model("LinearModel")
    return model.predict([[a]])[0], "LR"

mb.deploy(
    ABTesting,
    python_packages=["scikit-learn==1.1.1"],
    python_version="3.9"
)

modelbit.get_inference(
    workspace="dswithgaurav",
    deployment="ABTesting",
    data=[[1, 2], [2, 3], [3, 2]]
)
```



## 🐤 Canary Deployment

Release the updated model to a small, specific subset of users.

> 🔸 Unlike A/B testing, not all users are exposed to both models.

```python
def CanaryDeployment(a, user):
    if user in ["A", "B", "D", "E"]:
        model = mb.get_model("ForestModel")
        return model.predict([[a]])[0], "RF"
    
    model = mb.get_model("LinearModel")
    return model.predict([[a]])[0], "LR"
```



## ♻️ Interleaved Deployment

Expose users to both models simultaneously by combining their outputs.

> 🔸 Useful for use-cases like recommendations where multiple results are shown.

```python
def InterleavedDeployment(a):
    rf_pred = mb.get_model("ForestModel").predict([[a]])[0]
    lr_pred = mb.get_model("LinearModel").predict([[a]])[0]
    return (rf_pred + lr_pred) / 2
```



## 👻 Shadow Deployment

Run the new model in the background without affecting user experience.

> 🔸 Useful for monitoring model behavior without business impact.

```python
def ShadowLegacyModel(a):
    return mb.get_model("ForestModel").predict([[a]])[0]

def ShadowCandidateModel(a):
    return mb.get_model("LinearModel").predict([[a]])[0]
```


## ✅ Summary

| Strategy             | Live Traffic Impact | User Sees New Model? | Suitable For              |
|----------------------|---------------------|------------------------|----------------------------|
| A/B Testing          | Yes (some %)        | Yes (some users)       | Direct comparison         |
| Canary Deployment    | Yes (targeted)      | Yes (few users)        | Gradual rollout           |
| Interleaved Deployment | Yes               | Yes (blended output)   | Recommendations, ranking  |
| Shadow Deployment    | No                  | No                     | Safe monitoring           |

---