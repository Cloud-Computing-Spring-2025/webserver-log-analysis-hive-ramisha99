# Web Server Log Analysis using Apache Hive

## 📌 Project Overview
This project analyzes web server logs using **Apache Hive** to extract insights such as:
- Total web requests
- Status code analysis
- Most visited pages
- Traffic source analysis
- Suspicious IP addresses with failed requests
- Traffic trends over time

Data is stored in HDFS, and Hive queries are executed to process and retrieve meaningful analytics.

## 🛠️ Implementation Approach
The analysis is conducted through Hive queries on partitioned datasets. Key queries performed:
- **Data Ingestion**: Loading web server logs into Hive tables.
- **Partitioning**: Organizing data based on HTTP response status.
- **Aggregations & Analysis**: Extracting request patterns, user agents, top URLs, failure rates, and time-based trends.

## 🔍 Key Queries Used

### 1️⃣ **Create and Load Data**
```sql
CREATE EXTERNAL TABLE IF NOT EXISTS web_server_logs (
    ip STRING,
    `timestamp` STRING,
    url STRING,
    status INT,
    user_agent STRING
)
PARTITIONED BY (status INT)
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE
LOCATION '/user/hive/warehouse/web_logs/';

##Enable Dynamic Partitioning in Hive
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;

##Insert Data into Partitioned Table
INSERT INTO TABLE web_server_logs_partitioned PARTITION (status)
SELECT ip, `timestamp`, url, user_agent, status FROM web_server_logs;
2️⃣ Total Web Requests

SELECT COUNT(*) AS total_requests FROM web_server_logs;
3️⃣ Status Code Analysis

SELECT status, COUNT(*) AS frequency 
FROM web_server_logs 
GROUP BY status 
ORDER BY frequency DESC;
4️⃣ Most Visited Pages

SELECT url, COUNT(*) AS visit_count
FROM web_server_logs
GROUP BY url
ORDER BY visit_count DESC
LIMIT 3;
5️⃣ Traffic Source Analysis (User Agents)

SELECT user_agent, COUNT(*) AS count 
FROM web_server_logs 
GROUP BY user_agent 
ORDER BY count DESC;
6️⃣ Suspicious IP Addresses (Failed Requests)

SELECT ip, COUNT(*) AS failed_requests
FROM web_server_logs
WHERE status IN (404, 500)
GROUP BY ip
HAVING failed_requests > 3;
7️⃣ Traffic Trend Over Time

SELECT SUBSTRING(`timestamp`, 1, 16) AS minute, COUNT(*) AS request_count
FROM web_server_logs
GROUP BY SUBSTRING(`timestamp`, 1, 16)
ORDER BY minute;

🚀 Execution Steps
Queries were executed using Hue UI, and results were exported manually.

Steps:
Start Hive Metastore and Server


docker-compose up 
docker start hive-server
docker exec -it namenode bash

Download Results from Hue UI that was produced by the queries that were ran.

Query results were exported from the Hue UI and saved as CSV files.
Files saved:

query-hive-50.csv
query-hive-51.csv
query-hive-52.csv
query-hive-53.csv
query-hive-54.csv
query-hive-55.csv
query-hive-59.csv
query_results.csv

⚠️ Challenges Faced
Hive Partitioning Issue: Fixed using SET hive.exec.dynamic.partition = true;
Moving Exported Files: Instead of using hdfs dfs -getmerge, results were exported from Hue UI manually.
Incorrect Table Schema: Removed duplicate column in partitioning.

📂 Sample Input and Output
📥 Sample Input (web_server_logs.csv)

192.168.1.1,2025-02-26 12:00:00,/home,Mozilla/5.0,200
192.168.1.2,2025-02-26 12:05:00,/about,Chrome/91.0,404
192.168.1.3,2025-02-26 12:10:00,/contact,Safari/15.0,500
📤 Expected Output (query_results.csv and other exported files)
Each exported file contains results from specific Hive queries. Example:

csv
minute,request_count
2025-02-26 12:00,1
2025-02-26 12:05,1
2025-02-26 12:10,1
All query results are stored in:

query-hive-50.csv
query-hive-51.csv
query-hive-52.csv
query-hive-53.csv
query-hive-54.csv
query-hive-55.csv
query-hive-59.csv
query_results.csv
