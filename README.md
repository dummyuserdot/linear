# 📘 Linear Regression

# Chapter 1: Real Client Data – Excel/CSV Understanding

## 🎯 Scenario

Socho tumhe ek client ne **house_prices.xlsx** file di hai. Usme 20 columns hain. Tumhe ye samajhna hai ke kaunsa data useful hai aur kaunsa nahi.

## 1.1 Sab Se Pehle Kya Karna Hai?

Jab client data deta hai, to **sab se pehle** ye 5 steps karo:

```
STEP 1: Data Load karo
STEP 2: Data ko dekho (head, shape, info)
STEP 3: Data ki quality check karo (nulls, duplicates)
STEP 4: Columns ka meaning samjho (client se poocho)
STEP 5: Decide karo kaunsa column feature aur kaunsa target
```

## 1.2 Code – Har Line Explained

```python
import pandas as pd
```

**Explanation:**
- `import pandas as pd` → Pandas library ko load karta hai jo Excel/CSV files read karne ke liye use hoti hai.
- `as pd` → Pandas ko short name `pd` de rahe hain taake baar baar `pandas` na likhna pade.

```python
database = pd.read_excel("datasets/house_prices.xlsx")
```

**Explanation:**
- `pd.read_excel()` → Excel file read karne ka function.
- `"datasets/house_prices.xlsx"` → File ka path. Agar file usi folder mein hai to sirf `"house_prices.xlsx"` likhna kaafi hai.
- `database` → Variable jismein poora data store ho gaya. Ye ek **DataFrame** hai (Excel sheet ki tarah).

```python
print(database.head())
```

**Explanation:**
- `database.head()` → Pehli **5 rows** dikhata hai.
- Ye check karne ke liye ke data kaisa hai, columns kya hain, values kaisi hain.

```python
print(database.info())
```

**Explanation:**
- `database.info()` → Har column ka:
  - **Data type** (int, float, object/text)
  - **Non-null count** (kitni values missing nahi hain)
  - **Total rows** (dataset kitna bada hai)

**Example Output:**
```
RangeIndex: 1000 entries, 0 to 999
Data columns (total 7 columns):
 #   Column            Non-Null Count  Dtype  
---  ------            --------------  -----  
 0   area_marla        980 non-null    float64
 1   bedrooms          995 non-null    float64
 2   house_age_years   1000 non-null   int64  
 3   bathrooms         990 non-null    float64
 4   location          1000 non-null   object 
 5   parking_spaces    985 non-null    float64
 6   price_lakhs       1000 non-null   float64
```

**Is output se kya samjhe?**
- `area_marla` mein **20 values missing** hain (1000 - 980 = 20)
- `location` **object type** hai → matlab **categorical** (text) column hai
- `price_lakhs` mein koi missing value nahi

```python
print(database.describe())
```

**Explanation:**
- `describe()` → Sirf **numerical columns** ka statistical summary deta hai:
  - `mean` = average
  - `std` = standard deviation (data kitna spread hai)
  - `min`, `25%`, `50%` (median), `75%`, `max`

**Ye kyun zaroori hai?**
- Agar `price_lakhs` ka `min` = -50 hai, to **negative price** possible nahi → data error hai.
- Agar `area_marla` ka `max` = 5000 hai jabke `75%` = 10 hai → **outlier** ho sakta hai.

```python
print(database.isnull().sum())
```

**Explanation:**
- `isnull()` → Har cell check karta hai: missing hai ya nahi (True/False)
- `.sum()` → Har column mein kitni `True` (missing) values hain, wo count karta hai.

**Example Output:**
```
area_marla          20
bedrooms             5
house_age_years      0
bathrooms           10
location             0
parking_spaces      15
price_lakhs          0
dtype: int64
```

```python
print(database.duplicated().sum())
```

**Explanation:**
- `duplicated()` → Check karta hai ke koi row **repeat** hai ya nahi.
- `.sum()` → Total duplicate rows kitni hain.

Agar output `15` aaye → 15 duplicate rows hain jinhe remove karna hoga.

## 1.3 Kya Complete Excel Data Model Mein Use Hota Hai?

**❌ Nahi!** Har column training mein **nahi** jata.

| Column Type | Example | Model Mein Jayega? | Kyun? |
|-------------|---------|-------------------|-------|
| **ID/Serial** | `customer_id`, `order_no` | ❌ Nahi | Prediction mein koi kaam nahi |
| **Names** | `customer_name`, `city_name` | ❌ Nahi (mostly) | Text hai, pattern nahi |
| **Dates** | `purchase_date` | ⚠️ Kabhi kabhi | Feature engineering se (year, month) |
| **Irrelevant** | `remarks`, `notes` | ❌ Nahi | Free text, model nahi samajhta |
| **Numerical** | `area`, `bedrooms` | ✅ Haan | Directly usable |
| **Categorical** | `location`, `type` | ✅ Haan (encode karke) | Pattern banta hai |

## 1.4 Features (X) aur Target (y) Kya Hote Hain?

```
┌─────────────────────────────────────────────────────┐
│                    DATASET                          │
├─────────────────────────────────────────────────────┤
│  area_marla │ bedrooms │ location │ price_lakhs     │
│─────────────┼──────────┼──────────┼─────────────────│
│     10      │    3     │  DHA     │     150         │
│      5      │    2     │  Gulberg │      80         │
│      7      │    3     │  DHA     │     120         │
└─────────────┴──────────┴──────────┴─────────────────┘
       ↑            ↑          ↑            ↑
       └────────────┴──────────┘            │
              FEATURES (X)                  │
         (Input / Predictors)               │
                                    TARGET (y)
                                  (Output / Predict)
```

- **Features (X)** = Jo cheezein hum **jaante hain** (input)
- **Target (y)** = Jo cheez hum **predict karna chahte hain** (output)

## 1.5 Kaunsa Column Feature, Kaunsa Target? – Decision Rule

**Target column identify karne ke rules:**

1. **Client kya pooch raha hai?** → Jo predict karna hai, wo target hai.
2. **Jo column future mein pata nahi hota** → Wo target hai (e.g., price).
3. **Jo column target ke baad ki information hai** → Wo X mein **nahi** dalna (data leakage).

> ⚠️ **Example of Data Leakage:** Agar `price_lakhs` ke saath `tax_amount` column bhi hai, to `tax_amount` X mein **nahi** dalna kyunke tax price ke baad calculate hota hai. Ye cheating hai.

## 1.6 Agar 20 Columns Hon – Kya Sab X Mein Dalna Zaroori Hai?

**❌ Bilkul Nahi!**

> **Rule:** Sirf woh columns X mein dalo jo **genuinely price ko affect karte hain** aur **prediction ke waqt available hote hain**.

**20 columns mein se:**
- 5 irrelevant (ID, name, remarks) → **Remove**
- 3 data leakage → **Remove**
- 2 duplicate information → **Remove** (e.g., `area_sqft` aur `area_marla` same cheez hain)
- 10 useful → **X mein dalo**

## 1.7 Irrelevant Columns Kaise Identify Karein?

| Test | Kaise Check Karein | Action |
|------|-------------------|--------|
| **ID-like?** | Unique values = total rows | Remove |
| **Constant?** | Sirf 1 unique value | Remove |
| **Too many unique?** | >90% values different (names) | Remove |
| **Target se correlation?** | `df.corr()` | Check |

```python
# Numerical columns ki target ke saath correlation check
print(database.corr(numeric_only=True)["price_lakhs"].sort_values(ascending=False))
```

**Explanation:**
- `.corr()` → Correlation matrix banata hai.
- `["price_lakhs"]` → Sirf target ke saath correlation nikalo.
- `.sort_values(ascending=False)` → Sab se zyada correlation wale upar.

**Output:**
```
price_lakhs       1.00
area_marla        0.85  ← Strong positive
bathrooms         0.62  ← Moderate
bedrooms          0.55  ← Moderate
house_age_years  -0.45  ← Negative (purana ghar = kam price)
parking_spaces    0.12  ← Weak
```

- Correlation **0.7+** = strong → zaroor rakhna
- Correlation **0.3-0.7** = moderate → rakh sakte ho
- Correlation **0.0-0.3** = weak → soch samajh kar
- Correlation **negative** = ulta relation (age badhe to price kam)

