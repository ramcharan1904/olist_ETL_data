# olist_ETL_data
This project is designed as an end-to-end Big Data Engineering initiative utilizing an e-commerce dataset, which is distributed across multiple files and leverages both SQL and NoSQL databases.  The project employs a Medallion Architecture, organizing data into three distinct layers: Bronze, Silver, and Gold, ensuring incremental improvement in data quality and structure throughout the pipeline. 

The workflow is structured into key phases:
•
Data Ingestion: Raw data from various sources, including HTTPS requests (e.g., GitHub) and SQL databases, is ingested into Azure Data Lake Storage Gen 2 (ADLs Gen 2), forming the Bronze layer. This is primarily facilitated by Azure Data Factory, which utilizes features like parameterization and lookup activities for efficient processing of numerous files.
•
Data Transformation: The raw data is then processed and transformed using Azure Databricks. This stage involves cleaning, renaming, filtering, and joining multiple datasets, along with enriching the data by integrating information from a MongoDB database, resulting in a refined Silver layer in ADLs Gen 2.
•
Data Serving: The final Gold layer is prepared using Azure Synapse Analytics. This involves reading transformed data, performing advanced SQL queries, creating views, and generating external tables in ADLs Gen 2, making the data readily accessible for data scientists, analysts, and visualization tools.
This approach provides a robust simulation of industry-standard big data pipelines
