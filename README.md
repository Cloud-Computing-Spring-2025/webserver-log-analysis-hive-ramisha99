
# Web Server Log Analysis using Apache Hive

## 📌 Project Overview
This project analyzes web server logs using **Apache Hive** to extract insights such as request frequencies, most visited URLs, failed requests, and user agent distributions. The data is stored in HDFS, and Hive queries are executed to process and retrieve meaningful analytics.

## 🛠️ Implementation Approach
The analysis is conducted through Hive queries on partitioned datasets. Key queries performed:
- **Data Ingestion**: Loading web server logs into Hive tables.
- **Partitioning**: Organizing data based on HTTP response status.
- **Aggregations & Analysis**: Finding request patterns, user agents, top URLs, and failure rates.

## 🔍 Key Queries Used
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

SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;

INSERT INTO TABLE web_server_logs_partitioned PARTITION (status)
SELECT ip, `timestamp`, url, user_agent, status FROM web_server_logs;

SELECT SUBSTRING(`timestamp`, 1, 16) AS minute, COUNT(*) AS request_count
FROM web_server_logs
GROUP BY SUBSTRING(`timestamp`, 1, 16)
ORDER BY minute;

SELECT ip, COUNT(*) AS failed_requests
FROM web_server_logs
WHERE status IN (404, 500)
GROUP BY ip
HAVING failed_requests > 3;

SELECT user_agent, COUNT(*) AS count 
FROM web_server_logs 
GROUP BY user_agent 
ORDER BY count DESC;

SELECT url, COUNT(*) AS visit_count
FROM web_server_logs
GROUP BY url
ORDER BY visit_count DESC
LIMIT 3;

SELECT COUNT(*) AS total_requests FROM web_server_logs;

##Execution Steps

docker-compose up -d hive-metastore
docker start hive-server
docker exec -it namenode bash

hdfs dfs -ls /user/hive/output/
hdfs dfs -getmerge /user/hive/output/query_results query_results.csv
docker cp namenode:/query_results.csv ./query_results.csv

⚠️ Challenges Faced
Hive Partitioning Issue: Fixed using SET hive.exec.dynamic.partition = true;
Moving Exported Files: Used hdfs dfs -getmerge and docker cp to move files.
Incorrect Table Schema: Removed duplicate column in partitioning.
📂 Sample Input and Output
📥 Sample Input (web_server_logs.csv)
csv
Copy
Edit
192.168.1.1,2025-02-26 12:00:00,/home,Mozilla/5.0,200
192.168.1.2,2025-02-26 12:05:00,/about,Chrome/91.0,404
192.168.1.3,2025-02-26 12:10:00,/contact,Safari/15.0,500
📤 Expected Output (query_results.csv)
c
Copy
Edit
minute,request_count
2025-02-26 12:00,1
2025-02-26 12:05,1
2025-02-26 12:10,1