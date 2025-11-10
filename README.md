<img width="1488" height="725" alt="Screenshot 2025-11-10 at 23 37 00" src="https://github.com/user-attachments/assets/a7c69c03-cee3-4c45-bf07-87b3a1f18cbe" />
<img width="1489" height="712" alt="Screenshot 2025-11-10 at 23 37 21" src="https://github.com/user-attachments/assets/520b0a37-8470-44a4-a127-dd05ba4f0de8" />

# 🚀 Incremental Data Load Pipeline – Azure Data Factory

This project implements an **incremental data ingestion pipeline** using **Azure Data Factory (ADF)** to move data from **Azure SQL Database** into **Azure Data Lake Storage Gen2** efficiently.  
It’s built around the idea of tracking a *watermark (CDC column)* to capture only new or updated records on each run — no redundant full loads.

---

## 🧠 Project Overview

The pipeline dynamically constructs SQL queries at runtime using parameters such as schema name, table name, and CDC column.  
It stores the last loaded CDC value (a *watermark*) in ADLS as a JSON file, which acts as a checkpoint for the next run.

On every execution:
1. The pipeline **reads the last watermark** from ADLS.  
2. **Builds a query** that fetches only rows newer than that CDC value.  
3. **Copies new data** from Azure SQL to ADLS (Bronze layer).  
4. **Updates the watermark file** for the next incremental run.

---


---

## ⚙️ Pipeline Components

| Activity | Purpose |
|-----------|----------|
| **Lookup (Last_cdc)** | Reads the last CDC value from a JSON file in ADLS. |
| **Copy Data** | Copies new/changed rows from Azure SQL using a dynamic query. |
| **If Condition** | Checks whether watermark file exists (handles first-run logic). |
| **Set Variable / Stored Proc** | Updates the watermark JSON after successful load. |

---

## 🧩 Dynamic SQL Example

```sql
SELECT *
FROM @{pipeline().parameters.schema}.@{pipeline().parameters.table}
WHERE @{pipeline().parameters.cdc_collunm} > 
'@{if(empty(pipeline().parameters.fromdate),
       activity('Last_cdc').output.value[0].cdc,
       pipeline().parameters.fromdate)}'


