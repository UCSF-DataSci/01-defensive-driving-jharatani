# Part 2c: Config-Driven Development

Load and process patient data with BMI calculations.

**Your task:** Load configuration from `config.yaml` instead of hardcoding values.

---

## Load configuration


```python
import pandas as pd
import yaml
from pathlib import Path

from pathlib import Path
import yaml

CONFIG_PATH = Path("config.yaml")

with CONFIG_PATH.open() as f:
    config = yaml.safe_load(f) 

print(config)
print("BMI thresholds:", config["bmi_thresholds"])

# Example structure you'll get:
# config = {
#     "data": {"input_file": "data/patient_intake.csv"},
#     "bounds": {
#         "weight_kg": {"min": 30, "max": 250},
#         "height_cm": {"min": 120, "max": 230},
#         "age": {"min": 0, "max": 110}
#     },
#     "bmi_thresholds": {
#         "underweight": 18.5,
#         "normal": 25,
#         "overweight": 30
#     }
# }
```

    {'data': {'input_file': 'data/patient_intake.csv'}, 'bounds': {'weight_kg': {'min': 30, 'max': 250}, 'height_cm': {'min': 120, 'max': 230}, 'age': {'min': 0, 'max': 110}}, 'bmi_thresholds': {'underweight': 18.5, 'normal': 25, 'overweight': 30}}
    BMI thresholds: {'underweight': 18.5, 'normal': 25, 'overweight': 30}


---

## Load data


```python
# TODO: Replace hardcoded path with config["data"]["input_file"]
df = pd.read_csv(config["data"]["input_file"])

df.head()

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>patient_id</th>
      <th>first_name</th>
      <th>last_name</th>
      <th>weight_kg</th>
      <th>height_cm</th>
      <th>age</th>
      <th>sex</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>P001</td>
      <td>Mark</td>
      <td>Johnson</td>
      <td>91.5</td>
      <td>177</td>
      <td>46</td>
      <td>M</td>
    </tr>
    <tr>
      <th>1</th>
      <td>P002</td>
      <td>Donald</td>
      <td>Walker</td>
      <td>80.5</td>
      <td>164</td>
      <td>29</td>
      <td>M</td>
    </tr>
    <tr>
      <th>2</th>
      <td>P003</td>
      <td>Nancy</td>
      <td>Rhodes</td>
      <td>74.3</td>
      <td>163</td>
      <td>47</td>
      <td>F</td>
    </tr>
    <tr>
      <th>3</th>
      <td>P004</td>
      <td>Steven</td>
      <td>Miller</td>
      <td>64.4</td>
      <td>171</td>
      <td>71</td>
      <td>M</td>
    </tr>
    <tr>
      <th>4</th>
      <td>P005</td>
      <td>Javier</td>
      <td>Johnson</td>
      <td>72.8</td>
      <td>178</td>
      <td>18</td>
      <td>M</td>
    </tr>
  </tbody>
</table>
</div>



---

## Calculate BMI


```python
df["height_m"] = df["height_cm"] / 100
df["bmi"] = df["weight_kg"] / (df["height_m"] ** 2)
df["bmi"] = df["bmi"].round(1)

df[["patient_id", "weight_kg", "height_cm", "bmi"]].head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>patient_id</th>
      <th>weight_kg</th>
      <th>height_cm</th>
      <th>bmi</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>P001</td>
      <td>91.5</td>
      <td>177</td>
      <td>29.2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>P002</td>
      <td>80.5</td>
      <td>164</td>
      <td>29.9</td>
    </tr>
    <tr>
      <th>2</th>
      <td>P003</td>
      <td>74.3</td>
      <td>163</td>
      <td>28.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>P004</td>
      <td>64.4</td>
      <td>171</td>
      <td>22.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>P005</td>
      <td>72.8</td>
      <td>178</td>
      <td>23.0</td>
    </tr>
  </tbody>
</table>
</div>



---

## Categorize BMI


```python
# TODO: Replace hardcoded thresholds with values from config["bmi_thresholds"]
#       Use: underweight, normal, overweight thresholds from config
#       Bins should be: [0, underweight, normal, overweight, inf]

bmi_cfg = config["bmi_thresholds"]

df["bmi_category"] = pd.cut(
    df["bmi"],
    bins=[
        0,
        bmi_cfg["underweight"],
        bmi_cfg["normal"],
        bmi_cfg["overweight"],
        float("inf")
    ],
    labels=["Underweight", "Normal", "Overweight", "Obese"],
    right=False
)

df[["patient_id", "bmi", "bmi_category"]].head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>patient_id</th>
      <th>bmi</th>
      <th>bmi_category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>P001</td>
      <td>29.2</td>
      <td>Overweight</td>
    </tr>
    <tr>
      <th>1</th>
      <td>P002</td>
      <td>29.9</td>
      <td>Overweight</td>
    </tr>
    <tr>
      <th>2</th>
      <td>P003</td>
      <td>28.0</td>
      <td>Overweight</td>
    </tr>
    <tr>
      <th>3</th>
      <td>P004</td>
      <td>22.0</td>
      <td>Normal</td>
    </tr>
    <tr>
      <th>4</th>
      <td>P005</td>
      <td>23.0</td>
      <td>Normal</td>
    </tr>
  </tbody>
</table>
</div>



---

## Summary statistics


```python
summary = df.groupby("bmi_category")["patient_id"].count()
print("\nBMI category distribution:")
print(summary)

# TODO: Replace hardcoded 30 with config["bmi_thresholds"]["overweight"]
threshold = config["bmi_thresholds"]["overweight"]
high_risk = (df["bmi"] >= threshold).sum()
print(f"\nHigh-risk patients (BMI > {threshold}): {high_risk}")
```

    
    BMI category distribution:
    bmi_category
    Underweight     0
    Normal         15
    Overweight     21
    Obese          14
    Name: patient_id, dtype: int64
    
    High-risk patients (BMI > 30): 14


    /tmp/ipykernel_43115/1888430307.py:1: FutureWarning: The default of observed=False is deprecated and will be changed to True in a future version of pandas. Pass observed=False to retain current behavior or observed=True to adopt the future default and silence this warning.
      summary = df.groupby("bmi_category")["patient_id"].count()

