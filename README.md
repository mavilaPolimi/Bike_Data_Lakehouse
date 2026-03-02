# End-to-End Data Lakehouse Pipeline
### Powered by Databricks, Delta Lake, and Unity Catalog

## Project Overview
This project implements a scalable **Medallion Architecture** on the Databricks Lakehouse Platform. It transforms raw, siloed datasets into a high-performance **Star Schema** optimized for business intelligence. The pipeline is fully automated using Databricks Workflows, ensuring daily delivery of curated data for stakeholders.

## Technical Architecture
The project follows the industry-standard Medallion pattern to ensure data quality and lineage:

1.  **Bronze (Raw Layer):** 1:1 ingestion of source data into Delta tables.
2.  **Silver (Cleansed Layer):** Source-specific cleaning, normalization, and deduplication.
3.  **Gold (Curated Layer):** Dimensional modeling (Star Schema) for analytics-ready data.

### System Diagrams

| Architecture Overview | Data Flow & Lineage | Integration Model |
| :--- | :--- | :--- |
| ![Architecture](Images/architecture_diagram.png) | ![Lineage](Images/data_flow.png) | ![Integration](Images/integration_model.png) |

---

## Implementation Details

### 1. Data Ingestion (Bronze)
- **Source:** Datasets stored in **Unity Catalog Volumes**.
- **Process:** A single ingestion notebook performs a 1:1 copy of the raw files into the `bronze` schema.
- **Outcome:** Preservation of the raw state for auditability and reprocessing.

### 2. Transformation & Cleaning (Silver)
- **Modularity:** 6 dedicated notebooks (one per data source) handle specific cleaning logic, schema enforcement, and type casting.
- **Orchestration:** A `silver_orchestration` notebook manages the sequential execution of these 6 notebooks, ensuring all dependencies are met before proceeding.
- **Data Modeling:** An **Integration Model** was developed to identify common attributes across the 6 silver tables, facilitating the join logic required for the Gold layer.

### 3. Dimensional Modeling (Gold)
- **Business Objects identified:** `Customer`, `Product`, and `Sale`.
- **Structure:** 
  - **Dimensions:** `dim_customer` and `dim_product` (Type 1 SCD logic) .
  - **Facts:** `fact_sale` containing transactional metrics and foreign keys.
- **Orchestration:** A `gold_orchestration` notebook triggers the creation of the dimensions followed by the fact table.

---

## Pipeline Orchestration (Databricks Jobs)
The entire pipeline is automated using **Databricks Workflows** with the following task dependency chain:

`Bronze Ingestion` ➡️ `Silver Orchestration` ➡️ `Gold Orchestration`

- **Automation:** Configured with a CRON trigger to run **Daily at 8:00 AM**.
- **Reliability:** Sequential task dependencies ensure that the Gold layer only updates if the Silver transformations are successful.

---

## Engineering Decisions
- **Notebook Orchestration vs. DLT:** I chose a notebook-based orchestration pattern to demonstrate control over task dependencies and modular code design.
- **Unity Catalog:** Leveraged Unity Catalog for fine-grained access control and centralized metadata management across the three layers.
- **Star Schema:** Opted for a Star Schema in the Gold layer to minimize join complexity for end-users in Power BI/Tableau.

## Future Improvements
- [ ] Transition to **Delta Live Tables (DLT)** for automated lineage and data quality expectations.
- [ ] Implement **SCD Type 2** for the Customer dimension to track historical changes.
- [ ] Add **Unit Testing** using the `nutter` framework for Spark.