## 1.8 Numerical vs Categorical Columns

| Feature | Numerical | Categorical |
|---------|-----------|-------------|
| **Kya hai?** | Numbers (measurable) | Categories/labels |
| **Example** | area_marla, bedrooms, price | location, house_type, city |
| **Model mein kaise?** | Directly use | **Encode** karna padta hai |
| **Data type** | int, float | object, category |
| **Operations** | Mean, sum, average | Count, mode |

```python
# Numerical columns
numerical_cols = database.select_dtypes(include=["int64", "float64"]).columns
print("Numerical:", list(numerical_cols))

# Categorical columns
categorical_cols = database.select_dtypes(include=["object"]).columns
print("Categorical:", list(categorical_cols))
```

**Explanation:**
- `select_dtypes(include=["int64", "float64"])` → Sirf numerical columns select karta hai.
- `select_dtypes(include=["object"])` → Sirf text/categorical columns.

### 📝 Exercise 1

Ek CSV file socho jisme ye columns hain:
`id, customer_name, age, city, salary, years_experience, department, salary_next_year`

**Questions:**
1. Kaunsa target hai?
2. Kaunse columns remove karoge?
3. Kaunse categorical hain?
4. Kya koi data leakage hai?

<details>
<summary>Answer Dekho</summary>

1. Target = `salary_next_year` (future prediction)
2. Remove: `id` (ID), `customer_name` (name)
3. Categorical: `city`, `department`
4. Data leakage: `salary` agar `salary_next_year` predict kar rahe ho to `salary` X mein rakh sakte ho (ye current info hai), lekin koi bhi column jo next year ki info de (e.g., `next_year_bonus`) wo leakage hoga.
</details>

---

# Chapter 2: Features (X) aur Target (y)

## 2.1 Code Explanation

```python
x = database[["area_marla", "bedrooms", "house_age_years"]]
```

**Line by line:**
- `database[...]` → DataFrame se columns select kar rahe hain.
- `[["area_marla", "bedrooms", "house_age_years"]]` → **Double square brackets** `[[...]]` ka matlab: **multiple columns select karo** aur result **DataFrame** ho.

```python
y = database["price_lakhs"]
```

**Line by line:**
- `database["price_lakhs"]` → **Single square bracket** `[...]` ka matlab: **single column select karo** aur result **Series** ho (1D array).

## 2.2 Single vs Double Square Brackets – Critical Difference

| Syntax | Result Type | Shape | Kab Use Karein |
|--------|-------------|-------|----------------|
| `database["price_lakhs"]` | **Series** (1D) | `(n,)` | Target `y` ke liye |
| `database[["price_lakhs"]]` | **DataFrame** (2D) | `(n, 1)` | Multiple columns ya X ke liye |
| `database[["area", "beds"]]` | **DataFrame** (2D) | `(n, 2)` | Features `X` ke liye |

**Rule of Thumb:**
- **X** (features) → hamesha **2D DataFrame** hona chahiye → double brackets `[[...]]`
- **y** (target) → **1D Series** hona chahiye → single bracket `[...]`

> ⚠️ **Common Mistake:** `X = database["area_marla"]` → Ye **Series** banega, sklearn **error** dega: `Expected 2D array, got 1D array instead`.

## 2.3 Model Ko Kaise Pata Chalta Hai Ke Price Predict Karna Hai?

**Model ko "pata" nahi chalta** – hum **bataate hain**!

```python
llr.fit(x_train, y_train)
```

- `fit(X, y)` → Model ko X (features) aur y (target) dono dete hain.
- Model **X se y ka relationship** seekhta hai.
- Jo hum `y` mein dete hain, wahi model predict karta hai.

**Analogy:**
> Teacher (model) ko exam (X) aur answer key (y) di jaati hai. Teacher pattern seekhta hai ke "sawal aisa ho to jawab aisa hota hai."

## 2.4 Agar 15 Features Hon – Best Features Kaise Select Karein?

### Method 1: Correlation Analysis

```python
corr_matrix = x_train.corrwith(y_train).abs().sort_values(ascending=False)
print(corr_matrix)
```

**Explanation:**
- `.corrwith(y_train)` → Har feature ka target ke saath correlation.
- `.abs()` → Negative correlation ko positive bana do (strength matter karti hai).
- `.sort_values(ascending=False)` → Strongest pehle.

**Rule:** Top features rakho, weak (<0.1) hata do.

### Method 2: SelectKBest (Statistical)

```python
from sklearn.feature_selection import SelectKBest, f_regression

selector = SelectKBest(score_func=f_regression, k=5)
X_selected = selector.fit_transform(x_train, y_train)

# Kaunse columns select hue?
selected_cols = x_train.columns[selector.get_support()]
print("Selected:", list(selected_cols))
```

**Explanation:**
- `SelectKBest` → Best `k` features select karta hai.
- `f_regression` → Statistical test jo batata hai feature target se kitna related hai.
- `fit_transform` → Fit bhi karo aur transform bhi (selected columns return).
- `get_support()` → Boolean mask (True = selected).

### Method 3: Domain Knowledge

> **Sab se important!** Client se poocho: "Aap ke experience mein price par kaunse factors affect karte hain?"

Agar client kehta hai "location sab se important hai" → `location` zaroor rakhna, chahe correlation low ho.

### Method 4: Recursive Feature Elimination (RFE)

```python
from sklearn.feature_selection import RFE

rfe = RFE(estimator=LinearRegression(), n_features_to_select=5)
rfe.fit(x_train, y_train)

print("Selected:", list(x_train.columns[rfe.support_]))
print("Ranking:", dict(zip(x_train.columns, rfe.ranking_)))
```

- `RFE` → Recursively weakest feature remove karta hai, best `n` rakhta hai.
- `ranking_` → 1 = selected, 2+ = removed (lower = better).

### 📝 Exercise 2

```python
X = df[["area", "beds", "baths", "location", "age"]]
y = df["price"]
```

**Questions:**
1. `X` ka shape kya hoga agar 1000 rows hain?
2. `y` ka shape kya hoga?
3. Kya `location` directly X mein dal sakte hain? Kyun?

<details>
<summary>Answer</summary>

1. `X.shape = (1000, 5)` – 1000 rows, 5 columns (2D DataFrame)
2. `y.shape = (1000,)` – 1000 values, 1D Series
3. **Nahi**, `location` categorical (text) hai. Pehle **encode** karna hoga (LabelEncoder ya OneHotEncoder).
</details>

---

# Chapter 3: Train/Test Split – Complete Explanation

## 3.1 Code

```python
from sklearn.model_selection import train_test_split

x_train, x_test, y_train, y_test = train_test_split(
    x, y, test_size=0.2, random_state=42
)
```

## 3.2 Training Data Kya Hota Hai?

**Training Data (80%)** → Model **seekhne** ke liye use karta hai. Ye data model ko dikhaya jata hai.

**Analogy:**
> Jaise student **textbook** se padhta hai. Textbook = training data.

## 3.3 Testing Data Kya Hota Hai?

**Testing Data (20%)** → Model ki **performance check** karne ke liye. Ye data model ne **kabhi nahi dekha**.

**Analogy:**
> Jaise student **exam** deta hai jo usne pehle nahi dekha. Exam = testing data.

## 3.4 80/20 Split Kaise Hoti Hai?

```
Original Data: 1000 rows
        │
        ├─── test_size=0.2 (20%) ────→ 200 rows → Testing
        │
        └─── Baqi (80%) ─────────────→ 800 rows → Training
```

**Formula:**
- Testing = `1000 × 0.2 = 200 rows`
- Training = `1000 - 200 = 800 rows`

## 3.5 X_train, X_test, y_train, y_test – Difference

| Variable | Kya Hai? | Kitni Rows? | Model Mein Role |
|----------|----------|-------------|-----------------|
| `X_train` | Training features | 800 | Model **seekhta** hai |
| `y_train` | Training target | 800 | Model **seekhta** hai |
| `X_test` | Testing features | 200 | Model **predict** karta hai |
| `y_test` | Testing target | 200 | Prediction **compare** karte hain |

**Important:** `X_train` aur `y_train` **same rows** ke hain. `X_test` aur `y_test` **same rows** ke hain (jo training mein nahi hain).

