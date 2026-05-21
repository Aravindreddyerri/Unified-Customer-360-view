# Unified Customer 360 View on Azure

## Project Summary
This project builds a Customer 360 analytics platform on Azure by integrating retail sales, product master data, store data, customer service interactions, loyalty activity, exchange rates, and customer profile data into one unified analytical model. The solution uses Azure Synapse pipelines for orchestration, Azure Data Lake Storage Gen2 for Bronze and Silver layers, Azure Synapse Spark notebooks for transformation, Azure SQL Database and Synapse dedicated SQL pool as the Gold serving layer, and Power BI for reporting. 

## Business Objective
Retail customer information is usually spread across multiple operational systems, which makes it difficult to analyze total spend, purchase behavior, loyalty participation, service activity, and product trends from a single view. This project solves that problem by standardizing source data, transforming it into analytics-ready structures, creating curated Gold tables, and exposing the final model for reporting and dashboarding. 

## Architecture
The solution follows a medallion-style architecture with three layers: Bronze, Silver, and Gold. Raw source files are first landed in Bronze, cleaned and standardized in Silver, then joined and aggregated into Gold for reporting. Notebook1 performs Bronze-to-Silver processing, Notebook2 performs Silver-to-Gold processing, and the final Gold data is loaded into SQL-based serving tables. 

### Flow
1. Source files are ingested into the Bronze layer in ADLS Gen2. 
2. Notebook1 cleans and standardizes the data into Silver parquet files. 
3. Notebook2 joins and aggregates Silver data into Gold tables. 
4. Gold tables are loaded into Azure SQL Database and Synapse dedicated SQL pool. 
5. Power BI connects to the Gold layer for reporting. 

## Source Data
The project uses the following source files:
- `Sales.csv`
- `Products.csv`
- `Stores.csv`
- `Exchange_Rates.csv`
- `CustomerService.csv`
- `Loyalty.csv`
- `Customers.csv`

These files include transaction records, product details, store master data, exchange rates, support cases, loyalty data, and customer demographics. 

## Bronze To Silver
The Bronze-to-Silver notebook reads all source datasets from ADLS and applies schema standardization, type casting, trimming, date conversion, and duplicate removal. 

### Main transformations
- Sales: renamed columns, converted order and delivery dates, cast keys and quantity to integers, standardized currency codes, and removed duplicates. 
- Products: renamed fields, trimmed text values, and cast unit cost and unit price fields to decimal. 
- Stores: standardized store fields, cast numeric columns, and removed duplicates. 
- Exchange rates: converted rate dates, standardized currency values, cast exchange values, and removed duplicates.
- Customer service: cleaned case data, normalized text fields, and removed duplicates. 
- Loyalty: standardized loyalty attributes and converted point fields to numeric types. 
- Customers: cleaned customer names, geography fields, and date values for downstream use. 

### Silver outputs
The notebook writes cleaned parquet files for sales, products, stores, exchange rates, customer service, loyalty, and customers into the Silver layer in ADLS. 

## Silver To Gold
The Silver-to-Gold notebook reads the curated Silver parquet files and builds business-ready analytical tables. It creates an enriched sales table, a Customer 360 summary table, and a product summary table.

### Gold tables
#### Gold_Sales_Enriched
This table joins sales with products and stores to provide a flattened transactional dataset with order details, customer key, store geography, product information, quantity, currency, unit price, and sales amount. 

#### GoldCustomer360
This table combines customer data with sales summary, service summary, and loyalty data to produce a single customer profile view with spend, activity, and support metrics. 

#### GoldProductSummary
This table aggregates sales by product attributes and includes units sold, sales amount, customer count, and order count. 
### Validation results
The notebook output shows:
- 62,884 rows in Gold Sales Enriched. 
- 15,266 rows in Gold Customer 360. 
- 2,492 rows in Gold Product Summary. 

## Serving Layer
The final Gold tables are written to Azure SQL Database and Synapse dedicated SQL pool using JDBC. The key tables are `dbo.GoldCustomer360`, `dbo.GoldProductSummary`, and `dbo.GoldSalesEnriched`. Power BI then connects to these tables for dashboards and analysis. 

## Orchestration
<img width="802" height="311" alt="Screenshot 2026-05-21 at 11 58 39 AM" src="https://github.com/user-attachments/assets/15144444-4f65-4f3d-987d-d0954e5c7378" />

## Power BI
<img width="867" height="582" alt="Screenshot 2026-05-21 at 12 04 04 PM" src="https://github.com/user-attachments/assets/5b7a34e8-18e7-4e91-95f7-d1344aedb8f0" />


## Notes
A data quality issue exists in the product pricing fields because some values contain currency symbols or spaces, which can cause nulls when cast directly to decimal. This should be handled during transformation before loading to Gold. 

The Synapse workspace also showed a capacity-related Spark error during execution, so job scheduling and vCore availability should be considered when running the notebooks.

## How To Run
1. Upload the raw source files to the Bronze layer. 
2. Run the Bronze-to-Silver notebook.
3. Run the Silver-to-Gold notebook. 
4. Load the Gold tables into SQL using JDBC. 
5. Refresh Power BI to view the latest analytics. 

## Deliverables
- Synapse pipeline orchestration. 
- Bronze-to-Silver notebook. 
- Silver-to-Gold notebook. 
- Curated Silver parquet files. 
- Final Gold tables in SQL. 
- Power BI reporting layer. 

## Conclusion
This project demonstrates an end-to-end Azure Customer 360 implementation with ingestion, transformation, aggregation, relational serving, and dashboarding. It is a strong portfolio project for data engineering and analytics engineering roles. 
