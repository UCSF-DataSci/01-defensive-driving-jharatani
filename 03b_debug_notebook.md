# Assignment Part 3b: Debug Lab Results Analysis

This notebook analyzes patient glucose test results to identify diabetes risk. It contains hidden issues you need to uncover using VS Code's notebook debugger.

**Your task:**

- Use the debug icon to run cells interactively
- Set breakpoints and inspect variables
- Fix the issues and add concise comments explaining each change
- Restart the kernel + Run All to verify everything works

---

## Setup and load data


```python
import pandas as pd
from pathlib import Path

# Load patient data
data_path = Path("data/patient_intake.csv")
patients = pd.read_csv(data_path)

print(f"Loaded {len(patients)} patients")
patients.head()
```

    Loaded 50 patients





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

## Calculate fasting glucose estimates


```python
print("Estimating fasting glucose from BMI and age...")

# Simple glucose estimation for demonstration
# (In reality, would come from lab tests)
patients["glucose_mg_dl"] = (patients["weight_kg"] * 1.2 + patients["age"] * 0.3).round(0)

# Convert to string for "display formatting"
patients["glucose_mg_dl"] = patients["glucose_mg_dl"].astype(str)

print(f"Glucose calculated for {len(patients)} patients")
print(f"Sample values:")
print(patients[["patient_id", "glucose_mg_dl"]].head())

# Ensure at least one diabetes-range value appears for testing/coverage
patients.loc[patients.index[0], "glucose_mg_dl"] = 130.0
```

    Estimating fasting glucose from BMI and age...
    Glucose calculated for 50 patients
    Sample values:
      patient_id glucose_mg_dl
    0       P001         124.0
    1       P002         105.0
    2       P003         103.0
    3       P004          99.0
    4       P005          93.0


---

## Categorize diabetes risk


```python
patients["glucose_mg_dl"] = pd.to_numeric(
    patients["glucose_mg_dl"],
    errors="coerce"
) #Had to make sure all values in column are numeric

print("\nCategorizing diabetes risk based on fasting glucose...")

def categorize_glucose(glucose_value):
    """Categorize diabetes risk from fasting glucose (mg/dL)."""
    if pd.isna(glucose_value): #accounting for na values here
        return "Unknown"
    elif glucose_value < 100: #this was unable to process because some glucose values were strings not integers, so accounted for non-numeric values and NA values
        return "Low risk (normal)"
    elif glucose_value < 126:
        return "High risk (prediabetes)"
    else:
        return "Very high risk (diabetes)"

patients["diabetes_risk"] = patients["glucose_mg_dl"].apply(categorize_glucose)

print("Risk categories assigned:")
print(patients[["patient_id", "glucose_mg_dl", "diabetes_risk"]].head(10))
```

    
    Categorizing diabetes risk based on fasting glucose...
    Risk categories assigned:
      patient_id  glucose_mg_dl              diabetes_risk
    0       P001          130.0  Very high risk (diabetes)
    1       P002          105.0    High risk (prediabetes)
    2       P003          103.0    High risk (prediabetes)
    3       P004           99.0          Low risk (normal)
    4       P005           93.0          Low risk (normal)
    5       P006          105.0    High risk (prediabetes)
    6       P007          120.0    High risk (prediabetes)
    7       P008          118.0    High risk (prediabetes)
    8       P009          116.0    High risk (prediabetes)
    9       P010           92.0          Low risk (normal)


---

## Filter high-risk patients


```python
print("\nIdentifying patients needing follow-up...")

# Find patients with elevated glucose
high_risk = patients[
    patients["diabetes_risk"].str.contains("High risk")
].copy()

print(f"Found {len(high_risk)} patients with elevated glucose")
if len(high_risk) > 0:
    print(f"Glucose range: {high_risk['glucose_mg_dl'].min()} to {high_risk['glucose_mg_dl'].max()}")
```

    
    Identifying patients needing follow-up...
    Found 32 patients with elevated glucose
    Glucose range: 100.0 to 125.0


---

## Calculate intervention priority scores


```python
print("\nCalculating intervention priority scores...")

priority_patients = []
records = high_risk.to_dict("records")

# Calculate priority score for each high-risk patient
for i in range(len(records)): #changed to len(records), because was iterating over first patient initially, skipping
    patient = records[i]

    # Priority score: higher glucose + older age = higher priority
    glucose = float(patient["glucose_mg_dl"])
    age = int(patient["age"])
    priority_score = (glucose - 100) + (age * 0.5)

    priority_patients.append({
        "patient_id": patient["patient_id"],
        "glucose": glucose,
        "age": age,
        "priority_score": round(priority_score, 1)
    })

# Sort by priority score
priority_patients.sort(key=lambda x: x["priority_score"], reverse=True)

print(f"Priority scores calculated for {len(priority_patients)} patients")
if priority_patients:
    print(f"\nTop 3 priority patients:")
    for p in priority_patients[:3]:
        print(f"  {p['patient_id']}: score {p['priority_score']} (glucose={p['glucose']}, age={p['age']})")
```

    
    Calculating intervention priority scores...
    Priority scores calculated for 32 patients
    
    Top 3 priority patients:
      P024: score 61.0 (glucose=125.0, age=72)
      P028: score 55.0 (glucose=117.0, age=76)
      P020: score 51.0 (glucose=113.0, age=76)


---

## Summary


```python
print("\n" + "=" * 50)
print("Lab Results Analysis Complete")
print("=" * 50)
print(f"Total patients analyzed: {len(patients)}")
print(f"High-risk patients: {len(high_risk)}")
print(f"Patients prioritized for intervention: {len(priority_patients)}")
```

    
    ==================================================
    Lab Results Analysis Complete
    ==================================================
    Total patients analyzed: 50
    High-risk patients: 32
    Patients prioritized for intervention: 32


---

## Debugging Checklist

After fixing all bugs, verify:

- [ ] Runs without errors end-to-end
- [ ] Glucose values are reasonable numbers
- [ ] Risk categories make sense (high glucose = high risk)
- [ ] All high-risk patients are identified
- [ ] Priority scores calculated correctly
- [ ] Restart kernel + Run All completes successfully
- [ ] Added comments explaining each fix
