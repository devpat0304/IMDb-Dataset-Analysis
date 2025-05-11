# 🎬 IMDb Data Analysis – MapReduce + SQL Optimization
 <img src="readme-logo-image.png" alt="IMDb Project Banner" width="300"/>

An **integrated data analytics and engineering project** combining **Apache Hadoop MapReduce**, **Oracle SQL**, and **Python-based visualization** to extract meaningful insights from the **IMDb movie dataset**. This project was developed in **Spring 2025** as part of **CSE 4331/5331: Database Systems Models and Implementation** at **The University of Texas at Arlington** 🎓.

Quickly evolving media platforms generate vast volumes of content data — from genres and release years to user ratings and watch counts. This project simulates a real-world media analytics workflow by analyzing such data at scale, leveraging both **distributed computing** and **relational database systems** to answer a central question:

> _"What genre combinations have historically produced the highest-rated films across the last three decades?"_

## 🧠 How It Works (MapReduce + SQL Breakdown)

This project consists of two core data processing stages:

### 🗂️ Part 1 – Hadoop MapReduce (Java)

The custom `MapReduce.java` program filters and counts movies from a large IMDb dataset based on:

- **Genres**: Only considers movies with both genres from these combinations:
  - `Action` + `Thriller`
  - `Adventure` + `Drama`
  - `Comedy` + `Romance`
- **Rating**: Must have an average IMDb rating ≥ 7.0  
- **Release Year**: Bucketed by decade:
  - `[1991–2000]`
  - `[2001–2010]`
  - `[2011–2020]`

#### 📌 Map Phase
- Parses each line of the IMDb dataset.
- Filters out entries that don’t meet the above criteria.
- Emits key-value pairs:  
  **Key** → `[Decade],GenreCombo` (e.g., `[2001–2010],Comedy;Romance`)  
  **Value** → `1`

#### 📌 Reduce Phase
- Aggregates values for each unique key.
- Final output: Count of highly rated movies per genre combo per decade.

---

### 💾 Part 2 – SQL Query & Oracle EXPLAIN PLAN

We wrote an optimized SQL query on the IMDb Oracle database hosted on Omega to retrieve:

- 🎬 The **top 5 Action/Thriller movies** from 2011–2020
- With:
  - ≥150,000 votes
  - Rating ≥ 7.0
  - Lead actor/actress (ordering = 1)

#### 🧠 Query Plan Analysis
- We used `EXPLAIN PLAN` and `DBMS_XPLAN.DISPLAY()` to understand the optimizer’s execution flow.
- Oracle used:
  - **Full Table Scan** on `title_ratings`
  - **Hash Joins** and **Nested Loops**
  - **Window Sorting (RANK)** to select the top 5 efficiently
 
## 📁 Project File Descriptions

This section provides a detailed breakdown of the files used in the IMDb Data Analysis project. Each file contributes to either data processing, querying, or visualization.

<details>
  <summary><strong>🧩 MapReduce Component</strong></summary>

- `MapReduce.java`  
  This is the core Hadoop MapReduce program written in Java. It processes the raw IMDb input files to:
  - Filter movies with a rating ≥ 7.0
  - Segment them into decade ranges (1991–2000, 2001–2010, 2011–2020)
  - Group them by genre combinations
  - Output the count and average rating per genre combination

  Designed to be compiled using Hadoop tools and executed in a pseudo-distributed mode environment.
</details>

<details>
  <summary><strong>🧠 SQL Component</strong></summary>

- `IMDbDataExtractionSQL.sql`  
  Contains the SQL query used to extract and filter top-rated movies from the Oracle-hosted IMDb schema. Features:
  - Use of `GROUP BY`, `HAVING`, `ORDER BY`, and `JOIN` operations
  - Aggregation logic to compute average ratings
  - Integration with Oracle `EXPLAIN PLAN` for performance diagnostics

</details>

<details>
  <summary><strong>📊 Python Visualization Scripts</strong></summary>

- `plot1.py`  
  Generates a grouped bar chart to compare average ratings of genre combinations across decades. Reads in MapReduce output.

- `plot2.py`  
  Produces a bar graph representing SQL query results, typically for the most highly rated genre(s) overall.

- `plot3.py`  
  Displays a line chart that traces the average rating trend of selected genre combinations over three decades.

Each script is built with `matplotlib` and takes structured `.csv` or `.txt` data as input.
</details>

<details>
  <summary><strong>📄 Final Report</strong></summary>

- `Final Report.pdf`  
  A comprehensive write-up that outlines:
  - The design and structure of the MapReduce program
  - SQL query construction and explanation
  - Sample output screenshots
  - Execution analysis (including Oracle query plans)
  - Reflection on performance and tradeoffs