## 3.6 random_state=42 Ka Actual Meaning

**`random_state`** → Shuffle karne ka **seed** (random number generator ka starting point).

**Kyun zaroori hai?**
- `train_test_split` **randomly** rows choose karta hai.
- Bina seed ke, har baar **different split** milega → results reproducible nahi.
- Seed dene se **same split** har baar milega.

**Analogy:**
> Jaise lottery machine. Seed = machine ka starting position. Same seed = same result.

### 42, 10, 100, 0 – Kya Hoga?

| random_state | Result |
|--------------|--------|
| `42` | Ek specific split (most common) |
| `10` | Different split |
| `100` | Different split |
| `0` | Different split |
| `None` (default) | Har run par different split |

> **Kya random_state accuracy increase karta hai?**
>
> **❌ Nahi!** Random_state sirf **reproducibility** ke liye hai. Accuracy par directly effect nahi. Lekin different splits se **different scores** aa sakte hain – isliye same seed use karna best practice hai.

## 3.7 1000 Rows Ka Numerical Example

```python
# Assume: 1000 rows total, test_size=0.2
# Training: 800 rows, Testing: 200 rows

print("X_train shape:", x_train.shape)   # (800, 3)
print("X_test shape:", x_test.shape)     # (200, 3)
print("y_train shape:", y_train.shape)   # (800,)
print("y_test shape:", y_test.shape)     # (200,)
```

## 3.8 Visual Table

```
┌──────────────────────────────────────────────────────┐
│              ORIGINAL DATASET (1000 rows)            │
├──────────────────────────────────────────────────────┤
│  Row 1  │  Row 2  │  ...  │  Row 800  │  Row 801...1000 │
└──────────────────────────────────────────────────────┘
           │                              │
           ▼                              ▼
┌─────────────────────┐      ┌─────────────────────┐
│   TRAINING SET      │      │    TESTING SET      │
│   800 rows (80%)    │      │   200 rows (20%)    │
│                     │      │                     │
│  X_train │ y_train  │      │  X_test  │ y_test   │
│  (800,3) │ (800,)   │      │  (200,3) │ (200,)   │
└─────────────────────┘      └─────────────────────┘
        │                              │
        ▼                              ▼
   Model.fit()                  Model.predict()
   (Seekhna)                    (Exam dena)
```

## 3.9 ⚠️ Important: Split Karne Se Pehle Data Clean Karo

**❌ Galat Order:**
```
Split → Clean → Train  (Data leakage risk!)
```

**✅ Sahi Order:**
```
Clean → Split → Train  (Safe!)
```

**Kyun?** Agar pehle split kiya aur phir missing values ko **overall mean** se fill kiya, to testing data ki info training mein chali gayi = **data leakage**.

### 📝 Exercise 3

1000 rows, `test_size=0.3`, `random_state=7`

**Questions:**
1. Training mein kitni rows?
2. Testing mein kitni rows?
3. Agar `random_state=8` karein to scores same aayenge?

<details>
<summary>Answer</summary>

1. Testing = 1000 × 0.3 = **300 rows**
2. Training = 1000 - 300 = **700 rows**
3. **Nahi**, alag split hoga, scores thode different aa sakte hain.
</details>

---

# Chapter 4: Linear Regression Model Training

## 4.1 Code

```python
from sklearn.linear_model import LinearRegression

llr = LinearRegression()
llr.fit(x_train, y_train)
```

## 4.2 Linear Regression Kya Hai?

**Linear Regression** ek algorithm hai jo **features aur target ke beech straight-line relationship** dhoondta hai.

**Equation:**
```
y = b0 + b1*x1 + b2*x2 + b3*x3 + ... + bn*xn
```

Where:
- `y` = predicted value (price)
- `b0` = intercept (y-axis par jab sab x=0 hon)
- `b1, b2, ...` = coefficients (har feature ka weight)
- `x1, x2, ...` = features

**Example:**
```
price = 20 + 5*area + 3*bedrooms - 2*age
```
- Intercept = 20 (base price)
- area ka coefficient = 5 (har 1 marla par 5 lakh increase)
- bedrooms ka coefficient = 3 (har bedroom par 3 lakh increase)
- age ka coefficient = -2 (har saal purana hone par 2 lakh decrease)

## 4.3 Model Training Ka Actual Meaning

**Training** = Model ko data se **best coefficients (b0, b1, ...) dhoondhna**.

Model aise coefficients dhoondta hai jinse **predictions aur actual values ka difference (error) minimum ho**.

**Method:** **Ordinary Least Squares (OLS)** – squared errors ka sum minimize karta hai.

```
Minimize: Σ(actual - predicted)²
```

## 4.4 fit() Function Kya Karta Hai?

```python
llr.fit(x_train, y_train)
```

**Step by step:**
1. `x_train` (features) aur `y_train` (target) receive karta hai.
2. Har feature ka target ke saath relationship calculate karta hai.
3. **Best coefficients** dhoondta hai (OLS method se).
4. Coefficients ko **model ke andar store** kar deta hai.

> **fit() = "Seekhna"** – Model data se patterns seekhta hai.

## 4.5 Model Patterns Kaise Learn Karta Hai?

**Simple 1-feature example:**

Socho data ye hai:
| area | price |
|------|-------|
| 5 | 80 |
| 10 | 150 |
| 15 | 220 |
| 20 | 290 |

Model dekhta hai: "Jab area badhta hai, price badhta hai." Ye **line** fit karta hai:
```
price = 10 + 14*area
```

Check:
- area=5 → 10 + 14(5) = 80 ✅
- area=10 → 10 + 14(10) = 150 ✅

Model ne **pattern seekh liya**: price = 10 + 14 × area.

## 4.6 Coefficients Aur Intercept Kya Hote Hain?

```python
print("Intercept (b0):", llr.intercept_)
print("Coefficients:", llr.coef_)

# Feature names ke saath
for feature, coef in zip(x_train.columns, llr.coef_):
    print(f"{feature}: {coef:.2f}")
```

**Output Example:**
```
Intercept (b0): 15.5
area_marla: 12.3
bedrooms: 8.7
house_age_years: -2.1
```

**Interpretation:**
- `Intercept = 15.5` → Jab area=0, bedrooms=0, age=0 ho to price 15.5 lakh (theoretical base).
- `area_marla = 12.3` → Har 1 marla zyada area par price **12.3 lakh badhta** hai (agar baqi features same rahen).
- `bedrooms = 8.7` → Har ek extra bedroom par price **8.7 lakh badhta** hai.
- `house_age_years = -2.1` → Har saal purana ghar **2.1 lakh sasta** hota hai.

> **⚠️ Important:** Coefficients ko **"agar baqi sab same rahe"** condition mein interpret karo. Real life mein features ek saath change hote hain.

## 4.7 Model Training Ke Baad Kya Save Hota Hai?

Model ke andar ye cheezein save hoti hain:
1. **Coefficients** (`coef_`) – har feature ka weight
2. **Intercept** (`intercept_`) – base value
3. **Number of features** – kitne features par train hua
4. **Feature names** (agar DataFrame diya ho)

```python
# Model save karna (joblib se)
import joblib
joblib.dump(llr, "house_price_model.pkl")

# Load karna
loaded_model = joblib.load("house_price_model.pkl")
```

### 📝 Exercise 4

```python
llr.fit(X_train, y_train)
print(llr.coef_)
# Output: [10.5, 5.2, -1.8]
print(llr.intercept_)
# Output: 25
```

Features: `area`, `bedrooms`, `age`

**Questions:**
1. Agar area=10, bedrooms=3, age=5 ho to predicted price kya hoga?
2. Age ka negative coefficient kya batata hai?

<details>
<summary>Answer</summary>

1. `price = 25 + 10.5(10) + 5.2(3) + (-1.8)(5) = 25 + 105 + 15.6 - 9 = 136.6`
2. Purana ghar hone par price **kam** hota hai (negative relationship).
</details>

---

# Chapter 5: Training Score vs Testing Score (MOST IMPORTANT)

## 5.1 Code

```python
train_score = llr.score(x_train, y_train) * 100
test_score = llr.score(x_test, y_test) * 100

print("Training Score:", train_score)
print("Testing Score:", test_score)
```

