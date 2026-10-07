# Intelligent Warehouse Slotting Optimization

## Project Overview

Warehouse slotting plays an important role in operational efficiency by
determining where inventory is stored. Parts with higher demand may not
always be positioned in locations that best support their usage
patterns, which can contribute to unnecessary material movement and
inefficient use of warehouse space.

The objective of this project is to analyze historical warehouse
inventory, usage, and storage-location data to identify patterns that
can support improved slotting decisions. Exploratory data analysis
(EDA), feature engineering, and a baseline machine learning model were
used to examine relationships between SKU demand, inventory levels, and
current storage positions.

## Data

The analysis used a proprietary warehouse operational dataset containing
4,994 part records. Key fields included:

-   Part number and description
-   Average Daily Usage (ADU)
-   Current inventory level
-   Warehouse storage location

The raw dataset is not included in this public repository because it
contains proprietary operational information. Row-level part-number and
exact storage-location outputs have also been excluded from the public
notebook.

A significant portion of the records had missing storage-location
information. These records were retained where appropriate for overall
analysis but were not assigned to a warehouse area when their actual
home location could not be determined.

## Data Cleaning and Feature Engineering

The analysis included:

-   Review of missing values and duplicate records
-   Removal of unnecessary source columns
-   Preservation of missing storage and inventory values where
    assumptions could not be justified
-   Log transformation of Average Daily Usage and Current Inventory to
    reduce the influence of highly skewed values
-   Parsing of structured warehouse locations into warehouse area,
    aisle, vertical level, and horizontal position
-   Validation and cleaning of vertical storage levels
-   Creation of usage and location-based features to support exploratory
    analysis

Extremely high ADU values were retained because some materials may use
different units of measure and reliable conversion information was
unavailable.

## Exploratory Data Analysis

EDA examined the distributions of SKU usage and inventory, the
relationship between usage and inventory, and how high-usage parts were
distributed across warehouse storage positions.

Detailed location-based analysis focused on the SM101 small-parts
storage area because it contained sufficient records with consistently
structured locations.

### Key Findings

-   Average Daily Usage and Current Inventory were highly right-skewed.
    Log transformations made the distributions more suitable for
    analysis.
-   Log-transformed inventory and usage showed a moderate positive
    correlation of approximately 0.49.
-   High-usage SKUs were not evenly distributed across SM101 storage
    positions.
-   Aisle 21 showed the strongest potential slotting opportunity. Only
    approximately 2--3% of parts at the lowest vertical levels A and B
    were classified as high usage, while approximately 76--86% of parts
    at Levels E--H were classified as high usage.
-   SKU-level investigation confirmed that multiple high-usage parts
    were stored at upper vertical levels, identifying potential
    candidates for operational slotting review.

These findings identify potential optimization opportunities rather than
confirmed mis-slotting. Physical constraints such as part dimensions,
weight, handling requirements, safety considerations, and reserved
storage requirements were not available and would need to be considered
before recommending actual relocations.

## Baseline Model

A baseline Linear Regression model was developed to predict
log-transformed Average Daily Usage using:

-   Log-transformed Current Inventory
-   Aisle
-   Vertical Storage Level

The dataset was divided into 80% training data and 20% test data.
Missing inventory values were handled using median imputation, while
categorical storage variables were one-hot encoded through a
preprocessing pipeline.

### Model Results

The baseline model achieved:

  Metric     Result
  -------- --------
  R²          0.276
  MAE         0.587
  RMSE        0.796

The R² indicates that the available inventory and storage-location
characteristics explained approximately 27.6% of the variation in
log-transformed SKU usage in the test data. The Actual vs. Predicted
analysis also showed that the baseline model tended to underpredict some
of the highest-usage SKUs.

The baseline therefore provides an initial benchmark but also
demonstrates that inventory and storage position alone are insufficient
to fully explain SKU usage.

## Conclusions and Next Steps

The project identified meaningful relationships between warehouse usage,
inventory, and current storage placement. In particular, the
concentration of high-usage SKUs at upper storage levels in parts of
SM101 suggests opportunities for further slotting review.

Future development could incorporate additional operational variables
such as:

-   Pick frequency
-   Part dimensions and weight
-   Travel distance
-   Handling and equipment requirements
-   Complete home-location information
-   Production demand
-   Storage restrictions and reserved locations

These additional features could support a more comprehensive
machine-learning-based slotting recommendation system.

## Repository Contents

-   `Intelligent_Warehouse_Slotting_Capstone_project_PUBLIC_CLEAN.ipynb`
    --- Jupyter notebook containing the analysis, EDA, feature
    engineering, baseline model, and results.
-   `README.md` --- Project overview, methodology, findings, and model
    results.

## Data Privacy

The source dataset contains proprietary warehouse operational
information and is therefore not included in this repository. Public
project materials contain aggregate analytical results rather than raw
operational records.
