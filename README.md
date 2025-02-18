# Data Preparation Process for PhoneNow: Customer Churn Analysis

This process cleans and organizes raw data to make it ready for analysis. It involves importing data, fixing errors, and transforming it into a clear structure.

Here are the steps I followed:
1. [Data Import](#data-import)
2. [Data Cleaning](#data-cleaning)
3. [Data Transformation](#data-transformation)
4. [Create View](#create-view)

## Data Import

The first step is to import the raw CSV data into the database. This ensures the data is loaded correctly and ready for the next steps, such as cleaning, transformation, and creating views.

Below are the SQL queries and results for each Import and Inspection step.
```sql
/* ================================================================================
   STEP 1: DATA IMPORT
   ================================================================================ */


/* --------------------------------------------------------------------------------
   1.1. Create Table: customer_churn
   -------------------------------------------------------------------------------- */

CREATE TABLE 
	customer_churn(
	customerid VARCHAR(100) PRIMARY KEY,
	gender VARCHAR(20),
	seniorcitizen VARCHAR(20),
	partner VARCHAR(20),
	dependents VARCHAR(2 0),
	tenure INT,
	phoneservice VARCHAR(20),
	multiplelines VARCHAR(20),
	internetservice VARCHAR(50),
	onlinesecurity VARCHAR(20),
	onlinebackup VARCHAR(20),
	deviceprotection VARCHAR(20),
	techsupport VARCHAR(20),
	streamingtv VARCHAR(20),
	streamingmovies VARCHAR(20),
	contract VARCHAR(20),
	paperlessbilling VARCHAR(20),
	paymentmethod VARCHAR(50),
	monthlycharges VARCHAR(20),
	totalcharges VARCHAR(20),
	numadmintickets INT,
	numtechtickets INT,
	churn VARCHAR(20)
);

-- CSV Data Issue:
-- - Numeric columns (monthlycharges and totalcharges) in the CSV file include commas (','),
--   which prevents direct conversion to NUMERIC types during table creation (customer_churn).


/* --------------------------------------------------------------------------------
   1.2. Import data from csv files into customer_churn table
   -------------------------------------------------------------------------------- */

COPY 
	customer_churn 
FROM 
	'D:\Downloads\PwC Switzerland\PhoneNow Customer Churn.csv' 
DELIMITER 
	';' 
CSV HEADER
;
```

## Data Cleaning

After importing the data, the next step is to check for any missing values and duplicates. This helps ensure data integrity before moving on to transformation and analysis.

Below are the SQL queries and results for each Cleaning step.
```sql
/* ================================================================================
   STEP 2: DATA CLEANING
   ================================================================================ */


/* --------------------------------------------------------------------------------
    2.1 Null Handling for customer_churn
   -------------------------------------------------------------------------------- */

SELECT 
	COUNT(*) FILTER(WHERE customerid IS NULL) AS customerid_nulls,
	COUNT(*) FILTER(WHERE gender IS NULL) AS gender_nulls,
	COUNT(*) FILTER(WHERE seniorcitizen IS NULL) AS seniorcitizen_nulls,
	COUNT(*) FILTER(WHERE partner IS NULL) AS partner_nulls,
	COUNT(*) FILTER(WHERE dependents IS NULL) AS dependents_nulls,
	COUNT(*) FILTER(WHERE tenure IS NULL) AS tenure_nulls,
	COUNT(*) FILTER(WHERE phoneservice IS NULL) AS phoneservice_nulls,
	COUNT(*) FILTER(WHERE multiplelines IS NULL) AS multiplelines_nulls,
	COUNT(*) FILTER(WHERE internetservice IS NULL) AS internetservice_nulls,
	COUNT(*) FILTER(WHERE onlinesecurity IS NULL) AS onlinesecurity_nulls,
	COUNT(*) FILTER(WHERE onlinebackup IS NULL) AS onlinebackup_nulls,
	COUNT(*) FILTER(WHERE deviceprotection IS NULL) AS deviceprotection_nulls,
	COUNT(*) FILTER(WHERE techsupport IS NULL) AS techsupport_nulls,
	COUNT(*) FILTER(WHERE streamingtv IS NULL) AS streamingtv_nulls,
	COUNT(*) FILTER(WHERE streamingmovies IS NULL) AS streamingmovies_nulls,
	COUNT(*) FILTER(WHERE contract IS NULL) AS contract_nulls,
	COUNT(*) FILTER(WHERE paperlessbilling IS NULL) AS paperlessbilling_nulls,
	COUNT(*) FILTER(WHERE paymentmethod IS NULL) AS paymentmethod_nulls,
	COUNT(*) FILTER(WHERE monthlycharges IS NULL) AS monthlycharges_nulls,
	COUNT(*) FILTER(WHERE totalcharges IS NULL) AS totalcharges_nulls,
	COUNT(*) FILTER(WHERE numadmintickets IS NULL) AS numadmintickets_nulls,
	COUNT(*) FILTER(WHERE numtechtickets IS NULL) AS numtechtickets_nulls,
	COUNT(*) FILTER(WHERE churn IS NULL) AS churn_nulls
FROM 
	customer_churn
;

-- Nul and Empty Value Observation:
-- - No actual NULL values are found.
-- - However, many cells in the totalcharges column are empty but contain a space (' ').
-- *Plan:* Replace these spaces with 0 during Data Transformation.


/* --------------------------------------------------------------------------------
    2.2 Duplicate Check for customer_churn
   -------------------------------------------------------------------------------- */

SELECT 
	COUNT(customerid) 
FROM 
	customer_churn
GROUP BY 
	customerid
HAVING 
	COUNT(customerid) > 1
ORDER BY 
	customerid
;

-- No duplicate records found based on customerid.
```

## Data Transformation

In this step, the data is refined to improve consistency and usability. The transformations include updating and converting the data types for some of the columns.

Below are the SQL queries and results for each transformation step.
```sql
/* ================================================================================
   STEP 3: DATA TRANSFORMATION
   ================================================================================ */


/* --------------------------------------------------------------------------------
   3.1. Update 'totalcharges' Column Values
   -------------------------------------------------------------------------------- */

-- Replace empty spaces (' ') in 'totalcharges' with 0.

UPDATE
	customer_churn
SET
	totalcharges = 
		CASE 
		WHEN totalcharges = ' ' THEN '0'
		ELSE totalcharges
		END
;


/* --------------------------------------------------------------------------------
   3.2. Update 'monthlycharges' and 'totalcharges' Column Values 
   -------------------------------------------------------------------------------- */

-- Replace commas (',') with periods ('.') in both 'monthlycharges' and 'totalcharges',
-- to meet numeric type requirements.

UPDATE 
	customer_churn
SET
	monthlycharges = REPLACE(monthlycharges, ',', '.'),
	totalcharges = REPLACE(totalcharges, ',', '.')
;


/* --------------------------------------------------------------------------------
   3.3. Type Conversion for 'monthlycharges' and 'totalcharges' Column
   -------------------------------------------------------------------------------- */

-- Convert both 'monthlycharges' and 'totalcharges' columns to NUMERIC(10,2).

ALTER TABLE 
	customer_churn
ALTER COLUMN
	monthlycharges TYPE NUMERIC(10,2)
	USING monthlycharges :: NUMERIC(10,2),
ALTER COLUMN
	totalcharges TYPE NUMERIC(10,2)
	USING totalcharges :: NUMERIC(10,2)


/* --------------------------------------------------------------------------------
   3.4. Update All Services Column
   -------------------------------------------------------------------------------- */

-- Update service columns to simplify categorization for Power BI visualizations.

UPDATE
	customer_churn
SET
	phoneservice = 
		CASE 
      		WHEN phoneservice = 'Yes' 
		THEN 'Phone Service' 
      		ELSE phoneservice 
		END,

	multiplelines = 
		CASE 
		WHEN multiplelines = 'Yes' 
		THEN 'Multiple Lines' 
		ELSE multiplelines 
		END,

	internetservice = 
		CASE 
		WHEN internetservice = 'DSL' 
		THEN 'Internet Service - DSL' 
		WHEN internetservice = 'Fiber optic' 
		THEN 'Internet Service - Fiber Optic'
		ELSE internetservice
		END,

	onlinesecurity =
		CASE
		WHEN onlinesecurity = 'Yes'
		THEN 'Online Security'
		ELSE onlinesecurity
		END,

	onlinebackup =
		CASE 
		WHEN onlinebackup = 'Yes'
		THEN 'Online Backup'
		ELSE onlinebackup
		END,

	deviceprotection =
		CASE
		WHEN deviceprotection = 'Yes'
		THEN 'Device Protection'
		ELSE deviceprotection
		END,

	techsupport =
		CASE
		WHEN techsupport = 'Yes'
		THEN 'Tech Support'
		ELSE techsupport
		END,

	streamingtv =
		CASE 
		WHEN streamingtv = 'Yes'
		THEN 'Streaming TV'
		ELSE streamingtv
		END,

	streamingmovies =
		CASE
		WHEN streamingmovies = 'Yes'
		THEN 'Streaming Movies'
		ELSE streamingmovies
		END
;
```

## Create View

To enhance data organization and support analytics, database views are created, making it simple to reporting and dashboard development by pre-aggregating relevant data.

Below are the SQL queries and results for each transformation step.
```sql
/* ================================================================================
   STEP 4: CREATE VIEW
   ================================================================================ */


/* --------------------------------------------------------------------------------
   4.1. Create View 'service_type' 
   -------------------------------------------------------------------------------- */

-- Create View 'service_type':
-- - Reformats data to better display service categories and churn totals.
-- - This is needed because the original data lists all services for a customer in a single row,
--   which complicates visualizations.

CREATE VIEW service_type AS
SELECT 
	customerid,
	service
FROM (
	SELECT customerid, phoneservice AS service
	FROM customer_churn
	WHERE phoneservice LIKE 'Phone%'
		
	UNION ALL
	
	SELECT customerid, multiplelines AS service
	FROM customer_churn
	WHERE multiplelines LIKE 'Multiple%'
	
	UNION ALL
	
	SELECT customerid, internetservice AS service
	FROM customer_churn
	WHERE internetservice LIKE 'Internet%'
	
	UNION ALL
	
	SELECT customerid, onlinesecurity AS service
	FROM customer_churn 
	WHERE onlinesecurity LIKE '%Security'
	
	UNION ALL
	
	SELECT customerid, onlinebackup AS service
	FROM customer_churn
	WHERE onlinebackup LIKE '%Backup'
	
	UNION ALL
	
	SELECT customerid, deviceprotection AS service
	FROM customer_churn
	WHERE deviceprotection LIKE 'Device%'
	
	UNION ALL
	
	SELECT customerid, techsupport AS service
	FROM customer_churn
	WHERE techsupport LIKE 'tech%'
	
	UNION ALL
	
	SELECT customerid, streamingtv AS service
	FROM customer_churn
	WHERE streamingtv LIKE '%TV'
	
	UNION ALL
	
	SELECT customerid, streamingmovies AS service
	FROM customer_churn
	WHERE streamingmovies LIKE '%Movies')
GROUP BY
	customerid,
	service
ORDER BY
	customerid
;


/* --------------------------------------------------------------------------------
   4.2. Create View 'bundle_type' 
   -------------------------------------------------------------------------------- */

-- Create View 'bundle_type':
-- - Prepares for future optimizations (e.g., bundling products for PhoneNow.inc) to offer customers interesting choices.

CREATE VIEW bundle_services AS
SELECT 
	customerid,
	STRING_AGG(service, ', ' ORDER BY service_order) AS services
FROM (
	SELECT customerid, phoneservice AS service, 1 AS service_order
	FROM customer_churn
	WHERE phoneservice LIKE 'Phone%'
		
	UNION ALL
	
	SELECT customerid, multiplelines AS service, 2 AS service_order
	FROM customer_churn
	WHERE multiplelines LIKE 'Multiple%'
	
	UNION ALL
	
	SELECT customerid, internetservice AS service, 3 AS service_order
	FROM customer_churn
	WHERE internetservice LIKE 'Internet%'
	
	UNION ALL
	
	SELECT customerid, onlinesecurity AS service, 4 AS service_order
	FROM customer_churn 
	WHERE onlinesecurity LIKE '%Security'
	
	UNION ALL
	
	SELECT customerid, onlinebackup AS service , 5 AS service_order
	FROM customer_churn
	WHERE onlinebackup LIKE '%Backup'
	
	UNION ALL
	
	SELECT customerid, deviceprotection AS service, 6 AS service_order
	FROM customer_churn
	WHERE deviceprotection LIKE 'Device%'
	
	UNION ALL
	
	SELECT customerid, techsupport AS service, 7 AS service_order
	FROM customer_churn
	WHERE techsupport LIKE 'tech%'
	
	UNION ALL
	
	SELECT customerid, streamingtv AS service, 8 AS service_order
	FROM customer_churn
	WHERE streamingtv LIKE '%TV'
	
	UNION ALL
	
	SELECT customerid, streamingmovies AS service, 9 AS service_order
	FROM customer_churn
	WHERE streamingmovies LIKE '%Movies')
GROUP BY
	customerid
ORDER BY
	customerid
;
```
---
These queries will produce the final datasets that will be imported into Power BI to uncover insights.