## 5.2 ⚠️ CRITICAL: .score() R² Deta Hai, Accuracy Nahi!

> **Sab se bada misunderstanding:** `llr.score()` **R² score** return karta hai, **accuracy percentage nahi**.

**R² Score Kya Hai?**

```
R² = 1 - (SS_res / SS_total)

SS_res = Σ(actual - predicted)²  (model ka error)
SS_total = Σ(actual - mean)²      (baseline error)
```

**R² ka Meaning:**

| R² Value | Meaning |
|----------|---------|
| `1.0` (100%) | Perfect predictions (real life mein impossible) |
| `0.9` (90%) | Model **90% variance** explain kar raha hai |
| `0.5` (50%) | Model **50% variance** explain kar raha hai |
| `0.0` (0%) | Model **baseline se behtar nahi** (mean se bhi bura nahi) |
| `-0.5` (-50%) | Model **mean se bhi bura** predict kar raha hai! |

> **R² = 0% ka matlab:** Model ne kuch nahi seekha. Har prediction mean ke barabar hai.
>
> **R² negative ka matlab:** Model itna bura hai ke simple average (mean) se bhi bura predict kar raha hai. **Ye serious problem hai.**

## 5.3 Case A: Training 90%, Testing 85%

```
Training: 90%  ████████████████████░
Testing:  85%  ███████████████████░░
```

**Iska Matlab:**
- Model ne training data **achhe se** seekha (90%)
- Unseen data par bhi **achha perform** kar raha hai (85%)
- Difference sirf **5%** – normal hai

**Kya Model Good Hai?**
✅ **Haan!** Ye **ideal situation** hai:
- Training high ✅
- Testing high ✅
- Gap small ✅

**Action:** Model deploy karo. ✅

## 5.4 Case B: Training 40%, Testing 35%

```
Training: 40%  ████████░░░░░░░░░░░░░
Testing:  35%  ███████░░░░░░░░░░░░░░
```

**Problem:** **Underfitting** (model ne kuch seekha hi nahi)

**Reasons:**
1. Features target se related nahi hain (galat features)
2. Relationship non-linear hai (linear model fit nahi ho raha)
3. Important features missing hain
4. Data mein bahut noise hai

**Solutions:**

```python
# 1. Better features add karo
X = df[["area", "bedrooms", "bathrooms", "location", "parking", "age"]]

# 2. Non-linear features add karo (feature engineering)
X["area_squared"] = X["area"] ** 2
X["area_x_bedrooms"] = X["area"] * X["bedrooms"]

# 3. Polynomial features
from sklearn.preprocessing import PolynomialFeatures
poly = PolynomialFeatures(degree=2, include_bias=False)
X_poly = poly.fit_transform(X)

# 4. Different model try karo
from sklearn.ensemble import RandomForestRegressor
rf = RandomForestRegressor(random_state=42)
rf.fit(X_train, y_train)
```

**Before/After Comparison:**
```
Before: Train=40%, Test=35%  (Underfitting)
After (polynomial): Train=82%, Test=79%  ✅
```

## 5.5 Case C: Training 95%, Testing 45%

```
Training: 95%  █████████████████████░
Testing:  45%  █████████░░░░░░░░░░░░░
Gap:      50%  ← HUGE!
```

**Problem:** **Overfitting** (model training data **ratta** lagata hai)

**Kya Model Overfitting Kar Raha Hai?**
✅ **Bilkul!** 95% vs 45% ka **50% gap** clear overfitting hai.

**Reasons:**
1. Model **bahut complex** hai (bahut zyada features)
2. Training data **bahut kam** hai
3. **Noise** ko model ne pattern samajh liya
4. **Data leakage** (X mein target ki info hai)
5. Outliers ne model ko distort kiya

**Solutions (Step by Step):**

```python
# Solution 1: Features kam karo
from sklearn.feature_selection import SelectKBest, f_regression
selector = SelectKBest(f_regression, k=5)
X_train_sel = selector.fit_transform(X_train, y_train)
X_test_sel = selector.transform(X_test)  # ⚠️ Sirf transform, fit nahi!

# Solution 2: Regularization (Ridge/Lasso)
from sklearn.linear_model import Ridge, Lasso

ridge = Ridge(alpha=1.0)
ridge.fit(X_train, y_train)
print("Ridge Train:", ridge.score(X_train, y_train))
print("Ridge Test:", ridge.score(X_test, y_test))

# Solution 3: Cross-Validation
from sklearn.model_selection import cross_val_score
scores = cross_val_score(LinearRegression(), X, y, cv=5)
print("CV Scores:", scores)
print("Mean:", scores.mean())

# Solution 4: More data collect karo (agar possible ho)

# Solution 5: Outliers remove karo
Q1 = X_train.quantile(0.25)
Q3 = X_train.quantile(0.75)
IQR = Q3 - Q1
mask = ~((X_train < (Q1 - 1.5*IQR)) | (X_train > (Q3 + 1.5*IQR))).any(axis=1)
X_train_clean = X_train[mask]
y_train_clean = y_train[mask]
```

**Before/After:**
```
Before:  Train=95%, Test=45%   (Overfitting)
After:   Train=83%, Test=80%   ✅ (Balanced!)
```

## 5.6 Case D: Training 40%, Testing 80%

```
Training: 40%  ████████░░░░░░░░░░░░░
Testing:  80%  ████████████████████░
```

**Kya Ye Possible Hai?**
**✅ Haan, possible hai!** (Lekin rare)

**Testing Score Training Se Zyada Kyun?**

| Reason | Explanation |
|--------|-------------|
| **Lucky split** | Testing set mein easy examples aa gaye |
| **Training mein outliers** | Training data mein mushkil/outlier rows thin |
| **Small dataset** | Chhote data mein random variation zyada |
| **Data distribution difference** | Testing set training se "easier" hai |

**Kya Ye Normal Hai?**
- **Thoda gap (5-10%)** normal hai
- **Bada gap (40% vs 80%)** suspicious hai

**Kya Trust Karein?**
❌ **Nahi!** Ye **reliable** nahi hai. Reasons:
1. Ye **luck** ho sakta hai
2. Real-world mein aisa nahi hoga
3. Testing score **genuine performance** nahi dikha raha

**Solution:**
```python
# Cross-validation use karo – zyada reliable
from sklearn.model_selection import cross_val_score

scores = cross_val_score(llr, X, y, cv=5, scoring="r2")
print("CV Scores:", scores)        # [0.45, 0.38, 0.42, 0.40, 0.44]
print("Mean R²:", scores.mean())   # 0.418 → Real performance
print("Std:", scores.std())        # Variation
```

**Better approach:** Multiple random states try karo:

```python
for rs in [0, 10, 42, 100, 2024]:
    X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, random_state=rs)
    m = LinearRegression().fit(X_tr, y_tr)
    print(f"RS={rs}: Train={m.score(X_tr, y_tr):.2f}, Test={m.score(X_te, y_te):.2f}")
```

## 5.7 Case E: Training 75%, Testing 75%

```
Training: 75%  ███████████████░░░░░░
Testing:  75%  ███████████████░░░░░░
```

**Kya Model Balanced Hai?**
✅ **Haan!** Ye **balanced** hai:
- Same score ✅
- No overfitting ✅
- No underfitting ✅
- Model **generalize** kar raha hai

**Lekin ek sawal:** 75% **kaafi** hai ya nahi? Ye **business requirement** par depend karta hai:
- House price prediction: 75% **acceptable** ho sakta hai
- Medical diagnosis: 75% **nahi kaafi**

**Action:** Agar domain mein 75% acceptable hai to deploy karo. Warna features improve karo.

## 5.8 Complete Case Summary Table

| Case | Train | Test | Gap | Problem | Solution |
|------|-------|------|-----|---------|----------|
| A | 90% | 85% | 5% | ✅ None | Deploy! |
| B | 40% | 35% | 5% | Underfitting | Better features/model |
| C | 95% | 45% | 50% | Overfitting | Regularization, less features |
| D | 40% | 80% | -40% | Unreliable | Cross-validation |
| E | 75% | 75% | 0% | ✅ Balanced | Deploy (if acceptable) |

## 5.9 Negative R² aur 0% Score Ka Meaning