</details>


# 📂 IMDb Genre Analysis Project (Hadoop MapReduce + SQL)

An end-to-end data analysis project using **Hadoop MapReduce** and **Oracle SQL** on the **IMDb dataset** to extract genre trends and insights. This was developed as part of **CSE 4331/5331: Database Systems** at the University of Texas at Arlington (Spring 2025).

---

## ✅ Overview
This project is divided into two core parts:

- **Part 1**: Hadoop MapReduce – Analyze highly rated movies (rating ≥ 7.0) by genre combinations over time periods.
- **Part 2**: Oracle SQL – Run equivalent analysis and deeper queries on IMDb data stored in an RDBMS.

---

## ▼ Part 1: Hadoop MapReduce (Local Single Node)

### 1. Prerequisites
- Ubuntu (WSL or Virtual Machine)
- Java JDK 8 or 11
- Hadoop 3.3.2: [Download link](https://archive.apache.org/dist/hadoop/common/hadoop-3.3.2/hadoop-3.3.2.tar.gz)
- SSH installed and running (`sudo service ssh start`)

### 2. Hadoop Setup (Single Node)

```bash
# Extract Hadoop and move into directory
mkdir ~/hadoop_v1
cd ~/hadoop_v1
tar -xvzf hadoop-3.3.2.tar.gz
cd hadoop-3.3.2
```

- Edit `core-site.xml`, `hdfs-site.xml`, and `mapred-site.xml` as per [this guide](https://hadoop.apache.org/docs/r3.3.2/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation)
- Add DataNode and NameNode paths in `hdfs-site.xml`

```xml
<property>
  <name>dfs.namenode.name.dir</name>
  <value>file:/home/<username>/hadoop_data/namenode</value>
</property>
<property>
  <name>dfs.datanode.data.dir</name>
  <value>file:/home/<username>/hadoop_data/datanode</value>
</property>
```

### 3. Format NameNode & Start Hadoop

```bash
bin/hdfs namenode -format
sbin/start-dfs.sh
sbin/start-yarn.sh
jps  # Confirm services: NameNode, DataNode, ResourceManager, NodeManager
```

### 4. Load Dataset into HDFS

```bash
# Create input directory in HDFS
bin/hdfs dfs -mkdir /imdbInput

# Move .txt file from Windows to WSL
cp /mnt/c/Users/<your_name>/Downloads/Spring2025-Project3-IMDbData.txt ~/ 

# Upload to HDFS
bin/hdfs dfs -put ~/Spring2025-Project3-IMDbData.txt /imdbInput
```

### 5. Compile and Run Custom MapReduce Job

```bash
nano MapReduce.java  # Paste the provided code with Mapper and Reducer

mkdir -p classes
javac -d classes -cp `bin/hadoop classpath` MapReduce.java
jar cf genre.jar -C classes .
```

```bash
# Run the job
bin/hadoop jar genre.jar MapReduce /imdbInput /genreOutput
```

```bash
# View the output
bin/hdfs dfs -cat /genreOutput/part-r-00000
```

### 6. Download Output for Visualization

```bash
bin/hdfs dfs -get /genreOutput ~/genreOutput
```

Open `part-r-00000` with Excel/Google Sheets. Create bar or line charts to show genre trends.

---

## ▼ Part 2: Oracle SQL on Omega (UTA Server)

### 1. Connect to Omega

- Use PuTTY or Terminal with VPN (Ivanti Secure Client)
- Connect with NetID and password

```bash
sqlplus yournetid@omega.uta.edu
```

### 2. Explore the Schema

```sql
SELECT * FROM cat;  -- Lists accessible tables
DESC imdb00.title_basics;
DESC imdb00.title_ratings;
```

### 3. Run the SQL Query (Task 2)

Example:

```sql
SELECT tb.primarytitle, tr.averagerating
FROM imdb00.title_basics tb
JOIN imdb00.title_ratings tr ON tb.tconst = tr.tconst
WHERE tb.genres LIKE '%Comedy%'
  AND tb.genres LIKE '%Romance%'
  AND tb.startyear BETWEEN 2011 AND 2020
  AND tr.numvotes >= 150000
ORDER BY tr.averagerating DESC
FETCH FIRST 5 ROWS ONLY;
```

### 4. View the Query Plan

```sql
EXPLAIN PLAN FOR
<your_query_here>;
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

This will help you understand how Oracle executed the query:
- Full table scans vs. index usage
- Join strategies (hash, nested loop)
- Cost estimates and sorting

---
