# League of Legends Player Segmentation (Unsupervised Learning)

Unsupervised analysis of League of Legends (LoL) player data, covering preprocessing, exploratory data analysis (EDA), and clustering with K-Means, MiniBatch K-Means, and DBSCAN. The goal is to group players by skill, engagement, and recent performance, which can inform churn analysis.

## Dataset

- **Source:** [Mendeley Data – LoL gamer churn dataset](https://data.mendeley.com/datasets/74drmd922j/1)
- **Description:** Collected to analyze and predict gamer churn in League of Legends, Season 13. Data comes from the LoL **Korea** server via Riot Games' API and web scraping (Selenium, Pandas).
- **Size:** 295,486 raw cases (294,530 after cleaning).
- **Note:** The notebook loads the file `LOL RAW DATA.xlsx` from a GitHub copy of the dataset (`RihalMahmood/CSSE445-Assignments`). You can also download it from Mendeley and point `pd.read_excel` at the local file.

### Main columns

| Column | Meaning |
|---|---|
| `TIER`, `LP`, `LEVEL` | Current rank, League Points, account level |
| `AVSCORE` | Average in-game score |
| `LASTWINLOSE` | Recent record, e.g. `8W12L40%` |
| `NO TEAM PARTICIPANTS`, `NO TEAM GAMES`, `TEAM WIN`, `TEAM LOSE` | Team-game activity and results |
| `WIN DUMMY`, `LOSING STREAK` | Win flag (O/X) and losing streak |
| `NO CHAMPIONS PLAYED` | Number of champions played |
| `T_POSITION`, `P_NO POSITION` | Preferred position and positional info |
| `START SEASON`, `SEASON TIER`, `LAST TIER` | Season history |
| `TOTAL HOUR / MIN / DAY / MATCH` | Total playtime and matches |

## Project Structure

```
.
├── Unsupervised.ipynb   # Full pipeline: preprocessing → EDA → clustering
└── README.md
```

## Workflow

### 1. Data cleaning
- Dropped 478 rows with missing values in the season columns (`START SEASON`, `SEASON TIER`, `LAST SEASON`, `LAST TIER`), which were missing for the same rows.
- Dropped columns that are identifiers, redundant, or constant: `CASE`, `WINLOSEPER`, `LASTLOGIN`, `TEAM WINRATE`, `LAST SEASON`, `OBS DATE`, `DATE`.
- Translated Korean position names in `T_POSITION` to English (Jungle, Support, Top, Carry, Mid).

### 2. Feature engineering and encoding
- Parsed `LASTWINLOSE` into `LAST_WINS`, `LAST_LOSSES`, `LAST_WIN_PCT`.
- **Binary encoding:** `WIN DUMMY` (O → 1, X → 0).
- **Ordinal encoding:** `TIER`, `SEASON TIER`, `LAST TIER`, ordered by the LoL rank hierarchy (higher rank → higher value).
- **One-hot encoding:** `T_POSITION`, `START SEASON`, `P_NO POSITION` (first category dropped).

### 3. Outlier handling
- Detected outliers with both the IQR rule and Z-scores.
- Capped (winsorized) the 16 numeric features at the IQR bounds, leaving 0 IQR outliers.
- Dropped the rows left with NaNs after encoding, giving a final shape of **294,530 × 41**.

### 4. Scaling
- Applied `StandardScaler` to all features (mean ≈ 0, std ≈ 1).

### 5. Exploratory data analysis
Summary statistics, histograms, boxplots, a correlation heatmap, and relationship plots (e.g. LP vs. AVSCORE by tier, AVSCORE by position, TOTAL DAY by WIN_DUMMY, LAST_WIN_PCT vs. TOTAL MATCH).

Key observations:
- Engagement features such as `TOTAL HOUR` and `TOTAL MATCH` are skewed, indicating widely varying player engagement.
- Highly correlated features (e.g. `TOTAL HOUR` vs. `TOTAL MIN`) suggest redundancy, which motivates PCA.
- Players are concentrated in certain ranks.
- `AVSCORE` varies by position.
- Losing streaks and recent win percentage are plausible churn-risk signals.

### 6. Clustering
- **PCA:** reduced to **27 components**, retaining **96%** of the variance.
- **Elbow method** (K-Means, k = 1–10, 30,000-row sample): the curve flattens around **k = 4**.
- **K-Means** (k = 4) on the full PCA-reduced data.
- **MiniBatch K-Means** (k = 4, batch size 100) as a faster alternative.
- **DBSCAN** (`eps=0.5`, `min_samples=5`) fitted on a 10,000-row subset to save memory.
- **Visualization:** a 5,000-point sample projected to 2D with PCA, with the three algorithms shown side by side.

## Limitations

- **DBSCAN is approximate.** It was fitted on only 10,000 rows, and all remaining rows were labelled as noise (`-1`), so its labels over the full dataset are not a true clustering.
- **The 2D plot is a rough view.** It explains only about 24% of the variance of the 27-component data, so cluster separation may look different from how it really is.
- **No quantitative validation yet.** `silhouette_score` is imported but not used, and the clusters have not been profiled or interpreted. Adding silhouette scores and per-cluster feature summaries would strengthen the results.
- **Skewed capping.** Capping with the IQR rule was applied to heavily skewed features, which compresses genuinely high-engagement players.
- The dataset covers **Korean-server, Season 13** players only, so results may not generalize to other regions or seasons.

## Getting Started

### Requirements
- Python 3.8+
- `numpy`, `pandas`, `matplotlib`, `seaborn`, `scikit-learn`, `requests`, `openpyxl`

```bash
pip install numpy pandas matplotlib seaborn scikit-learn requests openpyxl jupyter
```

### Run
1. Clone this repository and open `Unsupervised.ipynb` in Jupyter or Google Colab.
2. (Optional) Download the dataset from [Mendeley](https://data.mendeley.com/datasets/74drmd922j/1) and update the file path in the data-loading cell.
3. Run all cells in order. The notebook samples the data in several places to stay within Colab's memory limits.

## Citation

> *League of Legends gamer churn dataset*, Mendeley Data, V1. https://data.mendeley.com/datasets/74drmd922j/1

Jongho Kim, Kihan Kim, Hansol Kim, Shanshan Li (27 January 2025)

## Course

CSE445 (A group of four(4) members)
