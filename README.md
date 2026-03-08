# Azure_DataEngineering_Retail_Project
# End‑to‑end Azure Data Engineering solution (ADF + ADLS Gen2 + Databricks + Delta + Power BI)

Retail Analytics Project – Azure Data Engineering (Medallion Architecture)
This project implements an end‑to‑end retail analytics pipeline on Azure using the Medallion architecture (Bronze–Silver–Gold), starting from multiple data sources and ending with analytics dashboards.

Architecture Overview:
Azure Data Factory (ADF): Ingests data from:
    Azure SQL Database: products, stores, transactions
    JSON files: customers
Azure Data Lake Storage Gen2 (ADLS Gen2): Landing and curated zones for Bronze, Silver, and Gold layers.
Azure Databricks: Data processing, cleaning, transformations, and aggregations using PySpark and Delta Lake.
Visualization: Dashboards built on top of Gold layer using Databricks SQL / Power BI.

Data Flow:
1.Ingestion (ADF → ADLS Gen2)
    Copy activities in ADF pipelines move:
    SQL tables (products, stores, transactions) to Bronze/Products, Bronze/Stores, Bronze/Transactions as Parquet.
    Customer JSON to Bronze/Customers.
2.Bronze Layer (Raw Data)
    Databricks reads raw Parquet/JSON from ADLS Gen2:
    df_transactions, df_products, df_stores, df_customers.
    Data is stored in raw form for traceability and replay.
3.Silver Layer (Clean & Conformed):
    Type casting and basic cleaning:
      Cast IDs and numeric fields (transaction_id, customer_id, product_id, store_id, quantity, price) to correct types.
      Convert transaction_date to date.
      Select relevant columns and drop duplicates in customers on customer_id.
    Business model join:
      Join transactions with customers, products, and stores on their keys.
      Compute total_amount = quantity * price.
    Persist as Delta:
      Write to Silver/Cleaned_retail in ADLS Gen2.
      Create Delta table retail_catalog.default.retail_silver_cleaned pointing to this location.
4.Gold Layer (Aggregated & Analytics Ready)
    Read cleaned Delta data from Silver.
    Aggregate metrics at transaction_date, product, and store grain:
      total_quantity_sold
      total_sales_amount
      number_of_transactions
      average_transaction_value
    Persist as Delta:
      Write to Gold/Aggregated_data.
      Create Delta table retail_catalog.default.retail_gold.
      
Analytics & Dashboarding:
    Power BI reports through a direct connection to the Gold Delta table.


