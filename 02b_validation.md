# Part 2b: Add Validation

Load and process patient data with BMI calculations.

**Your task:** Add schema and bounds validation to catch data quality issues early.

---

## Load data


```python
import pandas as pd
import logging 
from pathlib import Path
import os

df = pd.read_csv("data/patient_intake.csv")

# TODO: Define a function validate_schema(df, required_columns) that:
#       - Checks if all required columns are present
#       - Raises ValueError with list of missing columns if any are missing

required_columns = {"patient_id", "weight_kg", "height_cm", "age"}

bounds_dict = {
    "weight_kg": (30, 250),
    "height_cm": (120, 230),
    "age": (0, 110)}

def validate_schema(df, required_columns):
     missing = set(required_columns) - set(df.columns)

     if missing: 
          raise ValueError(f"Missing required columns: {sorted(missing)}")

# TODO: Define a function validate_bounds(df, bounds_dict) that:
#       - For each column in bounds_dict, check if values are within (min, max)
#       - Use df[col].between(min, max) to find out-of-bounds values
#       - Raises ValueError showing patient_id and value for any out-of-bounds rows

def validate_bounds(df, bounds_dict):
     for col, (min_val, max_val) in bounds_dict.items():
          in_bounds = df[col].between(min_val, max_val)
          out_of_bounds = df.loc[~in_bounds, ["patient_id", col]]

          if not out_of_bounds.empty:
               raise ValueError(
                    f"Out-of-bounds values in column '{col}' :\n"
                    f"{out_of_bounds.to_string(index = False)}"
               ) 

# TODO: Call validate_schema() to check for required columns:

validate_schema(df, required_columns)

# TODO: Call validate_bounds() with bounds

validate_bounds(df, bounds_dict)


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
df["bmi_category"] = pd.cut(
    df["bmi"],
    bins=[0, 18.5, 25, 30, float("inf")],
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

high_risk = df[df["bmi"] > 30]
print(f"\nHigh-risk patients (BMI > 30): {len(high_risk)}")
```

    
    BMI category distribution:
    bmi_category
    Underweight     0
    Normal         15
    Overweight     21
    Obese          14
    Name: patient_id, dtype: int64
    
    High-risk patients (BMI > 30): 14


    /tmp/ipykernel_14688/1212969269.py:1: FutureWarning: The default of observed=False is deprecated and will be changed to True in a future version of pandas. Pass observed=False to retain current behavior or observed=True to adopt the future default and silence this warning.
      summary = df.groupby("bmi_category")["patient_id"].count()

