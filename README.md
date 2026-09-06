# Logistics Data Cleaning & Preparation using Microsoft SQL Server

## Project Overview

This project focuses on cleaning and preparing a logistics dataset using Microsoft SQL Server (MSSQL).

The dataset contains logistics and delivery information such as order IDs, delivery status, delivery dates, shipping costs, vehicle types, payment methods, fuel surcharges, customer ratings, and origin and destination cities.

The cleaning process was performed directly in SQL Server to identify data quality issues, investigate missing values, handle duplicates, standardize inconsistent data, and produce a clean dataset ready for further analysis.

## Objectives

The main objectives of this project are:

- Inspect the structure and quality of the logistics dataset.
- Identify missing values across all columns.
- Investigate the relationship between missing values and other variables.
- Handle missing values based on business context.
- Identify and resolve duplicate `OrderID` records.
- Standardize inconsistent city names.
- Create a cleaned dataset for further analysis.

## Project Files

The repository contains the following files:

| File | Description |
|---|---|
| `ShippingLogistics.csv` | Original/raw logistics dataset |
| `ShippingLogisticsCleaning.csv` | Cleaned and prepared logistics dataset |
| `Logistics Data Cleaning.sql` | T-SQL script containing the complete cleaning and transformation process |

## Database Structure

The raw CSV dataset was imported into a Microsoft SQL Server database named:

```text
LogistikDB
```

The cleaning workflow uses three main tables:

```text
ShippingLogistics
        │
        ▼
ShippingLogistics_Cleaning
        │
        ▼
ShippingLogistics_Cleaned
```

### Table Description

| Table | Purpose |
|---|---|
| `dbo.ShippingLogistics` | Original dataset imported from CSV |
| `dbo.ShippingLogistics_Cleaning` | Working table used for cleaning and transformation |
| `dbo.ShippingLogistics_Cleaned` | Final cleaned dataset |

The original table was preserved so that the cleaning process could be performed without modifying the raw data.

## Data Cleaning Workflow

```text
Raw CSV Data
     │
     ▼
SQL Server Import
     │
     ▼
Data Inspection
     │
     ▼
Missing Value Analysis
     │
     ▼
Missing Value Treatment
     │
     ▼
Duplicate Analysis
     │
     ▼
Data Standardization
     │
     ▼
Final Clean Dataset
```

## 1. Data Inspection

The original dataset was first inspected using SQL queries to understand the number of records and identify potential data quality issues.

A working copy of the original table was then created:

```sql
SELECT *
INTO dbo.ShippingLogistics_Cleaning
FROM dbo.ShippingLogistics;
```

This approach ensures that the raw dataset remains unchanged throughout the cleaning process.

## 2. Missing Value Analysis

Missing values were analyzed across all columns using dynamic T-SQL and `INFORMATION_SCHEMA.COLUMNS`.

This allowed the missing-value count to be generated automatically for every column instead of manually checking each column.

The analysis identified missing values in several fields, including:

- `DelayReason`
- `DeliveryDate`
- `ActualDeliveryDays`
- `VehicleType`
- `PaymentMethod`
- `FuelSurchargeIDR`
- `CustomerRating`

Each missing-value pattern was investigated before deciding on an appropriate treatment.

## 3. Missing Value Treatment

### `DelayReason`

Missing `DelayReason` values were investigated against `DeliveryStatus`.

The analysis showed that the missing values were associated with successfully delivered orders.

Therefore, the missing values were interpreted as orders without a recorded delay reason.

```text
NULL → No Delay
```

This provides a meaningful categorical value while preserving the business context.

### `DeliveryDate` & `ActualDeliveryDays`

Missing values in `DeliveryDate` and `ActualDeliveryDays` were investigated together with `DeliveryStatus`.

For shipments that were not successfully delivered, such as lost shipments, delivery information was unavailable.

Therefore, these values were retained as `NULL` rather than artificially imputing delivery information.

### `VehicleType` & `PaymentMethod`

Missing values in `VehicleType`, `PaymentMethod`, and `FuelSurchargeIDR` were found to occur in the same records.