```python
# Negative R² example
y_true = [100, 200, 300, 400]
y_pred = [400, 300, 200, 100]  # Ulta predict kar diya!

r2 = r2_score(y_true, y_pred)
print(r2)  # -3.0 (negative!)
```

**Negative R² ka matlab:**
- Model **mean se bhi bura** predict kar raha hai
- **Serious problem** hai
- Reasons: wrong features, data error, model bug

**0% R² ka matlab:**
- Model ne **kuch nahi seekha**
- Har prediction = mean ke barabar
- Model useless hai

### 📝 Exercise 5

| Train | Test | Diagnose |
|-------|------|----------|
| 88% | 82% | ? |
| 30% | 28% | ? |
| 99% | 40% | ? |
| 60% | 85% | ? |

<details>
<summary>Answer</summary>

1. **Good fit** – deploy karo
2. **Underfitting** – features/model improve karo
3. **Overfitting** – regularization/less features
4. **Suspicious** – cross-validation karo, luck ho sakta hai
</details>

---

# Chapter 6: Model Performance Improve Karna

## 6.1 Missing Values Handle Karna

```python
# Check
print(df.isnull().sum())

# Strategy 1: Drop rows (agar bahut kam missing hon)
df_clean = df.dropna()

# Strategy 2: Mean/Median imputation (numerical)
from sklearn.impute import SimpleImputer

imputer = SimpleImputer(strategy="median")  # median outliers ke liye better
X_train_imp = imputer.fit_transform(X_train)
X_test_imp = imputer.transform(X_test)  # ⚠️ Sirf transform!

# Strategy 3: Mode imputation (categorical)
imputer_cat = SimpleImputer(strategy="most_frequent")

# Strategy 4: Forward fill (time series)
df["price"].fillna(method="ffill", inplace=True)
```

**Rules:**
- **<5% missing** → impute karo
- **5-30% missing** → impute + missing indicator add karo
- **>30% missing** → column drop karne ka socho

> ⚠️ **Data Leakage:** Imputer ko sirf `X_train` par **fit** karo, `X_test` par sirf **transform**.

## 6.2 Duplicate Rows Remove Karna

```python
print("Duplicates:", df.duplicated().sum())
df = df.drop_duplicates()
print("After:", df.duplicated().sum())
```

## 6.3 Outliers Detect Aur Handle Karna

**Detection Methods:**

```python
# Method 1: IQR (Interquartile Range)
Q1 = df["price"].quantile(0.25)
Q3 = df["price"].quantile(0.75)
IQR = Q3 - Q1
lower = Q1 - 1.5 * IQR
upper = Q3 + 1.5 * IQR

outliers = df[(df["price"] < lower) | (df["price"] > upper)]
print(f"Outliers: {len(outliers)}")

# Method 2: Z-score
from scipy import stats
z_scores = stats.zscore(df["price"])
outliers_z = df[abs(z_scores) > 3]

# Method 3: Visual
import matplotlib.pyplot as plt
plt.boxplot(df["price"])
plt.show()
```

**Treatment:**

```python
# Option 1: Remove
df_clean = df[(df["price"] >= lower) & (df["price"] <= upper)]

# Option 2: Cap (Winsorization)
df["price_capped"] = df["price"].clip(lower=lower, upper=upper)

# Option 3: Log transform (right-skewed data)
import numpy as np
df["price_log"] = np.log1p(df["price"])
```

**Kaunsa Outlier Remove Karein?**
- **Data entry error** (e.g., price = 99999 lakh) → Remove
- **Genuine extreme** (e.g., luxury mansion) → Rakhna, lekin alag model ya feature
- **Investigate pehle**, blindly remove nahi

## 6.4 Wrong Data Types Correct Karna

```python
# Check types
print(df.dtypes)

# String numbers ko numeric banao
df["bedrooms"] = pd.to_numeric(df["bedrooms"], errors="coerce")

# Dates
df["date"] = pd.to_datetime(df["date"])

# Category type
df["location"] = df["location"].astype("category")
```

## 6.5 Irrelevant Features Remove Karna

```python
# Constant columns
constant_cols = [col for col in df.columns if df[col].nunique() == 1]
df = df.drop(columns=constant_cols)

# ID-like columns
id_cols = [col for col in df.columns if "id" in col.lower()]
df = df.drop(columns=id_cols)

# High cardinality (too many unique)
for col in df.select_dtypes(include="object"):
    if df[col].nunique() > 50:
        print(f"High cardinality: {col} ({df[col].nunique()} unique)")
```

## 6.6 Useful Features Add Karna (Feature Engineering)

```python
# Ratio features
df["price_per_marla"] = df["price_lakhs"] / df["area_marla"]
df["rooms_per_bathroom"] = df["bedrooms"] / df["bathrooms"].replace(0, 1)

# Interaction features
df["area_x_bedrooms"] = df["area_marla"] * df["bedrooms"]

# Polynomial features
df["area_squared"] = df["area_marla"] ** 2
df["age_squared"] = df["house_age_years"] ** 2

# Date features (agar date column ho)
df["year"] = df["date"].dt.year
df["month"] = df["date"].dt.month
```

## 6.7 Categorical Data Encoding

```python
# Method 1: Label Encoding (ordinal – order matters)
from sklearn.preprocessing import LabelEncoder
le = LabelEncoder()
df["location_encoded"] = le.fit_transform(df["location"])

# Method 2: One-Hot Encoding (nominal – no order)
df = pd.get_dummies(df, columns=["location"], drop_first=True)

# Method 3: Ordinal Encoding (custom order)
from sklearn.preprocessing import OrdinalEncoder
ord_enc = OrdinalEncoder(categories=[["Low", "Medium", "High"]])
df["quality_encoded"] = ord_enc.fit_transform(df[["quality"]])
```

**Kaunsa Kab?**

| Encoding | Kab Use | Example |
|----------|---------|---------|
| Label | Ordered categories | Low/Medium/High |
| One-Hot | No order | City names |
| Ordinal | Custom order | Education level |

> ⚠️ **Rule:** One-Hot Encoding **train/test split ke baad** karo, warna data leakage.

## 6.8 Feature Scaling Kab Zaroori Hoti Hai?

**Linear Regression mein scaling zaroori NAHI** (coefficients interpret karna ho to useful hai).

**Lekin zaroori hai jab:**
- **Regularization** (Ridge/Lasso) use kar rahe ho
- **Gradient Descent** based optimization ho
- Features ke scales **bahut different** hon (e.g., age 0-100, price 1-100000)

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)  # ⚠️ Sirf transform!
```

## 6.9 Multicollinearity

**Problem:** Do ya zyada features **ek doosre se highly correlated** hain.

**Detection:**
```python
# Correlation matrix
corr_matrix = X_train.corr().abs()
print(corr_matrix)

# VIF (Variance Inflation Factor)
from statsmodels.stats.outliers_influence import variance_inflation_factor
vif_data = pd.DataFrame()
vif_data["Feature"] = X_train.columns
vif_data["VIF"] = [variance_inflation_factor(X_train.values, i) for i in range(X_train.shape[1])]
print(vif_data)
```

**VIF Rules:**
- VIF < 5 → OK
- VIF 5-10 → Moderate (investigate)
- VIF > 10 → **Serious multicollinearity** (remove one)

**Solution:** Correlated features mein se ek remove karo.

## 6.10 Non-Linear Relationships

**Detection:** Scatter plot dekho. Agar relationship **curve** hai, linear model fit nahi hoga.

**Solution:**
```python
from sklearn.preprocessing import PolynomialFeatures

poly = PolynomialFeatures(degree=2, include_bias=False)
X_train_poly = poly.fit_transform(X_train)
X_test_poly = poly.transform(X_test)

llr_poly = LinearRegression()
llr_poly.fit(X_train_poly, y_train)
print("Poly Test Score:", llr_poly.score(X_test_poly, y_test))
```

## 6.11 Underfitting

**Symptoms:** Train low, Test low

**Solutions:**
1. More/better features
2. Polynomial features
3. Complex model (Random Forest, Gradient Boosting)
4. Less regularization

## 6.12 Overfitting

**Symptoms:** Train high, Test low (bada gap)

**Solutions:**
1. Less features
2. Regularization (Ridge/Lasso)
3. Cross-validation
4. More data
5. Simplify model

## 6.13 Data Leakage

**Kya Hai?** Model ko training ke waqt aisi info mil jaye jo **prediction ke waqt available nahi hogi**.

**Examples:**
- Target ki info X mein
- Test data ki info training mein (scaling/imputation split se pehle)
- Future data use karna

**Detection:**
```python
# Agar test score suspiciously high ho
if test_score > 0.99:
    print("⚠️ Data leakage check karo!")
