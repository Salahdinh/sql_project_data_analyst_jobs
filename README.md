# Data Analyst Job Market Exploration

## Project Overview

This project explores the data analyst job market, focusing on:

- Setting up a database to store and query job postings
- Analyzing top-paying data analyst positions
- Identifying key skills required for these high-paying roles
- Highlighting the most in-demand skills across all data analyst jobs

The analysis utilizes SQL queries to derive insights, helping job seekers understand market trends and skill demands.

---

#### 1. **Database Creation**
The database is created using the following command:
```sql
CREATE DATABASE sql_project;
```

---

#### 2. **Table Structure and Schema**

##### 2.1. `company_dim` - Company Information
Stores information about companies associated with job postings.
```sql
CREATE TABLE public.company_dim
(
    company_id INT PRIMARY KEY,
    name TEXT,
    link TEXT,
    link_google TEXT,
    thumbnail TEXT
);
```
- **Primary Key:** `company_id`

##### 2.2. `skills_dim` - Skills Information
Contains information on skills required for job positions.
```sql
CREATE TABLE public.skills_dim
(
    skill_id INT PRIMARY KEY,
    skills TEXT,
    type TEXT
);
```
- **Primary Key:** `skill_id`

##### 2.3. `job_postings_fact` - Job Postings Data
Fact table storing detailed information on job postings.
```sql
CREATE TABLE public.job_postings_fact
(
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
- **Primary Key:** `job_id`
- **Foreign Key:** `company_id` references `company_dim.company_id`

##### 2.4. `skills_job_dim` - Job-Skills Relationship
Associative table linking job postings to required skills.
```sql
CREATE TABLE public.skills_job_dim
(
    job_id INT,
    skill_id INT,
    PRIMARY KEY (job_id, skill_id),
    FOREIGN KEY (job_id) REFERENCES public.job_postings_fact (job_id),
    FOREIGN KEY (skill_id) REFERENCES public.skills_dim (skill_id)
);
```
- **Composite Primary Key:** `(job_id, skill_id)`
- **Foreign Keys:** `job_id`, `skill_id` reference `job_postings_fact` and `skills_dim`

#### 3. **Table Ownership and Indexing**
To improve performance and maintain proper permissions:
```sql
ALTER TABLE public.company_dim OWNER TO postgres;
ALTER TABLE public.skills_dim OWNER TO postgres;
ALTER TABLE public.job_postings_fact OWNER TO postgres;
ALTER TABLE public.skills_job_dim OWNER TO postgres;

CREATE INDEX idx_company_id ON public.job_postings_fact (company_id);
CREATE INDEX idx_skill_id ON public.skills_job_dim (skill_id);
CREATE INDEX idx_job_id ON public.skills_job_dim (job_id);
```

---

#### 4. **Data Loading Instructions**
To load data from CSV files, use the `COPY` command in PostgreSQL:
```sql
\copy company_dim FROM '[File Path]/company_dim.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',', ENCODING 'UTF8');
\copy skills_dim FROM '[File Path]/skills_dim.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',', ENCODING 'UTF8');
\copy job_postings_fact FROM '[File Path]/job_postings_fact.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',', ENCODING 'UTF8');
\copy skills_job_dim FROM '[File Path]/skills_job_dim.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',', ENCODING 'UTF8');
```

---

#### 5. **Query Examples**

##### 5.1. Top-Paying Remote Data Analyst Jobs
Retrieve details of the highest-paying remote data analyst positions:
```sql
SELECT	
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    name AS company_name
FROM
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' AND 
    job_location = 'Anywhere' AND 
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10;
```

##### ** Results**
| job_id  | job_title                               | job_location | job_schedule_type | salary_year_avg | job_posted_date     | company_name                              |
|---------|----------------------------------------|--------------|-------------------|-----------------|---------------------|-------------------------------------------|
| 226942  | Data Analyst                           | Anywhere     | Full-time         | 650,000.0       | 2023-02-20 15:13:33 | Mantys                                    |
| 547382  | Director of Analytics                  | Anywhere     | Full-time         | 336,500.0       | 2023-08-23 12:04:42 | Meta                                      |
| 552322  | Associate Director- Data Insights      | Anywhere     | Full-time         | 255,829.5       | 2023-06-18 16:03:12 | AT&T                                      |
| 99305   | Data Analyst, Marketing                | Anywhere     | Full-time         | 232,423.0       | 2023-12-05 20:00:40 | Pinterest Job Advertisements              |
| 1021647 | Data Analyst (Hybrid/Remote)           | Anywhere     | Full-time         | 217,000.0       | 2023-01-17 00:17:23 | Uclahealthcareers                         |


##### 5.2. Skills for Top-Paying Remote Data Analyst Jobs
Identify the skills required for the highest-paying remote data analyst roles:
```sql
WITH top_paying_jobs AS (
    SELECT	
        job_id,
        job_title,
        salary_year_avg,
        name AS company_name
    FROM
        job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE
        job_title_short = 'Data Analyst' AND 
        job_location = 'Anywhere' AND 
        salary_year_avg IS NOT NULL
    ORDER BY
        salary_year_avg DESC
    LIMIT 10
)

SELECT 
    top_paying_jobs.*,
    skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    salary_year_avg DESC;
```

##### **Results**
The most demanded skills for the top 10 highest paying data analyst jobs in 2023:
SQL tops the list with a strong demand count of 8.
Python is close behind with a count of 7. 
Tableau is also in high demand, with a count of 6. 
Other skills such as R, Snowflake, Pandas, and Excel exhibit varying levels of popularity.

##### 5.3. Most In-Demand Skills for Remote Data Analysts
List the most sought-after skills for remote data analysts:
```sql
SELECT 
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' 
    AND job_work_from_home = True 
GROUP BY
    skills
ORDER BY
    demand_count DESC
LIMIT 5;
```

##### **Results**
| skills  | demand_count |
|---------|--------------|
| sql     | 7291         |
| excel   | 4611         |
| python  | 4330         |
| tableau | 3745         |
| power bi| 2609         |

---

## 6. Conclusion

This project provided valuable insights into the data analyst job market by leveraging SQL to extract and analyze data from job postings. Key takeaways include:

- **Top Salaries**: The highest-paying roles offer significant earning potential, with a diverse range of job titles and companies.
- **Skill Relevance**: Mastery of SQL, Python, and data visualization tools can greatly increase earning potential.
- **Skill Demand**: Foundational skills like SQL and Excel remain crucial, while programming and data visualization tools are increasingly sought after.

For aspiring data analysts, focusing on developing these key skills can improve job prospects and open doors to high-paying, remote opportunities. Continuous learning and skill enhancement are essential for staying competitive in this field.
```
