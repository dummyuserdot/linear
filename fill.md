## 📘 Missing Data Imputation
## 📑 Table of Contents

1.  Missing Values ko Detect Karna
2.  Numeric Data Fill Karna (Mean, Median, Mode)
3.  String / Categorical Data Fill Karna
4.  Other Data Types (Datetime, Boolean)
5.  Advanced Methods (Interpolation, KNN)
6.  Group-wise Imputation (Real World Scenario)
7.  Real World Project Example (Bank Data)
8.  Best Practices & Common Mistakes
9.  Practice Exercises

---

## 1. Missing Values ko Detect Karna

Kuch bhi fill karne se pehle, yeh dekhna zaroori hai ke **kitne values missing hain aur kahan hain**. Pandas mein `isna()` aur `notna()` use hotay hain.

```python
import pandas as pd
import numpy as np

# Sample Excel/CSV load karna
# df = pd.read_excel('mera_data.xlsx')   # Excel ke liye
# df = pd.read_csv('mera_data.csv')       # CSV ke liye

# Demo data
df = pd.DataFrame({
    'Name': ['Ali', 'Sara', None, 'Bilal', 'Hina'],
    'Age': [25, np.nan, 30, np.nan, 28],
    'City': ['Lahore', 'Karachi', 'Lahore', None, 'Islamabad'],
    'Salary': [50000, 60000, np.nan, 45000, np.nan]
})

# Har column mein missing values count karna
print(df.isna().sum())
```

**Output:**
```
Name      1
Age       2
City      1
Salary    2
dtype: int64
```

Yeh batata hai ke `Name` mein 1, `Age` mein 2, `City` mein 1, aur `Salary` mein 2 values missing hain.

---

## 2. Numeric Data Fill Karna (Mean, Median, Mode)

Numbers ke liye sab se common approach yeh hai:

- **Mean (Average):** Jab data normal distribution mein ho aur outliers na hon.
- **Median (Middle Value):** Jab outliers hon (mean se behtar hai).
- **Mode (Most Frequent):** Jab discrete numbers hon ya specific pattern ho.

Pandas mein `fillna()` method use hota hai.

```python
# MEAN se fill karna
df['Age'] = df['Age'].fillna(df['Age'].mean())

# MEDIAN se fill karna
# df['Age'] = df['Age'].fillna(df['Age'].median())

# MODE se fill karna
# df['Age'] = df['Age'].fillna(df['Age'].mode()[0])
```

**Real-world tip:** Agar `Salary` column mein kuch bohat zyada high values (outliers) hain, to **median** use karein, mean nahi. Mean outliers se affect ho jata hai.

**Important:** `inplace=True` use karne se FutureWarning aata hai, isliye `df['col'] = df['col'].fillna(...)` behtar hai.

---

## 3. String / Categorical Data Fill Karna

Text columns (jaise `City`, `Gender`, `Department`) ke liye **mode** (most frequent value) best hota hai.

```python
# Categorical column ka mode nikalna
city_mode = df['City'].mode()[0]
print("City ka mode:", city_mode)  # Lahore (kyunke Lahore 2 baar aaya)

# Mode se fill karna
df['City'] = df['City'].fillna(city_mode)
```

Agar mode clear na ho, to **"Unknown"** ya **"Not Specified"** constant value bhi daal sakte hain.

```python
# Constant value se fill karna
df['City'] = df['City'].fillna('Unknown')
```

---

## 4. Other Data Types (Datetime, Boolean)

### Datetime Columns

Agar date/time column hai, to **forward fill (`ffill`)** ya **backward fill (`bfill`)** use karein. Yeh last known value ko aage ya peeche propagate karta hai.

```python
# Date column banate hain
df['Joining_Date'] = pd.to_datetime(['2023-01-01', None, '2023-03-15', None, '2023-05-20'])

# Forward fill: pichli known date aage copy hoti hai
df['Joining_Date'] = df['Joining_Date'].ffill()

# Backward fill: agli known date peeche copy hoti hai
# df['Joining_Date'] = df['Joining_Date'].bfill()
```

### Boolean Columns

Boolean (True/False) ke liye **mode** ya **False** default use kar sakte hain.

```python
df['Is_Active'] = [True, None, False, True, None]
df['Is_Active'] = df['Is_Active'].fillna(df['Is_Active'].mode()[0])
```

---

## 5. Advanced Methods (Interpolation & KNN)

### Interpolation (Linearly Beech ki Values Bharo)

Jab data **time series** ho (jaise monthly sales, temperature), to `interpolate()` behtar kaam karta hai. Yeh do known points ke beech ki value calculate karta hai.

```python
# Linear interpolation
df['Salary'] = df['Salary'].interpolate(method='linear')
```

### KNN Imputer (Machine Learning se Fill Karna)

Yeh advanced method hai. Yeh **similar rows** dhoondta hai aur unki values se missing value predict karta hai. `sklearn` library chahiye.

