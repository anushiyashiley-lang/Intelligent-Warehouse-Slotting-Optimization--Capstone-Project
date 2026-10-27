# Intelligent Warehouse Slotting Optimization

**UC Berkeley Engineering — Machine Learning & Artificial Intelligence Capstone - Final Report**

## Executive Summary

Warehouses need to store frequently used parts in locations that support efficient picking and replenishment. When high-demand parts are stored in less accessible positions, workers may spend more time traveling or using handling equipment. This project examined **4,994 warehouse part records** to understand the relationship between demand, inventory, and storage location and to identify areas worth reviewing for improved slotting.

**Main findings:** Within the SM101 small-parts area, Aisle 21 showed a striking difference between lower and upper storage levels: approximately **2–3%** of records at levels A–B were classified as high usage, compared with **76–86%** at levels E–H. This is a *screening signal*, not proof that any part is incorrectly stored. In a separate demand-prediction exercise, **Random Forest Regression** performed best among the tested models, with a held-out test **R² of 0.412** and **RMSE of 0.717** on the transformed usage scale.

**Recommendation:** Prioritize Aisle 21 for an operational slotting review, checking demand, physical fit, equipment access, safety, and travel implications before moving inventory. The models developed here predict usage; they **do not directly prescribe optimal storage locations**.

## 1. Problem Statement and Business Goal

Warehouse slotting determines where inventory is stored. Storage assignments that do not reflect demand can create avoidable handling effort, but relocating stock without considering physical restrictions can introduce other costs or safety concerns.

The goals of this project were to:

1. Examine how part usage and inventory relate to current warehouse locations.
2. Identify storage areas with unusual concentrations of high-usage parts at upper levels.
3. Compare machine-learning models for predicting usage from available inventory and location attributes.
4. Translate the evidence into practical recommendations for a future slotting review.

**Potential benefit:** Better prioritization of slotting investigations and, after operational validation, opportunities to reduce picking and handling effort. No measured time or cost savings are claimed in this study.

## 2. Research Questions and Hypotheses

- **RQ1:** How are usage and current inventory related? **Hypothesis:** Higher-usage parts tend to have higher inventory levels.
- **RQ2:** Are high-usage parts distributed evenly across vertical storage levels in SM101? **Hypothesis:** Some aisles contain a disproportionate share of high-usage parts at upper levels.
- **RQ3:** Can nonlinear regression improve usage prediction over a simple linear model? **Hypothesis:** Random Forest may capture relationships not represented by Linear Regression.

These are exploratory questions; the observed associations do not establish causation.

## 3. Data Acquisition and Scope

The analysis used an internal operational extract with **4,994 part records**, including part number/description, **Average Daily Usage (ADU)**, current inventory, and warehouse storage-location information. The data were drawn from a proprietary operational source; no additional external data source was incorporated in this version.

**ADU** describes average daily consumption or demand for a part. The detailed aisle/level investigation focused on **SM101**, a small-parts storage area with sufficiently consistent location codes. This was analyzed separately from bulk-storage areas, where part quantities and storage characteristics are not directly comparable.

The raw dataset is **not shared** because it contains confidential operational information. Missing home locations limit the geographic coverage of the slotting analysis.

## 4. Data Cleaning and Preparation

The notebook documents the following preparation steps:

- Inspected missing values and duplicate records and removed unnecessary source fields.
- Preserved missing storage locations rather than inventing locations; excluded records with unknown aisle or level from location-specific comparisons.
- Examined extreme usage and inventory values. Very high ADU values were retained because units of measure may differ and reliable conversion information was unavailable.
- Applied **log(1 + value)** transformations to highly right-skewed ADU and inventory distributions.
- Parsed structured location codes into warehouse area, aisle, vertical level, and horizontal position; cleaned vertical-level labels.
- For predictive modeling, used **median imputation** for missing numeric inventory values and **one-hot encoding** for categorical aisle and level fields within a preprocessing pipeline.
- Used an **80% training / 20% held-out test** split. Cross-validation and hyperparameter search were conducted on the training portion.

Missing values and uncertain units remain limitations rather than issues assumed to be fully resolved.

## 5. Exploratory Analysis and Key Findings

### Demand and inventory

Both ADU and current inventory were strongly right-skewed. Their log-transformed values had a **moderate positive correlation of approximately 0.49**. This supports the expectation that higher-usage parts often carry higher inventory, although the relationship is far from perfect.

### SM101 vertical slotting

The analysis classified a part as **high usage** when its ADU exceeded the **SM101-specific 80th-percentile threshold (26.2 ADU)**. The percentage displayed for each aisle/level is the number of high-usage records **divided by all records in that same aisle/level**.

**Aisle 21 — high-usage share by level:**

| Level | High-usage share |
|---|---:|
| A (lowest) | 2.63% |
| B | 2.27% |
| E | 86.36% |
| F | 81.82% |
| G | 76.00% |
| H (highest) | 81.82% |

