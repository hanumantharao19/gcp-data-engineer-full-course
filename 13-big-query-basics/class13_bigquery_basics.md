# **Class 13: BigQuery Basics – Datasets, Tables, and Queries**

---

## 🎯 **Learning Objectives**
By the end of this class, you should be able to:
1. Understand what BigQuery is and how it fits in the GCP data ecosystem.  
2. Create datasets and tables in BigQuery.  
3. Load data into BigQuery from multiple sources.  
4. Run SQL queries to analyze data efficiently.  
5. Understand pricing and performance basics.

---

## 🧠 **1. What is BigQuery?**
**BigQuery** is Google Cloud’s **serverless, scalable data warehouse** designed for:
- Running **SQL queries** on massive datasets.
- Handling **real-time analytics**.
- Integrating with **GCP tools** (Dataflow, Pub/Sub, AI, Looker Studio, etc.).

### 🏗️ Key Features
| Feature | Description |
|----------|--------------|
| **Serverless** | No infrastructure management; GCP handles scaling and performance. |
| **Columnar Storage** | Data stored in column format for fast analytics. |
| **Fully Managed** | No manual provisioning or tuning. |
| **Cost-Effective** | Pay per query or use flat-rate pricing. |
| **Integrated ML** | Build models directly using SQL (`BigQuery ML`). |

---

## 🗂️ **2. Understanding BigQuery Structure**
BigQuery organizes data like this:

```
Project
 └── Dataset
      └── Table
```

### 🔹 **Project**
- A container for all datasets, tables, and resources.  
- Each project has a unique **Project ID**.

### 🔹 **Dataset**
- Logical grouping of tables (like a schema in RDBMS).  
- Example: `myproject.sales_data`

### 🔹 **Table**
- Stores actual data in **rows and columns**.
- Each table has a **schema** (field names, types).

---

## 🧩 **3. Creating Datasets and Tables**

### **Create a Dataset**
**Method 1: Console**
1. Go to **BigQuery Console** → Click **+ Create Dataset**.
2. Give dataset ID → Set location (region).
3. Click **Create Dataset**.

**Method 2: Command Line**
```bash
bq --location=US mk --dataset myproject:customer_data
```

---

### **Create a Table**
**From a CSV file (Cloud Storage):**
```bash
bq load --source_format=CSV myproject:customer_data.customers gs://my-bucket/customers.csv name:STRING,age:INTEGER,city:STRING
```

**From Query:**
```sql
CREATE TABLE myproject.customer_data.top_customers AS
SELECT name, SUM(spend) AS total_spend
FROM myproject.customer_data.sales
GROUP BY name
ORDER BY total_spend DESC;
```

---

## 💾 **4. Loading Data into BigQuery**

| Source | Method |
|---------|---------|
| **CSV / JSON** | Upload manually or use `bq load` command |
| **Google Cloud Storage (GCS)** | Most common for bulk data loads |
| **Cloud Dataflow / Dataproc** | For large ETL pipelines |
| **BigQuery Data Transfer Service** | Automated imports from Google Ads, YouTube, etc. |

---

## 🔍 **5. Querying Data in BigQuery**
BigQuery uses **Standard SQL** (similar to PostgreSQL).

### **Basic SELECT**
```sql
SELECT name, city
FROM `myproject.customer_data.customers`
WHERE age > 25
ORDER BY name;
```

### **Aggregation**
```sql
SELECT city, COUNT(*) AS total_customers
FROM `myproject.customer_data.customers`
GROUP BY city;
```

### **Join Example**
```sql
SELECT c.name, s.amount
FROM `myproject.customer_data.customers` AS c
JOIN `myproject.customer_data.sales` AS s
ON c.id = s.customer_id;
```

---

## ⚙️ **6. Partitioning and Clustering (Basics)**
Used to **optimize performance and reduce cost**.

### Partitioning
Splits a table into segments (e.g., by date).
```sql
CREATE TABLE sales_partitioned
PARTITION BY DATE(order_date)
AS SELECT * FROM sales;
```

### Clustering
Organizes data by specific columns.
```sql
CREATE TABLE sales_clustered
CLUSTER BY customer_id
AS SELECT * FROM sales;
```

---

## 💰 **7. Pricing Model**
| Operation | Pricing |
|------------|----------|
| **Storage** | ₹1.8–₹2.0 per GB per month (approx.) |
| **Query (On-demand)** | Pay ₹4.8 per TB scanned |
| **Flat-rate** | For high-volume, predictable workloads |

🧮 **Tip:**  
To control query cost, use:
```sql
SELECT * FROM `dataset.table`
WHERE DATE(timestamp_col) = "2025-10-29"
```
→ This limits scanned data.

---

## 📊 **8. Monitoring & Logs**
Use **Query History** in BigQuery Console to:
- View query cost
- See execution time
- Check job logs

Or use **Cloud Logging** for advanced monitoring.

---

## 🚀 **9. Hands-on Exercise**
1. Create a new dataset named `retail_data`.  
2. Upload a CSV file (e.g., `sales.csv`) from Cloud Storage.  
3. Create a table `transactions`.  
4. Write a query to find top 5 products by sales.  
5. Try creating a partitioned table by `order_date`.

---

## 🏁 **10. Summary**
| Concept | Key Takeaway |
|----------|---------------|
| BigQuery | Fully managed, serverless data warehouse |
| Dataset | Logical grouping of tables |
| Table | Stores actual structured data |
| Query | SQL-based data analysis |
| Pricing | Pay only for storage + data scanned |

---

## 💡 **Next Class (Class 14):**
➡️ **BigQuery Advanced – Partitioning, Clustering & Optimization**