```

**Solution:**
```python
# ✅ SAHI: Split pehle, phir preprocess
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)   # fit on train only
X_test_scaled = scaler.transform(X_test)          # transform only

# ❌ GALAT: Preprocess pehle, phir split
X_scaled = scaler.fit_transform(X)  # LEAKAGE!
X_train, X_test = train_test_split(X_scaled, ...)
```

## 6.14 Insufficient Data

**Symptoms:** High variance in scores, unstable model

**Solutions:**
1. More data collect karo
2. Data augmentation
3. Cross-validation
4. Simple model use karo

## 6.15 Train/Test Distribution Differences

**Detection:**
```python
# Training vs testing statistics compare
print("Train mean:", X_train.mean())
print("Test mean:", X_test.mean())

# Visual comparison
import matplotlib.pyplot as plt
plt.hist(X_train["area"], alpha=0.5, label="Train")
plt.hist(X_test["area"], alpha=0.5, label="Test")
plt.legend()
plt.show()
```

**Solution:** Stratified split use karo (categorical ke liye) ya ensure karo split random ho.

### 📝 Exercise 6

Ek dataset mein ye problems hain. Har ek ka solution likho:
1. `price` column mein 15% missing values
2. `bedrooms` mein 6% values `"3"` string format mein
3. `location` mein 20 unique cities
4. Train=92%, Test=48%
5. `area` aur `area_sqft` dono columns hain (same info)

<details>
<summary>Answer</summary>

1. Median imputation + missing indicator
2. `pd.to_numeric(..., errors="coerce")`
3. One-Hot Encoding (ya top cities + "Other")
4. Overfitting → regularization, feature selection, CV
5. Multicollinearity → ek column remove karo (e.g., area_sqft)
</details>

---

# Chapter 7: Model Evaluation Metrics

## 7.1 Code

```python
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score

y_pred = llr.predict(x_test)

print("MAE:", mean_absolute_error(y_test, y_pred))
print("MSE:", mean_squared_error(y_test, y_pred))
print("RMSE:", mean_squared_error(y_test, y_pred) ** 0.5)
print("R2 Score:", r2_score(y_test, y_pred))
```

## 7.2 Har Metric Ka Explanation

### R² Score

```
R² = 1 - (SS_res / SS_total)
```

- **Range:** -∞ to 1
- **Best:** 1.0
- **Meaning:** Model **kitna % variance** explain kar raha hai
- **Kab use:** Overall model fit judge karne ke liye

### MAE (Mean Absolute Error)

```
MAE = (1/n) × Σ|actual - predicted|
```

- **Unit:** Target ke same unit mein (lakh, rupees)
- **Meaning:** Average prediction **kitna wrong** hai
- **Example:** MAE = 8 lakh → predictions average 8 lakh off hain
- **Kab use:** Jab outliers **kam** hon aur simple interpretation chahiye

### MSE (Mean Squared Error)

```
MSE = (1/n) × Σ(actual - predicted)²
```

- **Unit:** Target ka **square** (lakh²)
- **Meaning:** Errors ko square karke average
- **Problem:** Unit interpret karna mushkil
- **Kab use:** Optimization (gradient descent) mein

### RMSE (Root Mean Squared Error)

```
RMSE = √MSE
```

- **Unit:** Target ke same unit (lakh)
- **Meaning:** Error ka standard deviation jaisa
- **Kab use:** Jab **outliers** important hon

## 7.3 MAE vs RMSE – Key Difference

| Aspect | MAE | RMSE |
|--------|-----|------|
| **Outliers** | Less affected | **More affected** (square karta hai) |
| **Interpretation** | Direct average error | Penalizes large errors |
| **Use when** | Outliers kam/ignore | Outliers important |

**Example:**
```
Predictions: [10, 20, 100]
Errors: [2, 3, 50]  ← ek bada error

MAE = (2+3+50)/3 = 18.3
RMSE = √((4+9+2500)/3) = √837.7 = 28.9  ← Bada error zyada penalize
```

## 7.4 Kis Metric Ko Kab Use Karna Chahiye?

| Situation | Metric |
|-----------|--------|
| **Overall model fit** | R² |
| **Simple error interpretation** | MAE |
| **Outliers matter** | RMSE |
| **Compare models** | R² + RMSE |
| **Business reporting** | MAE (easy to explain) |

## 7.5 Kya High R² Hamesha Good Model Hota Hai?

**❌ Nahi!** Reasons:

1. **Overfitting:** R² train par high, test par low
2. **Data leakage:** R² artificially high
3. **Wrong target:** R² high lekin business problem solve nahi
4. **Small dataset:** R² unreliable

**Example:**
```
Model A: R² = 0.95, MAE = 15 lakh  ← High R² lekin bada error
Model B: R² = 0.85, MAE = 3 lakh   ← Lower R² lekin practical better
```

Business ke liye **Model B** better ho sakta hai!

### 📝 Exercise 7

```python
y_true = [100, 200, 300, 400, 500]
y_pred = [110, 190, 310, 380, 520]
```

Calculate karo: MAE, MSE, RMSE, R²

<details>
<summary>Answer</summary>

Errors: [10, -10, 10, -20, 20]
- MAE = (10+10+10+20+20)/5 = **14**
- MSE = (100+100+100+400+400)/5 = **220**
- RMSE = √220 = **14.83**
- Mean = 300
- SS_res = 100+100+100+400+400 = 1100
- SS_total = (200²+100²+0²+100²+200²) = 40000+10000+0+10000+40000 = 100000
- R² = 1 - (1100/100000) = **0.989**
</details>

---

# Chapter 8: Prediction – New Client Data

## 8.1 Code

```python
predict = llr.predict([[7, 3, 5]])
print("Predicted Price (Lakhs):", predict)
```

## 8.2 Har Number Ka Meaning

**Assume features order:** `["area_marla", "bedrooms", "house_age_years"]`

- `7` = **area_marla** = 7 marla ka ghar
- `3` = **bedrooms** = 3 bedrooms
- `5` = **house_age_years** = 5 saal purana

## 8.3 New Client Data Isi Order Mein Kyun?

**Model ne training ke waqt is order mein features seekhe:**
```
Position 0: area_marla
Position 1: bedrooms
Position 2: house_age_years
```

**Agar order change ho jaye:**
```python
# ❌ GALAT: [bedrooms, area, age]
predict = llr.predict([[3, 7, 5]])  # Galat prediction!
```

Model samjhega:
- bedrooms = 3 ✅ (lekin ye area ki jagah hai!)
- area = 7 (lekin ye bedrooms ki jagah hai!)

**Result:** Bilkul **galat prediction**.

## 8.4 Features Ordering Change Ho To Kya Hoga?

**Solution:** Feature names ke saath DataFrame pass karo:

```python
import pandas as pd

new_house = pd.DataFrame({
    "area_marla": [7],
    "bedrooms": [3],
    "house_age_years": [5]
})

predict = llr.predict(new_house)
print("Predicted:", predict)
```

**Best Practice:** **Hamesha DataFrame** use karo prediction ke liye, list nahi. Isse:
1. Order ka issue nahi
2. Feature names match hote hain
3. Warnings nahi aate

## 8.5 Real Client Ke Naye House Data Ki Price Kaise Predict Karein?

**Complete Workflow:**

```python
# Step 1: Naya data create karo (same columns, same order)
new_house = pd.DataFrame({
    "area_marla": [10, 5, 7],
    "bedrooms": [4, 2, 3],
    "house_age_years": [2, 10, 5],
    "bathrooms": [3, 1, 2],
    "parking_spaces": [2, 1, 1],
    "location": ["DHA", "Gulberg", "Bahria"]
})

# Step 2: Same preprocessing apply karo (encoding, scaling)
# Agar training mein One-Hot kiya tha:
new_house_encoded = pd.get_dummies(new_house, columns=["location"])
# ⚠️ Ensure karo ke columns training ke saath match karein
new_house_encoded = new_house_encoded.reindex(columns=X_train.columns, fill_value=0)

