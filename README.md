# Airbnb Listing Availability Prediction — ETL Pipeline

This project builds a complete ETL and feature engineering pipeline for predicting Airbnb listing demand using historical listing, review, and calendar data. The main goal is to transform raw relational database tables into a clean, versioned, machine-learning-ready dataset. 

---

## 1. Setup Environment

In the first step, the required Python libraries such as `pandas`, `numpy`, `sqlalchemy`, `psycopg2-binary`, and `pyarrow` were installed and imported. Environment variables were used to safely store database credentials instead of hardcoding them inside the notebook. 

---

## 2. Database Connection and Query Utilities

A PostgreSQL connection was established using SQLAlchemy by constructing a database URL from environment variables.

---

## 3. Dataset Audit and Time Window Definition

Based on the assignment requirements, a 90-day historical feature window and a 30-day future label window were defined. The cutoff date was selected so that enough historical and future observations existed for every listing. 

---

## 4. PII Audit and Privacy Review

A privacy audit was performed to identify columns that contain personally identifiable information (PII) or sensitive identifiers. Columns such as `host_id`, `host_pseudo_id`, `review_id`, `reviewer_id`, `reviewer_pseudo_id`, and `license` were documented and excluded from future model inputs.

---

## 5. Loading Static Reference Tables

The tables `listing`, `host`, and `neighbourhood` were loaded directly into pandas DataFrames. Only the columns required for feature engineering were selected in order to reduce memory usage and improve efficiency. 

---

## 6. Cleaning Static Fields

Boolean columns were converted to proper boolean types, numeric fields were converted to numeric formats, and missing values were standardized. The text field `bathrooms_text` was transformed into a new numeric feature called `bathrooms`. For example, values such as `"1 bath"` became `1.0`, `"1.5 baths"` became `1.5`, and `"Half-bath"` became `0.5`. This conversion ensures that bathroom information can be used directly by machine learning algorithms.

---

## 7. Building Static Listing Features

The listing table was enriched by joining it with host and neighbourhood information. Additional host-level information was created by calculating the number of listings owned by each host (`host_listing_count`). 

---

## 8. Building Review Features

Review information was aggregated directly in SQL to avoid loading millions of review records into memory. For each listing, several historical review statistics were calculated, including total number of reviews, number of unique reviewers, average review length, maximum review length, and days since the last review. Only reviews that occurred before the cutoff date were included. These features provide useful signals about listing popularity and guest engagement without exposing any raw review text.

---

## 9. Building Calendar History Features

Historical availability information was aggregated from the calendar table using both 90-day and 30-day lookback windows. Features such as available days, availability rates, and average minimum and maximum stay requirements were calculated. For example, a listing with very low availability may indicate high booking demand. All calculations were restricted to dates before the cutoff date to avoid future leakage. These features capture recent and long-term availability behavior of each listing.

---

## 10. Building the Target Label

The target variable was created using future calendar information. For each listing, availability was measured during the 30-day period immediately after the cutoff date. Listings with a future availability rate less than or equal to 30% were labeled as high demand (`high_demand_proxy = 1`), while the rest were labeled as low demand (`high_demand_proxy = 0`). This target is a proxy for demand rather than a direct booking count, but it provides a reasonable supervised learning objective using the available data.

---

## 11. Joining Feature Groups

The static features, review features, calendar features, and target label were merged into a single machine-learning dataset. An inner join with the label table ensured that only listings with valid target values were included. Additional metadata columns such as `cutoff_date` and `dataset_version` were added for auditing purposes. Missing review counts were filled with zero, and listings with no reviews received appropriate default values. The result was a single row per listing containing all engineered features and the target label.

---

## 12. Feature Quality Review

Before saving the dataset, feature quality checks were performed. Missing value rates were calculated for every column, and columns with more than 95% missing values or constant values across all rows were identified. In this dataset, no columns exceeded the missing-value threshold and no constant features were found, meaning all engineered features provided potentially useful information. This confirmed that the feature engineering process produced a well-populated dataset.

---

## 13. Dataset Validation

A comprehensive validation step ensured the dataset was safe for machine learning. Checks confirmed that there were no duplicate listing-cutoff combinations, no missing target values, no forbidden PII columns, and no future leakage features in the model input set. The target variable contained only valid binary values (0 and 1). The validation report demonstrated that the dataset met all project requirements and was suitable for supervised learning.

---
## 14. Dataset Exploration and Quality Analysis

Several summary reports were created to understand the final dataset. Most engineered features had no missing values, while a few original fields such as `listing_price` and `beds` contained some missing data. The target distribution showed that about 76% of listings were labeled as high demand and 24% as low demand. Calendar checks confirmed that all listings had a complete 30-day future observation period, making the target labels consistent and reliable.

---
## 15. Saving Versioned Outputs

The final dataset was saved in both CSV and Parquet formats along with metadata, validation reports, and a PII audit report. Versioning makes the dataset reproducible and allows future notebooks to use the same processed data without querying the raw database again.

Generated files:

* `listing_availability_features_v1_student.csv`
* `listing_availability_features_v1_student.parquet`
* `listing_availability_features_v1_student_metadata.json`
* `listing_availability_features_v1_student_validation_report.json`
* `pii_audit_v1_student.csv`
---
## Conclusion

This ETL pipeline transformed raw Airbnb data into a clean machine-learning dataset. Historical features and future labels were separated to avoid data leakage, sensitive information was removed, and validation checks ensured data quality. The final dataset contains one row per listing and is ready for model training, experiment tracking, and deployment.

