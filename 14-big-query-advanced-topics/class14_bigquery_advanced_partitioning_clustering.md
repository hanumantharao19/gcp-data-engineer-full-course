# **Class 14: BigQuery Advanced – Partitioning & Clustering**

## 🎯 **Learning Objectives**
By the end of this class, you’ll be able to:
1. Understand **why partitioning and clustering matter** for query performance.  
2. Learn how to **create and manage partitioned and clustered tables**.  
3. Know when to use **both together**.  
4. Monitor and optimize **query cost** using these techniques.  

---

## 🧠 **1️⃣ Why We Need Optimization**
BigQuery charges **by data scanned**, not by rows returned.  
➡️ If you query a 1 TB table but need only a few MB, you still pay for 1 TB (unless partitioning/clustering is used).  

Partitioning and clustering reduce:
- **Query cost**
- **Execution time**
- **Resource usage**

---

## 🗂️ **2️⃣ Partitioning – Breaking Data into Logical Chunks**

### 🏗️ What Is Partitioning?
Partitioning splits a table into segments (partitions) based on a **column value**, commonly **DATE**, **TIMESTAMP**, or **INTEGER RANGE**.

Each partition is stored and processed separately.

```
sales_data (partitioned by order_date)
├── 2025-10-01
├── 2025-10-02
├── 2025-10-03
```

---

### 📦 **Types of Partitioning**

| Type | Description | Example |
|------|--------------|----------|
| **Ingestion-time** | Automatically partitions by data load time | Logs, streaming data |
| **Column-based** | Partition by a specific column (DATE/TIMESTAMP) | `order_date` |
| **Integer-range** | Custom range partitions | `customer_id` between 1000–2000 |
| **Time-unit column** | Partition by `DAY`, `MONTH`, `YEAR` | `PARTITION BY DATE(order_date)` |

---

### 🧰 **Create Partitioned Table**
```sql
CREATE TABLE myproject.sales.sales_partitioned
PARTITION BY DATE(order_date)
AS
SELECT * FROM myproject.sales.transactions;
```

Or when loading from CSV:
```bash
bq load --time_partitioning_field order_date myproject:sales.sales_partitioned gs://my-bucket/sales.csv
```

---

### 🔍 **Querying Partitioned Tables**
Use filters on the partition column:
```sql
SELECT SUM(amount) AS daily_sales
FROM `myproject.sales.sales_partitioned`
WHERE order_date BETWEEN '2025-10-01' AND '2025-10-07';
```
💡 BigQuery scans only those partitions → lower cost + faster query.

---

### ⚙️ **Managing Partitions**
| Command | Description |
|----------|--------------|
| `bq show --format=prettyjson` | View partition details |
| `bq partition delete` | Remove partitions manually |
| `bq query --destination_table` | Append data to partitions |
| **Partition expiration** | Auto-delete old data to save cost |

Example:
```sql
CREATE TABLE sales_partitioned (
  order_id STRING, 
  order_date DATE, 
  amount FLOAT64
)
PARTITION BY DATE(order_date)
OPTIONS (partition_expiration_days = 90);
```

---

## 🔢 **3️⃣ Clustering – Organizing Within Partitions**

### 🧩 What Is Clustering?
Clustering arranges data **within each partition** (or whole table) based on one or more columns.  

BigQuery then uses clustering columns to **reduce scanned blocks** during queries.

```
sales_data (partitioned by order_date, clustered by customer_id)
```

---

### 🧱 **Creating Clustered Table**
```sql
CREATE TABLE myproject.sales.sales_clustered
PARTITION BY DATE(order_date)
CLUSTER BY customer_id, region
AS
SELECT * FROM myproject.sales.transactions;
```

---

### 💡 **When to Use Clustering**
✅ Ideal when:
- Data volume is large > 10 GB per partition  
- Queries often filter by specific columns  
- Cardinality (distinct values) is moderate (1K – 1M)

❌ Avoid when:
- Small tables (< 1 GB)
- High-cardinality columns (unique IDs per row)
- Queries don’t filter by cluster columns

---

### 📊 **Performance Comparison**

| Technique | When to Use | Effect |
|------------|-------------|---------|
| **Partitioning** | Time-based queries | Reduce data scanned |
| **Clustering** | Column-based queries | Faster filter & join |
| **Both** | Time + Column filters | Best performance combo |

---

## 💰 **4️⃣ Cost Optimization with Partitioning & Clustering**

### Example – Without Partitioning:
```sql
SELECT * FROM sales WHERE order_date > '2025-10-01';
```
→ Scans entire table (Expensive)

### With Partitioning:
```sql
SELECT * FROM sales_partitioned WHERE order_date > '2025-10-01';
```
→ Scans only recent partitions (Cheap)

### With Partitioning + Clustering:
```sql
SELECT * FROM sales_clustered
WHERE order_date > '2025-10-01' AND customer_id = 12345;
```
→ Scans only a tiny subset (Fast + Low Cost)

---

## 📈 **5️⃣ Monitoring Query Performance**

In the BigQuery Console:
- Click “Execution Details” tab to see **bytes scanned**.  
- Use `EXPLAIN` keyword to inspect query plan:  
  ```sql
  EXPLAIN SELECT * FROM sales_clustered WHERE region = 'APAC';
  ```
- Use **INFORMATION_SCHEMA.PARTITIONS** to inspect metadata.

---

## 🧮 **6️⃣ Best Practices**

✅ Partition by DATE/TIMESTAMP columns  
✅ Cluster on columns frequently used in WHERE/JOIN  
✅ Avoid too many partitions (< 4000 recommended)  
✅ Monitor scanned bytes regularly  
✅ Use table expiration for temporary data  
✅ Combine with Materialized Views for fast dashboards  

---

## 🚀 **7️⃣ Hands-On Exercise**

1. Create a partitioned table by `DATE(order_date)`  
2. Insert sample sales records  
3. Query specific date range and note bytes scanned  
4. Recreate same table with `CLUSTER BY customer_id`  
5. Compare query speed & cost  

---

## 🏁 **8️⃣ Summary**

| Concept | Key Point |
|----------|------------|
| **Partitioning** | Break large table into smaller date or range segments |
| **Clustering** | Organize data within partitions for fast filtering |
| **Combined Use** | Best balance of speed and cost |
| **Monitoring** | Always track bytes processed per query |

---

## 💡 **Next Class (Class 15):**
➡️ **Dataflow (Apache Beam) – Building Batch Pipelines**