There were **202 records at levels A–B** and **91 records at E–H** in Aisle 21. The contrasting percentages are therefore not explained simply by having more records at upper levels. The notebook includes the corresponding heatmap and count table, along with additional exploratory charts.

**Business interpretation:** Aisle 21 is a strong candidate for review because some high-usage parts appear at higher, potentially less accessible levels. However, the dataset does not establish whether these locations are inappropriate: dimensions, weight, lift-equipment needs, safety rules, reserved slots, and actual travel or handling times were not available.

## 6. Modeling Approach and Predicted Outcome

This is a **supervised regression** problem: the models predict **log(1 + ADU)** using log-transformed current inventory, aisle, and vertical storage level. A prediction of demand is an intermediate input to a future slotting decision system—not a prediction of the best storage location itself.

The notebook compares:

- **Training-mean benchmark:** predicts the same average transformed usage for every record; establishes a minimum comparison point.
- **Linear Regression:** an interpretable starting model.
- **Ridge Regression:** a regularized linear alternative.
- **Random Forest Regression:** a nonlinear ensemble that can capture more complex relationships.

The modeling section includes **five-fold cross-validation** and **GridSearchCV** for hyperparameter tuning, with the held-out test set reserved for final performance assessment.

## 7. Model Evaluation and Results

Three metrics were used:

- **R²:** how much of the variation in transformed demand is explained by predictions; higher is better.
- **MAE:** average absolute prediction error; lower is better.
- **RMSE:** gives greater weight to larger prediction errors; lower is better and was used for cross-validation model selection.

| Model | Test R² | Test MAE | Test RMSE |
|---|---:|---:|---:|
| Training-mean benchmark | −0.000 | 0.748 | 0.936 |
| Linear Regression | 0.276 | 0.587 | 0.796 |
| Ridge Regression | 0.275 | 0.588 | 0.797 |
| **Random Forest Regression** | **0.412** | **0.469** | **0.717** |

*MAE and RMSE are expressed on the **log(1 + ADU)** scale, not in original daily-use units.*

**Random Forest** achieved the lowest reported five-fold cross-validation RMSE (**0.710**) and the best held-out test performance among the evaluated models. Its test RMSE of **0.717** was approximately **9.9% lower** than the Linear Regression baseline's **0.796**. Its test R² of **0.412** indicates that it explains approximately **41.2%** of the variation in transformed usage in the test sample. This is a meaningful improvement, but substantial variation remains unexplained.

The closeness of the Random Forest cross-validation and test RMSE values is encouraging, although it does not guarantee performance on future warehouse data. These results support choosing Random Forest **for the tested usage-prediction task**, not for automatic slot assignment.

## 8. Conclusions and Actionable Recommendations

**What the data support:** Usage and inventory are moderately associated; high-usage SKUs are unevenly distributed across SM101 levels; Aisle 21 warrants further investigation; and Random Forest predicts transformed ADU more accurately than the other tested models on this dataset.

**Recommended next steps:**

1. **Review Aisle 21 first.** Validate high-usage parts currently stored at upper levels against accessible lower-level alternatives.
2. **Apply physical and safety constraints.** Check dimensions, weight, equipment, location capacity, reserved storage, and handling requirements before any relocation.
3. **Add operational measures.** Collect pick frequency, travel distance/time, labor effort, and relocation cost to estimate potential benefits.
4. **Pilot and measure.** Test a small number of approved moves and compare picking/handling performance before and after implementation.
5. **Improve demand modeling.** Add more relevant predictors and evaluate on later time periods before operational deployment.

**Bottom line:** This capstone creates a practical, evidence-based starting point for prioritizing slotting reviews. It does not establish measured efficiency savings or a fully automated slotting recommendation engine.

## 9. Limitations and Future Development

- Incomplete storage-location data restricts coverage.
- Different material units of measure may influence extreme ADU values.
- Demand predictions rely on a limited set of inventory/location features.
- Current location may reflect existing operational decisions; correlations should not be treated as causal effects.
- Physical fit, travel, equipment, safety, and cost data are needed to turn insights into feasible slotting recommendations.
- Model evaluation uses a held-out sample; future time-based validation would strengthen confidence in operational use.

## 10. Technical Notebook and Repository Contents

- **[Capstone analysis notebook](Intelligent_Warehouse_Slotting_Capstone.ipynb)** — data preparation, exploratory charts, location heatmap, baseline and advanced regression models, cross-validation, hyperparameter tuning, and findings.
- **`README.md`** — nontechnical final report and summary of the work.

The confidential source dataset is intentionally omitted.

## 11. Data Confidentiality and Reproducibility

This repository is intended to present **aggregate findings only**. Do not publish raw operational extracts, part numbers, part descriptions, precise storage-location records, private file paths, or notebook outputs exposing confidential data. Before making the repository public, inspect both notebook code and saved cell outputs, and confirm that all displayed content is authorized for release.

Because the raw dataset cannot be distributed, external readers may not be able to rerun every analysis. The notebook documents the methods and reports the results from the authorized internal execution.