# Step 3: Predict
predictions = llr.predict(new_house_encoded)

# Step 4: Results dikhao
for i, price in enumerate(predictions):
    print(f"House {i+1}: {price:.2f} Lakhs")
```

## 8.6 Prediction Ko Actual Ke Saath Compare Karna

```python
# Testing data par predictions
y_pred = llr.predict(x_test)

# Comparison DataFrame
comparison = pd.DataFrame({
    "Actual": y_test.values,
    "Predicted": y_pred,
    "Difference": y_test.values - y_pred,
    "Abs_Error": abs(y_test.values - y_pred)
})

print(comparison.head(10))

# Percentage error
comparison["Pct_Error"] = (comparison["Abs_Error"] / comparison["Actual"]) * 100

# Average percentage error
print(f"Avg % Error: {comparison['Pct_Error'].mean():.2f}%")
```

**Output Example:**
```
   Actual  Predicted  Difference  Abs_Error  Pct_Error
0   150.0      145.2         4.8        4.8       3.2%
1    80.0       92.1       -12.1       12.1      15.1%
2   120.0      118.5         1.5        1.5       1.3%
```

### 📝 Exercise 8

Model features: `["area", "beds", "age"]`, trained coefficients: `[10, 5, -2]`, intercept: `20`

Naya ghar: area=8, beds=3, age=4

**Questions:**
1. Prediction kya hogi?
2. Agar features order `[beds, area, age]` ho jaye to kya hoga?

<details>
<summary>Answer</summary>

1. `20 + 10(8) + 5(3) + (-2)(4) = 20 + 80 + 15 - 8 = 107 Lakhs`
2. Galat prediction: `20 + 10(3) + 5(8) + (-2)(4) = 20 + 30 + 40 - 8 = 82 Lakhs` (galat!)
</details>

---

# Chapter 9: All Possible Linear Regression Problems – Troubleshooting Guide

| Problem | Reason | Solution | Code |
|---------|--------|----------|------|
| **Training score low** | Underfitting | Better features/model | Polynomial features, complex model |
| **Testing score low** | Poor generalization | Data improvement | More data, CV |
| **Training high, testing low** | Overfitting | Regularization | Ridge/Lasso, feature selection |
| **Negative R²** | Poor predictions | Diagnose data | Check features, outliers |
| **Missing values** | Incomplete data | Imputation | SimpleImputer |
| **Outliers** | Extreme values | Investigation + treatment | IQR, capping |
| **Data leakage** | Future info in X | Correct splitting | Split before preprocess |
| **Categorical columns** | Text data | Encoding | OneHotEncoder |
| **Wrong predictions** | Data/model issues | Debugging | Check feature order |

## 9.1 Practical Solution – Har Problem Ka

### Problem 1: Training Score Low

```python
# Diagnose
print("Train:", llr.score(X_train, y_train))

# Solution: Polynomial features
poly = PolynomialFeatures(degree=2, include_bias=False)
X_train_poly = poly.fit_transform(X_train)
X_test_poly = poly.transform(X_test)

llr2 = LinearRegression().fit(X_train_poly, y_train)
print("After:", llr2.score(X_train_poly, y_train))
```

### Problem 2: Data Leakage

```python
# ❌ GALAT
X_scaled = StandardScaler().fit_transform(X)
X_train, X_test = train_test_split(X_scaled, ...)

# ✅ SAHI
X_train, X_test, y_train, y_test = train_test_split(X, y, ...)
scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)
```

### Problem 3: Categorical Columns

```python
# Check
cat_cols = X.select_dtypes(include="object").columns
print(cat_cols)

# Encode
X_encoded = pd.get_dummies(X, columns=cat_cols, drop_first=True)
```

### Problem 4: Wrong Predictions

```python
# Debug checklist
print("Feature order:", X_train.columns.tolist())
print("Model expects:", llr.n_features_in_)
print("Input shape:", new_data.shape)

# Compare
print("Sample train:", X_train.iloc[0].values)
print("New input:", new_data.values)
```

---

# Chapter 10: Complete Real-World House Price Prediction Project

## 10.1 Sample Dataset Generate Karna

```python
import numpy as np
import pandas as pd

np.random.seed(42)
n = 500

# Realistic data generate karo
data = {
    "area_marla": np.random.randint(3, 25, n),
    "bedrooms": np.random.randint(1, 7, n),
    "house_age_years": np.random.randint(0, 30, n),
    "bathrooms": np.random.randint(1, 5, n),
    "location": np.random.choice(["DHA", "Gulberg", "Bahria", "Model Town", "Johar Town"], n),
    "parking_spaces": np.random.randint(0, 3, n),
}

df = pd.DataFrame(data)

# Realistic price formula + noise
location_multiplier = {"DHA": 1.5, "Gulberg": 1.3, "Bahria": 1.1, "Model Town": 1.2, "Johar Town": 1.0}
df["price_lakhs"] = (
    10
    + df["area_marla"] * 8
    + df["bedrooms"] * 5
    + df["bathrooms"] * 3
    - df["house_age_years"] * 0.5
    + df["parking_spaces"] * 2
    + df["location"].map(location_multiplier) * 20
    + np.random.normal(0, 10, n)  # noise
)

# Problems inject karo
# 1. Missing values
df.loc[np.random.choice(n, 20, replace=False), "area_marla"] = np.nan
df.loc[np.random.choice(n, 10, replace=False), "bathrooms"] = np.nan

# 2. Duplicates
df = pd.concat([df, df.sample(10)], ignore_index=True)

# 3. Outliers
df.loc[0, "price_lakhs"] = 5000  # unrealistic
df.loc[1, "area_marla"] = 200     # unrealistic

# 4. Wrong data type
df["bedrooms"] = df["bedrooms"].astype(str)

print(df.shape)
print(df.isnull().sum())
print(df.duplicated().sum())
```

## 10.2 Step-by-Step Complete Pipeline

```python
# ═══════════════════════════════════════════════════
# STEP 1: DATA LOAD & INSPECT
# ═══════════════════════════════════════════════════
df = pd.read_csv("house_prices.csv")
print(df.head())
print(df.info())
print(df.describe())
print("Missing:\n", df.isnull().sum())
print("Duplicates:", df.duplicated().sum())

# ═══════════════════════════════════════════════════
# STEP 2: DATA CLEANING
# ═══════════════════════════════════════════════════

# 2a. Duplicates remove
df = df.drop_duplicates().reset_index(drop=True)
print("After dedup:", df.shape)

# 2b. Wrong data types fix
df["bedrooms"] = pd.to_numeric(df["bedrooms"], errors="coerce")

# 2c. Outliers handle
Q1 = df["price_lakhs"].quantile(0.25)
Q3 = df["price_lakhs"].quantile(0.75)
IQR = Q3 - Q1
lower, upper = Q1 - 1.5*IQR, Q3 + 1.5*IQR
df = df[(df["price_lakhs"] >= lower) & (df["price_lakhs"] <= upper)]
print("After outlier removal:", df.shape)

# 2d. Missing values handle
df["area_marla"] = df["area_marla"].fillna(df["area_marla"].median())
df["bathrooms"] = df["bathrooms"].fillna(df["bathrooms"].median())
print("Missing after:", df.isnull().sum().sum())

# ═══════════════════════════════════════════════════
# STEP 3: FEATURE SELECTION
# ═══════════════════════════════════════════════════
features = ["area_marla", "bedrooms", "house_age_years", 
            "bathrooms", "location", "parking_spaces"]
target = "price_lakhs"

X = df[features]
y = df[target]

# ═══════════════════════════════════════════════════
# STEP 4: TRAIN/TEST SPLIT (BEFORE ENCODING!)
# ═══════════════════════════════════════════════════
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)
print("Train:", X_train.shape, "Test:", X_test.shape)

# ═══════════════════════════════════════════════════
# STEP 5: ENCODING (AFTER SPLIT!)
# ═══════════════════════════════════════════════════
X_train_enc = pd.get_dummies(X_train, columns=["location"], drop_first=True)
X_test_enc = pd.get_dummies(X_test, columns=["location"], drop_first=True)

