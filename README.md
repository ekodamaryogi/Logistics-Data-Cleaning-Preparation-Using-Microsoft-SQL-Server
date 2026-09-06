# Logistics-Data-Cleaning-Preparation-Using-Microsoft-SQL-Server

Project Overview
This project focuses on cleaning and preparing a logistics dataset using Microsoft SQL Server.
The data cleaning process was performed directly in Microsoft SQL Server (MSSQL) to ensure that the dataset was consistent, complete, and ready for further exploratory data analysis and dashboard development.
Objectives
The main objectives of this project were to:
•	Identify missing values across all columns.
•	Investigate the causes and patterns of missing data.
•	Handle missing values based on their business context.
•	Identify and resolve duplicate OrderID records.
•	Standardize inconsistent city names.
•	Create a clean dataset for subsequent analysis.
Data Preparation Workflow
The cleaning process followed these main stages:
1.	Data inspection
2.	Dataset duplication for cleaning
3.	Missing value analysis
4.	Missing value treatment
5.	Duplicate record analysis
6.	Data standardization
7.	Creation of the final cleaned dataset
1. Data Inspection
The original dataset was stored in Microsoft SQL Server under the LogistikDB database.
The original table was inspected to understand the structure and identify potential data quality issues before performing any transformation.
A separate table, ShippingLogistics_Cleaning, was then created as a working copy to ensure that the original dataset remained unchanged.
2. Missing Value Analysis
Missing values were analyzed across all columns using a dynamic T-SQL query based on INFORMATION_SCHEMA.COLUMNS.
This approach allowed missing values to be identified systematically without manually writing a separate query for every column.
Key findings
Several missing-value patterns were identified:
•	DelayReason
•	DeliveryDate
•	ActualDeliveryDays
•	VehicleType
•	PaymentMethod
•	FuelSurchargeIDR
•	CustomerRating
The missing values were not immediately replaced with generic values. Each column was investigated based on its relationship with other variables and its business meaning.
3. Handling Missing Values
DelayReason
A significant number of DelayReason values were missing.
Further analysis showed that these missing values were associated with orders having a Delivered status. Since successfully delivered orders do not necessarily have a delay reason, the missing values were interpreted as indicating that no delivery delay occurred.
Therefore:
**NULL → **No Delay
This transformation makes the column easier to analyze while preserving its business meaning.
DeliveryDate and ActualDeliveryDays
Missing DeliveryDate and ActualDeliveryDays values were investigated together with DeliveryStatus.
The analysis indicated that these missing values were associated with orders where delivery could not be completed, such as lost shipments.
These values were therefore retained as NULL rather than artificially imputing delivery information that did not exist.
VehicleType and PaymentMethod
The analysis found that missing values in:
•	VehicleType
•	PaymentMethod
•	FuelSurchargeIDR
occurred in the same set of records.
Since VehicleType and PaymentMethod are categorical variables and their actual values could not be reliably determined, the missing values were replaced with:
**NULL → **Unknown
This preserves the records while clearly indicating that the original information was unavailable.
FuelSurchargeIDR
FuelSurchargeIDR was treated differently because it is a numerical variable.
The missing values were examined against DeliveryStatus, and the missing records were associated with multiple delivery statuses.
Median imputation was then applied to the missing FuelSurchargeIDR values.
The median was selected because it is less sensitive to extreme values than the mean and is therefore more appropriate for potentially skewed monetary data.
CustomerRating
CustomerRating was intentionally left as NULL.
Customer ratings represent optional customer feedback, so the absence of a rating does not necessarily indicate a data quality problem. Replacing these values with an arbitrary value could introduce misleading information into subsequent analysis.
4. Duplicate Order Analysis
Duplicate records were investigated using OrderID.
The analysis showed that several OrderID values appeared more than once. The duplicate records were compared across the available columns to determine whether they represented:
•	Exact duplicate records, or
•	Records containing conflicting values.
Most duplicate records contained identical information across the columns.
Two duplicate cases showed differences in ShippingCostIDR. A cleaning rule was therefore applied to retain one record per OrderID, using the lower ShippingCostIDR value as the selected record.
After the transformation, the dataset was checked again to ensure that no duplicate OrderID remained.
Note: In a production environment, conflicting duplicate records should ideally be validated against the original transaction source before deciding which value is correct.
5. Data Standardization
The OriginCity and DestinationCity columns contained inconsistent formatting, including differences in capitalization and unnecessary spaces.
The values were standardized by:
•	Removing leading and trailing spaces.
•	Converting city names to uppercase.
For example:
Jakarta → JAKARTA
This standardization improves consistency when grouping, filtering, or visualizing city-level logistics data.
6. Final Clean Dataset
After completing the cleaning and transformation process, the cleaned dataset was stored in a new table:
ShippingLogistics_Cleaned
The original data remained preserved, while the new table contains the version prepared for further analysis.
Final Data Pipeline
Raw Data
ShippingLogistics
↓
Working Table
ShippingLogistics_Cleaning
↓
Data Quality Analysis
Missing Values + Duplicates + Inconsistencies
↓
Data Cleaning & Transformation
Imputation + Duplicate Removal + Standardization
↓
Final Dataset
ShippingLogistics_Cleaned
Key Data Quality Decisions
Data Issue	Treatment	Reason
DelayReason missing	No Delay	Associated with successfully delivered orders
DeliveryDate missing	Kept as NULL	Delivery information was unavailable for undelivered cases
ActualDeliveryDays missing	Kept as NULL	Cannot calculate delivery duration without delivery completion
VehicleType missing	Unknown	Categorical information was unavailable
PaymentMethod missing	Unknown	Categorical information was unavailable
FuelSurchargeIDR missing	Median imputation	Numerical variable requiring a reasonable estimate
CustomerRating missing	Kept as NULL	Customer feedback is optional
Duplicate OrderID	One record retained	Prevent duplicate orders from affecting analysis
City formatting	TRIM + UPPER	Standardize location names
Tools & Technologies
•	Microsoft SQL Server
•	T-SQL
•	INFORMATION_SCHEMA
•	CTE (WITH)
•	Window Functions
•	ROW_NUMBER()
•	PERCENTILE_CONT()
•	Dynamic SQL (T-SQL)
•	Data Cleaning & Data Preparation
Outcome
The project produced a structured and cleaned logistics dataset that is ready for the next stage of the data analytics workflow.
The cleaned data can subsequently be used to analyze:
•	Delivery performance
•	Delivery delays
•	Average delivery time
•	Shipping costs
•	Fuel surcharge patterns
•	Courier performance
•	Vehicle utilization
•	Origin and destination performance
•	Customer ratings
•	Delivery status distribution
This project demonstrates the use of SQL not only for querying data, but also for systematic data quality assessment, transformation, and preparation for business analysis.