Since `VehicleType` and `PaymentMethod` are categorical variables and their original values could not be determined reliably, they were replaced with:

```text
NULL → Unknown
```

### `FuelSurchargeIDR`

`FuelSurchargeIDR` is a numerical variable.

Missing values were handled using median imputation with `PERCENTILE_CONT()`.

The median was selected because it is less affected by extreme values than the mean and is therefore more suitable for potentially skewed monetary data.

### `CustomerRating`

Missing `CustomerRating` values were intentionally retained as `NULL`.

Customer ratings represent optional customer feedback. Therefore, the absence of a rating does not necessarily represent a data quality issue.

## 4. Duplicate Order Analysis

Duplicate records were identified using `OrderID`.

The duplicate records were then examined to determine whether they contained identical information or conflicting values.

Most duplicate records contained identical information across the columns.

Two duplicate cases contained different `ShippingCostIDR` values.

A cleaning rule was applied to retain one record per `OrderID`, with the record having the lower `ShippingCostIDR` being retained.

```sql
ROW_NUMBER() OVER (
    PARTITION BY OrderID
    ORDER BY ShippingCostIDR ASC
)
```

After the deletion process, the dataset was checked again to ensure that duplicate `OrderID` records no longer remained.

> Note: In a real production environment, conflicting records should ideally be validated against the original transaction source before deciding which value should be retained.

## 5. Data Standardization

The `OriginCity` and `DestinationCity` columns contained inconsistent capitalization and unnecessary spaces.

The values were standardized using:

```sql
UPDATE dbo.ShippingLogistics_Cleaning
SET OriginCity = TRIM(UPPER(OriginCity)),
    DestinationCity = TRIM(UPPER(DestinationCity));
```

For example:

```text
 Jakarta  → JAKARTA
```

This improves consistency when performing grouping, filtering, and location-based analysis.

## Key Data Quality Decisions

| Data Issue | Treatment | Reason |
|---|---|---|
| `DelayReason` | `No Delay` | Missing values associated with delivered orders |
| `DeliveryDate` | Keep `NULL` | Delivery information unavailable |
| `ActualDeliveryDays` | Keep `NULL` | Cannot calculate without completed delivery |
| `VehicleType` | `Unknown` | Original categorical value unavailable |
| `PaymentMethod` | `Unknown` | Original categorical value unavailable |
| `FuelSurchargeIDR` | Median imputation | Numerical variable |
| `CustomerRating` | Keep `NULL` | Customer feedback is optional |
| Duplicate `OrderID` | Remove duplicate records | Prevent duplicate orders from affecting analysis |
| City names | `TRIM()` + `UPPER()` | Standardize formatting |

## Final Dataset

After the cleaning and transformation process, the final dataset was stored in:

```text
dbo.ShippingLogistics_Cleaned
```

The cleaned data can then be exported as:

```text
ShippingLogisticsCleaning.csv
```

The final dataset is ready for further Exploratory Data Analysis (EDA), statistical analysis, and dashboard development.

## Potential Analysis

The cleaned logistics dataset can be used to analyze:

- Delivery performance
- Delivery delays
- Average delivery time
- Shipping costs
- Fuel surcharge patterns
- Courier performance
- Vehicle utilization
- Origin and destination performance
- Customer ratings
- Delivery status distribution

## Tools & Technologies

- Microsoft SQL Server
- T-SQL
- Dynamic SQL
- `INFORMATION_SCHEMA`
- CTE
- Window Functions
- `ROW_NUMBER()`
- `PERCENTILE_CONT()`
- Data Cleaning
- Data Preparation

## Repository Structure

```text
Logistics-Data-Cleaning/
│
├── ShippingLogistics.csv
├── ShippingLogisticsCleaning.csv
├── Logistics Data Cleaning.sql
└── README.md
```

## Outcome

This project demonstrates how SQL can be used beyond basic data querying, including systematic data quality assessment, missing-value treatment, duplicate handling, data standardization, and preparation of a dataset for downstream analytics.

The resulting `ShippingLogistics_Cleaned` dataset provides a more consistent and analysis-ready foundation for understanding logistics and delivery performance.