# Align columns
X_test_enc = X_test_enc.reindex(columns=X_train_enc.columns, fill_value=0)

print("Encoded columns:", X_train_enc.columns.tolist())

# ═══════════════════════════════════════════════════
# STEP 6: MODEL TRAINING
# ═══════════════════════════════════════════════════
from sklearn.linear_model import LinearRegression
llr = LinearRegression()
llr.fit(X_train_enc, y_train)

# ═══════════════════════════════════════════════════
# STEP 7: SCORES
# ═══════════════════════════════════════════════════
train_score = llr.score(X_train_enc, y_train)
test_score = llr.score(X_test_enc, y_test)
print(f"Train R²: {train_score:.4f} ({train_score*100:.2f}%)")
print(f"Test R²:  {test_score:.4f} ({test_score*100:.2f}%)")

# ═══════════════════════════════════════════════════
# STEP 8: EVALUATION METRICS
# ═══════════════════════════════════════════════════
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score

y_pred = llr.predict(X_test_enc)
mae = mean_absolute_error(y_test, y_pred)
mse = mean_squared_error(y_test, y_pred)
rmse = mse ** 0.5
r2 = r2_score(y_test, y_pred)

print(f"MAE:  {mae:.2f} Lakhs")
print(f"MSE:  {mse:.2f}")
print(f"RMSE: {rmse:.2f} Lakhs")
print(f"R²:   {r2:.4f}")

# ═══════════════════════════════════════════════════
# STEP 9: ACTUAL VS PREDICTED
# ═══════════════════════════════════════════════════
comparison = pd.DataFrame({
    "Actual": y_test.values,
    "Predicted": y_pred,
    "Error": y_test.values - y_pred
})
print(comparison.head(10))

# ═══════════════════════════════════════════════════
# STEP 10: NEW HOUSE PREDICTION
# ═══════════════════════════════════════════════════
new_house = pd.DataFrame({
    "area_marla": [10],
    "bedrooms": [3],
    "house_age_years": [5],
    "bathrooms": [2],
    "location": ["DHA"],
    "parking_spaces": [1]
})

new_house_enc = pd.get_dummies(new_house, columns=["location"], drop_first=True)
new_house_enc = new_house_enc.reindex(columns=X_train_enc.columns, fill_value=0)

predicted_price = llr.predict(new_house_enc)[0]
print(f"\n🏠 New House Prediction: {predicted_price:.2f} Lakhs")

# ═══════════════════════════════════════════════════
# STEP 11: MODEL IMPROVEMENT
# ═══════════════════════════════════════════════════

# 11a. Feature engineering
X_train_fe = X_train_enc.copy()
X_test_fe = X_test_enc.copy()
X_train_fe["area_x_bedrooms"] = X_train_fe["area_marla"] * X_train_fe["bedrooms"]
X_test_fe["area_x_bedrooms"] = X_test_fe["area_marla"] * X_test_fe["bedrooms"]

llr_fe = LinearRegression().fit(X_train_fe, y_train)
print("With FE - Test R²:", llr_fe.score(X_test_fe, y_test))

# 11b. Regularization
from sklearn.linear_model import Ridge
ridge = Ridge(alpha=1.0)
ridge.fit(X_train_enc, y_train)
print("Ridge - Train:", ridge.score(X_train_enc, y_train))
print("Ridge - Test:", ridge.score(X_test_enc, y_test))

# 11c. Cross-validation
from sklearn.model_selection import cross_val_score
cv_scores = cross_val_score(LinearRegression(), X_train_enc, y_train, cv=5)
print(f"CV Mean: {cv_scores.mean():.4f} (+/- {cv_scores.std():.4f})")

# ═══════════════════════════════════════════════════
# STEP 12: FINAL MODEL SAVE
# ═══════════════════════════════════════════════════
import joblib
joblib.dump(llr, "final_house_price_model.pkl")
print("✅ Model saved!")

# Load test
loaded_model = joblib.load("final_house_price_model.pkl")
print("Loaded model test score:", loaded_model.score(X_test_enc, y_test))
```

## 10.3 Expected Output

```
Train R²: 0.9123 (91.23%)
Test R²:  0.8956 (89.56%)
MAE:  8.45 Lakhs
MSE:  112.34
RMSE: 10.60 Lakhs
R²:   0.8956

🏠 New House Prediction: 152.34 Lakhs

With FE - Test R²: 0.9012
Ridge - Train: 0.9120
Ridge - Test: 0.8960
CV Mean: 0.8934 (+/- 0.0156)
```

---

# Chapter 11: Teaching Summary – Common Mistakes & Exercises

## 11.1 Common Mistakes (Aur Unke Solutions)

| # | Mistake | Solution |
|---|---------|----------|
| 1 | `X = df["area"]` (single bracket) | `X = df[["area"]]` (double bracket) |
| 2 | Preprocess pehle, split baad mein | **Split pehle**, preprocess baad mein |
| 3 | Imputer test par `fit` karna | Test par sirf `transform` |
| 4 | Categorical encode nahi karna | OneHot/Label Encoding |
| 5 | Feature order change karna | DataFrame use karo, list nahi |
| 6 | R² ko accuracy samajhna | R² = variance explained |
| 7 | Outliers blindly remove | Investigate pehle |
| 8 | Test score high dekh kar khush hona | Train-test gap check karo |
| 9 | `random_state` na dena | Hamesha seed do (42) |
| 10 | Sari columns X mein dalna | Feature selection karo |

## 11.2 Complete Exercises

### Exercise A: Data Understanding
Diya gaya dataset: `id, name, age, city, income, expenses, savings`

1. Target kya hoga? (savings predict karna hai)
2. Kaunse columns remove karoge?
3. Categorical kaunsa hai?
4. Kya koi data leakage hai?

### Exercise B: Split
500 rows, test_size=0.15, random_state=42

1. Training rows?
2. Testing rows?
3. Kya random_state=42 aur 43 se same split milega?

### Exercise C: Score Interpretation
| Train | Test | Diagnosis | Action |
|-------|------|-----------|--------|
| 92% | 88% | ? | ? |
| 45% | 42% | ? | ? |
| 98% | 30% | ? | ? |
| 70% | 72% | ? | ? |

### Exercise D: Metrics
```
y_true = [50, 60, 70, 80, 90]
y_pred = [55, 58, 72, 78, 95]
```
Calculate: MAE, MSE, RMSE, R²

### Exercise E: Prediction
Model: `price = 15 + 8*area + 4*beds - 1*age`
Naya ghar: area=12, beds=4, age=6

1. Price predict karo
2. Agar area aur beds swap ho jayein to?

## 11.3 Final Checklist – Client Data Aane Par

```
□ Data load karo (read_excel/read_csv)
□ head(), info(), describe() dekho
□ Missing values check karo
□ Duplicates check karo
□ Outliers investigate karo
□ Data types correct karo
□ Irrelevant columns remove karo
□ Target column identify karo
□ Features select karo
□ Train/test split karo (BEFORE preprocessing)
□ Missing values impute karo (fit on train)
□ Categorical encode karo
□ Model train karo
□ Train/Test score check karo
□ Metrics calculate karo (MAE, RMSE, R²)
□ Actual vs predicted compare karo
□ Improve karo (features, regularization, CV)
□ Final model evaluate karo
□ New client data predict karo
□ Model save karo (joblib)
```

---

## 🎯 Final Goal Achieved?

Ab tum **independently** kar sakte ho:

| Skill | Status |
|-------|--------|
| Dataset understand karna | ✅ |
| Features/target select karna | ✅ |
| Train/test split karna | ✅ |
| Model train karna | ✅ |
| Scores interpret karna (R² vs accuracy) | ✅ |
| Low performance diagnose karna | ✅ |
| Problems solve karna | ✅ |
| Model evaluate karna (MAE, RMSE, R²) | ✅ |
| New data predict karna | ✅ |
| Overfitting/Underfitting handle karna | ✅ |
| Data leakage avoid karna | ✅ |

---

## 💡 Ek Aakhri Baat

> **"Sirf score increase karne ke liye testing data ko manipulate mat karna."**
>
> Model ko **genuine unseen data** par evaluate karo. Agar test score artificially high hai, to real-world mein model **fail** karega. **Honest evaluation** hi real improvement ki taraf le jaati hai.


