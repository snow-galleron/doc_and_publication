# Data Lineage in Dataform: A 7-Stage Process

This document details a comprehensive 7-stage data lineage process using Dataform, designed to transform raw data into clean, structured, and readily analyzable information for business use cases.  Each stage builds upon the previous one, progressively refining the data.  We also include an 8th stage for business-specific outputs.

## Data Lineage Stages

The data pipeline is structured into the following stages:

**0. Raw History**

* **Description:** This initial stage captures the complete historical data from all source systems, retained for 90 days.  It serves as an audit trail and allows for historical analysis.
* **Naming Convention:** `<Source System Name>_history`
* **Purpose:** Preserve historical data for auditing and retrospective analysis.
* **Example SQL:**
```sql
SELECT distinct _PARTITIONDATE
FROM source_system.odoo_table_history
WHERE _PARTITIONDATE >= DATE_SUB(CURRENT_DATE(), INTERVAL 90 DAY);
```
* **Example Tables:** `partner_history`, `orderline_history`, `table_from_hubspot_history`

___

**1. Raw**

* **Description:**  This stage contains the most current snapshot of data from each source system, cleaned of historical entries.  It represents the current state of each source.
* **Naming Convention:** `<Source System Name>_raw`
* **Purpose:** Provide an up-to-date view of data from each source, without historical context.
* **Example SQL:**
```sql
SELECT * 
FROM source_system.odoo_table_history
WHERE _PARTITIONDATE = CURRENT_DATE();
```
* **Example Tables:** `odoo.partner`, `orderline`, `table_from_hubspot`


**2. Enriched**

* **Description:**  Basic data transformations are applied here, such as column renaming, data type standardization, and initial data model application.  No complex logic or joins are performed.
* **Naming Convention:** `<Source System Name>_enriched`
* **Purpose:** Harmonize field names and data formats across tables, preparing the data for subsequent transformations.
* **Example SQL:**
```sql
SELECT 
  odoo_id AS id,
  customer_name AS name,
  FORMAT_TIMESTAMP('%Y-%m-%d', created_at) AS creation_date
FROM customer_raw;
```
* **Example Tables:** `odoo1_enriched`, `odoo2_enriched`, `hubspot_enriched`, `salesforce_enriched`


**3. Transformed**

* **Description:** More complex transformations are applied at this stage, including data cleaning, aggregations, and manipulations specific to each source system.  Data remains isolated by source.
* **Naming Convention:** `<Source System Name>_transformed`
* **Purpose:** Perform source-specific data cleaning and preparation before integration with other sources.
* **Example SQL:**
```sql
SELECT 
  id, 
  name, 
  country,
  CASE WHEN revenue > 10000 THEN 'high' ELSE 'low' END AS revenue_category
FROM customer_enriched;
```
* **Example Tables:** `odoo1_transformed`, `odoo2_transformed`, `hubspot_transformed`, `salesforce_transformed`


**4. Full**

* **Description:** This critical stage integrates data from multiple source systems.  Data from different sources representing the same entity are joined and combined into a unified view.
* **Naming Convention:** `<Entity Name>_full` (e.g., combining Odoo1 and Odoo2 data into `odoo_full`)
* **Purpose:** Create a consolidated dataset by merging data from disparate sources.
* **Example SQL:**
```sql
SELECT 
  odoo1.id,
  odoo1.name,
  odoo2.additional_info
FROM customer_transformed odoo1
UNION ALL customer_transformed odoo2;
```
* **Example Tables:** `customer_full`, `order_full`, `user_full`


**5. Taxonomy**

* **Description:**  Focuses on standardizing the data vocabulary and ensuring consistent naming conventions across all integrated datasets.
* **Naming Convention:** `<Entity Name>_taxonomy`
* **Purpose:** Enforce a standardized taxonomy, ensuring data coherence and consistency across the entire platform.
* **Example SQL:**
```sql
SELECT 
  LOWER(name) AS standardized_name,
  UPPER(country) AS standardized_country
FROM customer_full;
```
* **Example Tables:** `customer_taxonomy`, `order_taxonomy`, `user_taxonomy`


**6. Core**

* **Description:** This stage represents the final, standardized data model ready for analysis and reporting.  It's the culmination of all previous transformations.
* **Naming Convention:** `<Entity Name>_core`
* **Purpose:** Provide a consistent, well-structured dataset for downstream analytics and reporting applications.
* **Example SQL:**
```sql
SELECT 
  id, 
  standardized_name, 
  standardized_country, 
  revenue_category
FROM customer_taxonomy;
```
* **Example Tables:** `customer_core`, `sales_core`, `revenue_core`


**7. Metrics**

* **Description:** Pre-calculated metrics and aggregations are generated from the core models, simplifying reporting and dashboard creation.
* **Naming Convention:** `<Entity Name>_metrics`
* **Purpose:**  Streamline report generation by providing readily available, pre-computed metrics.
* **Example SQL:**
```sql
SELECT 
  COUNT(id) AS customer_count,
  SUM(revenue) AS total_revenue
FROM customer_core;
```
* **Example Tables:** `customer_metrics`, `sales_metrics`, `revenue_metrics`


**8. Business Case / Export / Dashboard**

* **Description:** This stage contains custom datasets and exports tailored to specific business needs, dashboards, or reports. These are derived from the core models or metrics tables.
* **Naming Convention:** Custom naming based on the specific business case or dashboard (e.g., `dashboard_sales`, `marketing_report_q3_2024`).
* **Purpose:**  Provide readily consumable data for specific business analyses and reporting requirements.
* **Example SQL:**
```sql
SELECT 
  customer_core.id, 
  customer_core.name, 
  sales_metrics.total_revenue
FROM customer_core
JOIN sales_metrics ON customer_core.id = sales_metrics.customer_id;
```
* **Example Tables:** `dashboard_sales`, `use_case_export`, `marketing_dashboard`


## Summary of Data Lineage Stages

This multi-stage process ensures data quality, consistency, and traceability, allowing for efficient and reliable data analysis and reporting.  The clear naming conventions facilitate understanding and navigation of the data pipeline.
