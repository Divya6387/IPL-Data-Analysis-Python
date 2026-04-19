# 🏏 IPL Data Analysis — Python Portfolio Project
## Project Overview

This project performs a complete analysis of the IPL (Indian Premier League) ball-by-ball dataset. It follows a real-world data analyst workflow — from raw messy data to clean insights and visual dashboards.

**Dataset** : IPL Ball-by-Ball Data  
**Rows**    : 278,205  
**Columns** : 64  
**Source**  : [Kaggle — IPL Dataset](https://www.kaggle.com/)

---

##  Insights Discovered

| # | Question | Answer |
|---|----------|--------|
| 1 |  Who scored the most runs? | Virat Kohli leads all-time |
| 2 |  Who took the most wickets? | Dwayne Bravo leads dismissals |
| 3 |  Does toss winner win the match? | Only ~51% — nearly irrelevant |
| 4 |  How have runs trended across seasons? | Increasing year over year |
| 5 |  Which over phase scores most? | Death overs (16-20) are highest |

---

## Tools & Technologies

| Tool | Version | Purpose |
|------|---------|---------|
| Python | 3.10+ | Programming language |
| Pandas | 2.0+ | Data loading & manipulation |
| NumPy | 1.24+ | Numerical operations |
| Matplotlib | 3.7+ | Base plotting |
| Seaborn | 0.12+ | Statistical visualizations |
| Jupyter Notebook | 1.0+ | Interactive analysis environment |
| Power BI / Tableau | Latest | Interactive dashboard (see guide below) |

---


## How to Run This Project

### Step 1 — Clone the repository
```bash
git clone https://github.com/YOUR_USERNAME/ipl-data-analysis-python.git
cd ipl-data-analysis-python
```

### Step 2 — Install dependencies
```bash
pip install -r requirements.txt
```

### Step 3 — Add the dataset
Download `IPL.csv` from Kaggle and place it in the project folder.

### Step 4 — Launch Jupyter Notebook
```bash
jupyter notebook
```
Open `IPL_Data_Analysis.ipynb` → Kernel → Restart & Run All

---

## Python Coding — Step-by-Step Walkthrough

> This section explains every Python coding step used in this project with real code.  
> Each step builds on the previous one — follow them in order.

---

## Step 1 — Import Libraries

Every Python data analysis project starts by importing libraries.  
Think of libraries as toolboxes — each one gives you special powers.

```python
import pandas as pd           # Load and manipulate data tables
import numpy as np            # Fast numerical calculations
import sqlite3                # Built-in SQL database (no install needed)
import matplotlib.pyplot as plt  # Draw charts and graphs
import seaborn as sns         # Beautiful statistical visualizations
import warnings
warnings.filterwarnings('ignore')   # Hide harmless warning messages

# Make all charts look clean and large by default
plt.rcParams['figure.figsize'] = (13, 6)
plt.rcParams['font.size']      = 12
sns.set_theme(style='whitegrid', palette='muted')

print(" All libraries imported!")
```

> **Why each library?**
> - `pandas` — without this, we can't even open the CSV file
> - `numpy` — used behind the scenes for fast maths
> - `sqlite3` — lets us write SQL queries directly in Python
> - `matplotlib` + `seaborn` — every chart in this project uses these two

---

## Step 2 — Load the Dataset

```python
# pd.read_csv() reads a CSV file and creates a DataFrame (like an Excel table)
# low_memory=False prevents errors on large files with mixed column types

df = pd.read_csv('IPL.csv', low_memory=False)

# Check how big the data is
print(f"Rows    : {df.shape[0]:,}")    # df.shape[0] = number of rows
print(f"Columns : {df.shape[1]}")      # df.shape[1] = number of columns
```

> **What is a DataFrame?**  
> A DataFrame is a 2D table with rows and columns — exactly like an Excel sheet,  
> but stored in Python memory and much faster to work with.

---

## Step 3 — Explore the Dataset (EDA)

Before touching the data, always understand what you have.  
This is called **Exploratory Data Analysis (EDA)**.

```python
# See the first 5 rows
df.head()

# See the last 5 rows
df.tail()

# Column names, data types, and how many non-null values exist in each column
df.info()

# Statistical summary — count, mean, std, min, max for all numeric columns
df.describe()

# Count missing (null/NaN) values per column
missing = df.isnull().sum()
missing_pct = (missing / len(df) * 100).round(2)

# Combine into a readable report
missing_report = pd.DataFrame({
    'Missing Count': missing,
    'Missing (%)': missing_pct
}).query('`Missing Count` > 0').sort_values('Missing (%)', ascending=False)

print(missing_report)
```

> **Key things to look for:**
> - Columns with 90%+ missing values → usually safe to drop
> - Columns stored as `object` (text) that should be numbers → need fixing
> - Duplicate rows → need removing before analysis

---

## Step 4 — Data Cleaning

Data cleaning is the **most important step** in any real project.  
Real-world data is always messy. Here is how we fix it:

```python
# RULE: Always work on a COPY — never edit the original df
df_clean = df.copy()

# 4a. Remove duplicate rows 
before = len(df_clean)
df_clean.drop_duplicates(inplace=True)
print(f"Duplicates removed: {before - len(df_clean)}")

# 4b. Fill missing values in categorical columns 
# These columns are empty when the event didn't happen (no wicket, no extra)
# Filling with 'None' makes this explicit instead of leaving NaN

fill_cols = ['extra_type', 'wicket_kind', 'player_out',
             'fielders', 'superover_winner', 'result_type', 'method']

for col in fill_cols:
    if col in df_clean.columns:
        df_clean[col] = df_clean[col].fillna('None')

# 4c. Drop columns that are more than 95% empty 
# A column with 95%+ missing values adds noise, not information
threshold = 0.95
drop_cols = [
    col for col in df_clean.columns
    if df_clean[col].isnull().sum() / len(df_clean) > threshold
]
df_clean.drop(columns=drop_cols, inplace=True)
print(f"Columns dropped: {drop_cols}")

# 4d. Fix data types 
# Date column must be datetime type, not plain string
df_clean['date'] = pd.to_datetime(df_clean['date'], errors='coerce')

# Numeric columns must be float/int — errors='coerce' turns bad values into NaN
df_clean['runs_batter'] = pd.to_numeric(df_clean['runs_batter'], errors='coerce').fillna(0)
df_clean['runs_total']  = pd.to_numeric(df_clean['runs_total'],  errors='coerce').fillna(0)
df_clean['balls_faced'] = pd.to_numeric(df_clean['balls_faced'], errors='coerce').fillna(0)

# 4e. Extract year from season column 
# Season values look like "2007/08" — we extract just the 4-digit year
df_clean['season_year'] = df_clean['season'].astype(str).str[:4].str.strip()
df_clean = df_clean[df_clean['season_year'].str.match(r'^\d{4}$')]

print(f" Clean dataset: {df_clean.shape[0]:,} rows × {df_clean.shape[1]} columns")
```

---

## Step 5 — Load Data into SQLite (SQL inside Python)

```python
import sqlite3

# Create a database file (or connect to existing one)
conn   = sqlite3.connect('ipl_database.db')
cursor = conn.cursor()

# Load the clean DataFrame as a SQL table named 'ipl'
# if_exists='replace' → overwrites the table if it already exists
df_clean.to_sql('ipl', conn, if_exists='replace', index=False)

# Verify: query the table using SQL
row_count = pd.read_sql("SELECT COUNT(*) AS total FROM ipl", conn)
print(f"Rows in SQL table: {row_count['total'][0]:,}")
```

> **Why load into SQL?**  
> In real jobs, data lives in databases — not CSV files.  
> `pd.read_sql()` lets you write SQL queries and get results back as a DataFrame.

---

## Step 6 — Extract Insights Using SQL + Python

```python
# ── Insight 1: Top 10 Run Scorers ─────────────────────────────────────
top_scorers = pd.read_sql("""
    SELECT
        batter                   AS Batter,
        SUM(runs_batter)         AS Total_Runs,
        COUNT(DISTINCT match_id) AS Matches
    FROM ipl
    GROUP BY batter
    ORDER BY Total_Runs DESC
    LIMIT 10
""", conn)
print(top_scorers)

# ── Insight 2: Top 10 Wicket Takers ───────────────────────────────────
top_bowlers = pd.read_sql("""
    SELECT bowler, COUNT(*) AS Wickets
    FROM ipl
    WHERE wicket_kind NOT IN ('None','run out','retired hurt')
    GROUP BY bowler
    ORDER BY Wickets DESC
    LIMIT 10
""", conn)

# ── Insight 3: Toss Impact ────────────────────────────────────────────
toss_data = pd.read_sql("""
    SELECT match_id, toss_winner, match_won_by
    FROM ipl
    GROUP BY match_id
    HAVING match_won_by IS NOT NULL AND toss_winner IS NOT NULL
""", conn)

# Python logic: did the toss winner also win the match?
toss_data['toss_won_match'] = (toss_data['toss_winner'] == toss_data['match_won_by'])
win_pct = toss_data['toss_won_match'].mean() * 100
print(f"Toss winner win rate: {win_pct:.1f}%")
```

---

## Step 7 — Visualise Insight 1 : Top 10 Run Scorers

```python
fig, ax = plt.subplots(figsize=(13, 7))

# Color gradient — each bar gets a progressively darker orange
colors = sns.color_palette("YlOrRd", n_colors=10)[::-1]

# barh() = horizontal bar chart (better for long player names)
bars = ax.barh(
    y     = top_scorers['Batter'][::-1],      # [::-1] reverses so #1 is on top
    width = top_scorers['Total_Runs'][::-1],
    color = colors,
    height = 0.65
)

# Add value labels at the end of each bar
for bar in bars:
    ax.text(
        bar.get_width() + 50,               # Position: just right of bar end
        bar.get_y() + bar.get_height() / 2, # Position: vertically centred
        f"{int(bar.get_width()):,}",        # Text: formatted number e.g. 7,200
        va='center', fontweight='bold', fontsize=11
    )

ax.set_title(' Top 10 Run Scorers — IPL All-Time', fontsize=17, fontweight='bold')
ax.set_xlabel('Total Runs Scored')
sns.despine(left=True, bottom=True)  # Remove top and right borders
plt.tight_layout()
plt.savefig('ipl_charts/insight1_top_scorers.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

## Step 8 — Visualise Insight 2 : Top 10 Wicket Takers

```python
fig, ax = plt.subplots(figsize=(13, 7))

# bar() = vertical bar chart
palette = sns.color_palette('Blues_d', n_colors=10)
bars = ax.bar(
    top_bowlers['Bowler'], top_bowlers['Wickets'],
    color=palette, edgecolor='navy', linewidth=0.7, width=0.6
)

# Add count labels on top of each bar
for bar in bars:
    ax.text(
        bar.get_x() + bar.get_width() / 2,   # Horizontally centred
        bar.get_height() + 2,                 # Just above the bar
        str(int(bar.get_height())),           # The wicket count as text
        ha='center', va='bottom',
        fontweight='bold', fontsize=12, color='navy'
    )

ax.set_title(' Top 10 Wicket-Takers — IPL All-Time', fontsize=17, fontweight='bold')
ax.set_xlabel('Bowler Name')
ax.set_ylabel('Total Wickets')
plt.xticks(rotation=30, ha='right')   # Rotate x-axis labels so they don't overlap
sns.despine()
plt.tight_layout()
plt.savefig('ipl_charts/insight2_top_bowlers.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

##  Step 9 — Visualise Insight 3 : Toss vs Match Result

```python
fig, axes = plt.subplots(1, 2, figsize=(14, 6))  # 1 row, 2 side-by-side charts

result_counts = toss_data['toss_won_match'].value_counts()

#  LEFT chart: Pie chart 
axes[0].pie(
    [result_counts.get(True, 0), result_counts.get(False, 0)],
    labels   = ['Toss Winner\nWon Match', 'Toss Winner\nLost Match'],
    autopct  = '%1.1f%%',         # Show % on each slice e.g. "51.3%"
    colors   = ['#4CAF50', '#EF5350'],
    explode  = (0.04, 0),         # Slightly pop out the first slice
    startangle = 140
)
axes[0].set_title('Overall Toss Impact', fontsize=14, fontweight='bold')

#  RIGHT chart: Grouped bar — bat vs field decision 
decision = (
    toss_data
    .groupby(['toss_decision', 'toss_won_match'])
    .size()
    .unstack(fill_value=0)
)
decision.columns = ['Lost', 'Won']
decision.plot(kind='bar', ax=axes[1], color=['#EF5350','#4CAF50'],
              edgecolor='white', width=0.5, rot=0)

axes[1].set_title('Result by Toss Decision', fontsize=14, fontweight='bold')
axes[1].set_xlabel('Toss Decision (bat / field)')
axes[1].set_ylabel('Matches')

plt.suptitle('🪙 Does Winning the Toss Help?', fontsize=17, fontweight='bold')
plt.tight_layout()
plt.savefig('ipl_charts/insight3_toss_impact.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

###  Step 10 — Visualise Bonus : Season-wise Run Trend

```python
# Aggregate total runs per season using groupby
season_runs = (
    df_clean
    .groupby('season_year')['runs_total']   # Group rows by season
    .sum()                                   # Add up all runs in each season
    .reset_index()                           # Convert back to DataFrame
)
season_runs.columns = ['Season', 'Total_Runs']
season_runs['Season'] = season_runs['Season'].astype(int)
season_runs = season_runs.sort_values('Season')

fig, ax = plt.subplots(figsize=(13, 6))

# Line chart with circles at each data point
ax.plot(season_runs['Season'], season_runs['Total_Runs'],
        color='#1565C0', linewidth=2.8, marker='o',
        markersize=9, markerfacecolor='white', markeredgewidth=2.5)

# Shade the area under the line for visual impact
ax.fill_between(season_runs['Season'], season_runs['Total_Runs'],
                alpha=0.15, color='#1565C0')

ax.set_title(' Season-wise Total Runs — IPL', fontsize=17, fontweight='bold')
ax.set_xlabel('Season')
ax.set_ylabel('Total Runs Scored')
plt.tight_layout()
plt.savefig('ipl_charts/bonus_season_trend.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

### Step 11 — Export Clean Data for Power BI / Tableau

```python
# Export the full clean dataset
df_clean.to_csv('ipl_clean.csv', index=False)
print(" ipl_clean.csv saved")

# Create a batters summary with strike rate calculation
batters_summary = (
    df_clean.groupby('batter')
    .agg(
        total_runs  = ('runs_batter', 'sum'),    # Sum of all runs
        total_balls = ('balls_faced', 'sum'),    # Sum of all balls faced
        matches     = ('match_id', 'nunique')    # Count of unique matches
    )
    .reset_index()
)
# Calculate strike rate: (runs / balls) × 100
batters_summary['strike_rate'] = (
    batters_summary['total_runs'] / batters_summary['total_balls'].replace(0, np.nan) * 100
).round(2)

batters_summary.to_csv('ipl_batters_summary.csv', index=False)
print(" ipl_batters_summary.csv saved")

# Close database connection when done
conn.close()
```

> **Power BI / Tableau**: Import the exported CSVs directly.  
> Detailed steps for both tools are in the sections below.

---

### Python Concepts Used — Quick Reference

| Concept | Code Example | What It Does |
|---------|-------------|--------------|
| Load CSV | `pd.read_csv('file.csv')` | Opens CSV as a DataFrame |
| Copy data | `df.copy()` | Safe copy — never edit original |
| Remove dupes | `df.drop_duplicates()` | Removes identical rows |
| Fill nulls | `df[col].fillna('None')` | Replaces NaN with a value |
| Fix type | `pd.to_numeric(col, errors='coerce')` | Convert text → number |
| GroupBy | `df.groupby('col')['val'].sum()` | Aggregate rows by a column |
| Sort | `.sort_values(ascending=False)` | Largest first |
| Filter rows | `df[df['col'] != 'value']` | Keep rows matching condition |
| SQL query | `pd.read_sql("SELECT ...", conn)` | Run SQL, get DataFrame back |
| Save chart | `plt.savefig('name.png', dpi=150)` | Export chart as image |
| Export CSV | `df.to_csv('name.csv', index=False)` | Save DataFrame as CSV |

---

## Visualizations Preview

| Chart | Type | What It Shows |
|-------|------|---------------|
| Top 10 Run Scorers | Horizontal Bar | All-time batting leaders |
| Top 10 Wicket Takers | Vertical Bar | All-time bowling leaders |
| Toss Impact | Pie + Grouped Bar | Toss winner win percentage |
| Season Trend | Line Chart | Run-scoring growth over seasons |
| Over Pattern | Phase Bar Chart | Powerplay vs Middle vs Death |

---

## Power BI Dashboard — Step-by-Step Guide

### What You Need
- Power BI Desktop (free download from Microsoft)
- The exported CSV files: `ipl_clean.csv`, `ipl_batters_summary.csv`, `ipl_bowlers_summary.csv`

### Steps

**Step 1 — Download Power BI Desktop**
- Go to: https://powerbi.microsoft.com/desktop
- Click "Download free" → Install

**Step 2 — Open Power BI and Load Data**
1. Open Power BI Desktop
2. Click **Home → Get Data → Text/CSV**
3. Select `ipl_batters_summary.csv` → Click Load
4. Repeat for `ipl_bowlers_summary.csv` and `ipl_matches_summary.csv`

**Step 3 — Create Visuals (Report View)**

Visual 1 — Top Run Scorers Bar Chart:
1. Click **Clustered Bar Chart** icon in Visualizations panel
2. Drag `batter` → Y-axis
3. Drag `total_runs` → X-axis
4. In Filters pane: add `batter` filter, select Top N = 10 by `total_runs`
5. Format: Add data labels, change color to orange

Visual 2 — Top Wicket Takers:
1. Click **Clustered Column Chart**
2. Drag `bowler` → X-axis
3. Drag `wickets` → Y-axis
4. Filter: Top 10 by `wickets`
5. Format: Add data labels, change color to blue

Visual 3 — Toss Impact Pie Chart:
1. Load `ipl_matches_summary.csv`
2. Click **Pie Chart** visual
3. Drag `toss_won_match` → Legend
4. Drag count → Values (use Count aggregation)

Visual 4 — Season Trend Line Chart:
1. Click **Line Chart**
2. Drag `season_clean` → X-axis
3. Drag `total_runs` → Y-axis

**Step 4 — Add Slicers (Filters)**
1. Click **Slicer** visual
2. Drag `season_clean` field into it
3. Users can now filter all visuals by season!

**Step 5 — Publish to Power BI Service**
1. Click **Home → Publish**
2. Sign in with free Microsoft account
3. Select "My Workspace"
4. Share the link with anyone

---

## Tableau Dashboard — Step-by-Step Guide

### What You Need
- Tableau Public (free): https://public.tableau.com/
- CSV files: `ipl_clean.csv`, `ipl_batters_summary.csv`

### Steps

**Step 1 — Download and Open Tableau Public**
- Go to: https://public.tableau.com/en-us/s/download
- Install and open

**Step 2 — Connect to Data**
1. On the Start screen → Click **Text File**
2. Select `ipl_batters_summary.csv` → Open
3. You'll see the Data Source page — verify columns look correct

**Step 3 — Create Sheet 1 — Top Run Scorers**
1. Click **Sheet 1** tab at the bottom
2. Drag `Batter` → Rows shelf
3. Drag `Total Runs` → Columns shelf
4. Click the **Sort Descending** button (toolbar)
5. Right-click on Batter pill → Filter → Top → Top 10 by Total Runs
6. Change chart type to **Bar** in the Marks card
7. Drag `Total Runs` → Label in Marks card to show values

**Step 4 — Create Sheet 2 — Top Wicket Takers**
1. Click the **+** button to add a new sheet
2. Connect to `ipl_bowlers_summary.csv` (Data → New Data Source)
3. Drag `Bowler` → Rows
4. Drag `Wickets` → Columns
5. Sort descending, Filter Top 10
6. Change color: Click Color in Marks → Edit Colors → Blue

**Step 5 — Create Sheet 3 — Season Trend**
1. Connect to `ipl_clean.csv`
2. New sheet: Drag `Season Clean` → Columns
3. Drag `Runs Total` → Rows (it will auto SUM)
4. Change Mark type to **Line**
5. Add `Runs Total` to Label

**Step 6 — Create Dashboard**
1. Click **Dashboard → New Dashboard**
2. Set size: Automatic or 1200 × 800
3. Drag Sheet 1 to the top-left quadrant
4. Drag Sheet 2 to the top-right
5. Drag Sheet 3 to the bottom
6. Click **Dashboard → Actions → Add Action → Filter**
   - This links all charts — clicking one filters the others!

**Step 7 — Publish to Tableau Public**
1. File → Save to Tableau Public
2. Sign in (free account)
3. Give a name: "IPL Analysis Dashboard"
4. Your dashboard gets a public URL to share!

---

## Key Learnings

- Real IPL data has 64 columns — most analysis needs only 8-10 key columns
- Data cleaning (nulls, types, duplicates) took more work than the analysis itself
- The toss has minimal impact — this is a genuinely surprising statistical finding
- Death overs consistently produce the highest run rates across all seasons

---

*⭐ If this project helped you, please give it a star on GitHub!*
