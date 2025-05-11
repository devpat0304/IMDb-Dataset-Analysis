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