```python
from sklearn.impute import KNNImputer

# Sirf numeric columns pe KNN lagta hai
numeric_cols = ['Age', 'Salary']
imputer = KNNImputer(n_neighbors=5)  # 5 similar rows dekhega
df[numeric_cols] = imputer.fit_transform(df[numeric_cols])
```

---

## 6. Group-wise Imputation (Real World Scenario)

Asal duniya mein aap chahte hain ke **har group ke andar** mean/median se fill ho. Misal: Har **Customer_Group** ka apna average ho, poore data ka nahi. Iske liye `groupby().transform()` use karein.

```python
data = {
    'Customer_Group': ['A', 'A', 'A', 'B', 'B', 'B'],
    'Value': [10, np.nan, 20, 50, np.nan, 60]
}
df2 = pd.DataFrame(data)

# Har group ka mean nikalna
group_means = df2.groupby('Customer_Group')['Value'].transform('mean')
print(group_means)
# A group ka mean = 15, B group ka mean = 55

# Us group mean se fill karna
df2['Value_Filled'] = df2['Value'].fillna(group_means)
```

**Output:**
```
  Customer_Group  Value  Value_Filled
0              A   10.0          10.0
1              A    NaN          15.0   # A group ke mean se
2              A   20.0          20.0
3              B   50.0          50.0
4              B    NaN          55.0   # B group ke mean se
5              B   60.0          60.0
```

---

## 7. Real World Project Example (Bank Data)

Ek real-world bank dataset ka example dekhte hain jahan **deposit values** missing thin aur **negative values** ko bhi handle karna tha.

```python
# Real bank data scenario
bank_df = pd.DataFrame({
    'Customer_ID': [101, 102, 103, 104, 105],
    'Deposit_Value': [5000, -200, np.nan, 7000, np.nan],
    'Credit_Score': [700, np.nan, 650, np.nan, 720]
})

# Step 1: Negative values ko NaN banao
bank_df.loc[bank_df['Deposit_Value'] < 0, 'Deposit_Value'] = np.nan

# Step 2: Median se fill karo (outliers ke liye safe)
bank_df['Deposit_Value'] = bank_df['Deposit_Value'].fillna(
    bank_df['Deposit_Value'].median()
)

# Step 3: Credit score mean se fill
bank_df['Credit_Score'] = bank_df['Credit_Score'].fillna(
    bank_df['Credit_Score'].mean()
)

print(bank_df)
```

Yeh approach real projects mein use hoti hai: pehle **invalid values ko NaN** banaya, phir **median/mean** se fill kiya.

---

## 8. Best Practices & Common Mistakes

### ✅ Kya Karein

1. **Har column ka alag strategy** use karein: numeric = mean/median, categorical = mode.
2. **Outliers** hon to **median** use karein, mean nahi.
3. **Time series** data mein `ffill` / `interpolate` use karein.
4. Fill karne se pehle **missing percentage** check karein. Agar 40-50% missing hai, to us column ko drop karna behtar ho sakta hai.
5. **Group-wise** fill karein agar data groups mein divide ho sakta hai.

### ❌ Kya Na Karein

1. **Sab columns ko 0 se fill na karein** — is se galat signal jata hai (jaise salary 0 means free).
2. **Deprecated methods** (`inplace=True` with chained assignment) use na karein.
3. **Mean ko blindly** har jagah use na karein — outliers ko handle karna zaroori hai.

---

## 9. Practice Exercises

### Exercise 1: Basic Fill
Neeche diya gaya DataFrame banayein aur `Age` ko mean se, `City` ko mode se fill karein.

```python
df = pd.DataFrame({
    'Name': ['A', 'B', 'C', 'D'],
    'Age': [20, np.nan, 25, np.nan],
    'City': ['Lahore', None, 'Lahore', 'Karachi']
})
```

### Exercise 2: Group-wise Fill
Har `Department` ka apna average salary nikal kar missing salary fill karein.

```python
df = pd.DataFrame({
    'Department': ['HR', 'HR', 'IT', 'IT', 'IT'],
    'Salary': [30000, np.nan, 50000, np.nan, 60000]
})
```

### Exercise 3: Time Series
Date column ko `ffill` se fill karein.

```python
df = pd.DataFrame({
    'Date': pd.to_datetime(['2023-01-01', None, None, '2023-04-01']),
    'Sales': [100, np.nan, 300, 400]
})
df['Date'] = df['Date'].ffill()
```

---

## 🎯 Summary

| Data Type | Best Method | Code |
|---|---|---|
| Numeric (normal) | Mean | `df['col'].fillna(df['col'].mean())` |
| Numeric (outliers) | Median | `df['col'].fillna(df['col'].median())` |
| Categorical/String | Mode | `df['col'].fillna(df['col'].mode()[0])` |
| Time Series | Forward Fill | `df['col'].ffill()` |
| Group Data | Group Mean | `df['col'].fillna(df.groupby('grp')['col'].transform('mean'))` |
| Advanced | KNN | `KNNImputer(n_neighbors=5)` |
