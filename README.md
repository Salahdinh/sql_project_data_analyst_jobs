# Data Analyst Job Market Exploration

## Project Overview

This project explores the data analyst job market, focusing on:

- Setting up a database to store and query job postings
- Analyzing top-paying data analyst positions
- Identifying key skills required for these high-paying roles
- Highlighting the most in-demand skills across all data analyst jobs

The analysis utilizes SQL queries to derive insights, helping job seekers understand market trends and skill demands.

---

## 1. Database Setup

### Step 1: Create Database

```sql
CREATE DATABASE sql_project;
```

### Step 2: Create Tables

The following tables were created to store data related to job postings, companies, and required skills:

#### 2.1. `company_dim` Table

```sql
CREATE TABLE public.company_dim (
    company_id INT PRIMARY KEY,
    name TEXT,
    link TEXT,
    link_google TEXT,
    thumbnail TEXT
);
```

#### 2.2. `skills_dim` Table

```sql
CREATE TABLE public.skills_dim (
    skill_id INT PRIMARY KEY,
    skills TEXT,
    type TEXT
);
```

#### 2.3. `job_postings_fact` Table

```sql
CREATE TABLE public.job_postings_fact (
    job_id INT PRIMARY KEY,
    company_id INT,
    job_title_short VARCHAR(255),
    job_title TEXT,
    job_location TEXT,
    job_via TEXT,
    job_schedule_type TEXT,
    job_work_from_home BOOLEAN,
    search_location TEXT,
    job_posted_date TIMESTAMP,
    job_no_degree_mention BOOLEAN,
    job_health_insurance BOOLEAN,
    job_country TEXT,
    salary_rate TEXT,
    salary_year_avg NUMERIC,
    salary_hour_avg NUMERIC,
    FOREIGN KEY (company_id) REFERENCES public.company_dim (company_id)
);
```

#### 2.4. `skills_job_dim` Table

```sql
CREATE TABLE public.skills_job_dim (
    job_id INT,
    skill_id INT,
    PRIMARY KEY (job_id, skill_id),
    FOREIGN KEY (job_id) REFERENCES public.job_postings_fact (job_id),
    FOREIGN KEY (skill_id) REFERENCES public.skills_dim (skill_id)
);
```

### Step 3: Indexes and Table Ownership

```sql
-- Set ownership of the tables to the postgres user
ALTER TABLE public.company_dim OWNER TO postgres;
ALTER TABLE public.skills_dim OWNER TO postgres;
ALTER TABLE public.job_postings_fact OWNER TO postgres;
ALTER TABLE public.skills_job_dim OWNER TO postgres;

-- Create indexes on foreign key columns for better performance
CREATE INDEX idx_company_id ON public.job_postings_fact (company_id);
CREATE INDEX idx_skill_id ON public.skills_job_dim (skill_id);
CREATE INDEX idx_job_id ON public.skills_job_dim (job_id);
```

---

## 2. Data Import

Data was imported from CSV files into the database using the following commands:

```sql
COPY company_dim
FROM 'C:\\Program Files\\PostgreSQL\\16\\data\\Datasets\\sql_course\\company_dim.csv'
WITH (FORMAT csv, HEADER true, DELIMITER ',', ENCODING 'UTF8');

COPY skills_dim
FROM 'C:\\Program Files\\PostgreSQL\\16\\data\\Datasets\\sql_course\\skills_dim.csv'
WITH (FORMAT csv, HEADER true, DELIMITER ',', ENCODING 'UTF8');

COPY job_postings_fact
FROM 'C:\\Program Files\\PostgreSQL\\16\\data\\Datasets\\sql_course\\job_postings_fact.csv'
WITH (FORMAT csv, HEADER true, DELIMITER ',', ENCODING 'UTF8');

COPY skills_job_dim
FROM 'C:\\Program Files\\PostgreSQL\\16\\data\\Datasets\\sql_course\\skills_job_dim.csv'
WITH (FORMAT csv, HEADER true, DELIMITER ',', ENCODING 'UTF8');
```

---

## 3. Data Analysis

### 3.1. Top-Paying Data Analyst Jobs

**Question:** What are the top-paying data analyst jobs?

**Objective:** Identify the top 10 highest-paying remote Data Analyst roles with specified salaries.

```sql
SELECT
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    name AS company_name
FROM job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE job_title_short = 'Data Analyst'
  AND job_location = 'Anywhere'
  AND salary_year_avg IS NOT NULL
ORDER BY salary_year_avg DESC
LIMIT 10;
```

**Key Findings:**
- Top salaries range from $184,000 to $650,000.
- Employers like Meta, AT&T, and SmartAsset offer some of the highest-paying positions.
- A variety of roles, including "Data Analyst" and "Director of Analytics," are represented.

### 3.2. Skills for Top-Paying Jobs

**Question:** What skills are required for the top-paying data analyst jobs?

**Objective:** Identify the skills associated with the highest-paying remote Data Analyst roles.

```sql
WITH top_paying_jobs AS (
    SELECT
        job_id,
        job_title,
        salary_year_avg,
        name AS company_name
    FROM job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE job_title_short = 'Data Analyst'
      AND job_location = 'Anywhere'
      AND salary_year_avg IS NOT NULL
    ORDER BY salary_year_avg DESC
    LIMIT 10
)
SELECT 
    top_paying_jobs.*,
    skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY salary_year_avg DESC;
```

**Key Findings:**
- Skills like SQL, Python, and Tableau are prominent among high-paying roles.
- Other valuable skills include R, Azure, Databricks, AWS, and Excel.

### 3.3. Most In-Demand Skills

**Question:** What are the most in-demand skills for data analysts?

**Objective:** Identify the top 5 skills most frequently listed in job postings for data analysts.

```sql
SELECT 
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE job_title_short = 'Data Analyst'
  AND job_work_from_home = True
GROUP BY skills
ORDER BY demand_count DESC
LIMIT 5;
```

**Key Findings:**
- **SQL** is the most demanded skill, followed by **Excel**, **Python**, **Tableau**, and **Power BI**.
- The prominence of SQL and Excel emphasizes the importance of foundational data processing and analysis skills.
- Programming and visualization tools are critical for data storytelling and effective decision-making.

---

## 4. Conclusion

This project provided valuable insights into the data analyst job market by leveraging SQL to extract and analyze data from job postings. Key takeaways include:

- **Top Salaries**: The highest-paying roles offer significant earning potential, with a diverse range of job titles and companies.
- **Skill Relevance**: Mastery of SQL, Python, and data visualization tools can greatly increase earning potential.
- **Skill Demand**: Foundational skills like SQL and Excel remain crucial, while programming and data visualization tools are increasingly sought after.

For aspiring data analysts, focusing on developing these key skills can improve job prospects and open doors to high-paying, remote opportunities. Continuous learning and skill enhancement are essential for staying competitive in this field.
```
