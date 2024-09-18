# Introduction
Taking a sample data set from Luke Barousse's SQL for Data Analytics course, I applied SQL querying to explore the different dimensions of data analyst jobs to understand more about hiring companies, role compensation, and related skills. 

# Background
My drive to better understand the world of analytics led me to Luke's course material for learning SQL and other tools related to data analysis. Fortunately, this same SQL course included access to a large set of data on analyst roles and the relevant skills in today's job market. I had the chance to dive deeper on the topic and gain insight into making my own data-driven decision on what a career in data anlytics might look like, as well as the means to pursue it!

The information is drawn from Luke's [SQL Course](https://www.lukebarousse.com/sql). The education I gained here is incredibly helpful while providing me the opportunity to quantify this knowledge with certification and additional resources to use on my own.

### The course works through the following questions, drawn from the data provided:

1. What are the top-paying data anlyst jobs?
2. What skills are required for these top-paying jobs?
3. What skills are most in-demand for data analysts?
4. Which skills are associated with higher salaries?
5. What are the most optimal skills to learn?

# Tools I Used
I learned how to use the following tools to query this data, as part of the course:

- **SQL**: the primary resource in manipulating the data - writing queries which highlighted the different aspects of data analyst roles across the multiple stages of this project, each building upon the other.
- **PostGreSQL**: database management system that served to host the data I was provided through the course
- **Visual Studio Code**: the workplace for writing my SQL queries and troubleshooting errors in order to gleam the right data for answering my target questions
- **Git & GitHub**: background tools that allowed for sharing my work and analysis, providing the ability for further collaboration with friends and the "data nerd" community at large.

# The Analysis
Each stage of the project is intended to produce a query in which higher degrees of information can be gleamed from job market for "Data Analyst" roles. The goals are as follows:

### 1. Top Paying Data Analyst Jobs
I searched for and identified the 10 highest-paying roles that were quantified as data analysts, filtered by average annual salary and location (emphasis on remote availability). 
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
ORDER BY salary_year_avg DESC
LIMIT 10;
```

- **Remote Availability:** Highly-compensated remote data analyst roles are still available as of Q4 2023, even after a global return-to-office mandate many companies are implementing following the COVID-19 pandemic.
- **Wide Salary Range:** The top 3 roles by compensation are titled "Data Analyst", "Director of Analytics", and "Associate Director - Data Insights". The posted salaries of these roles, respectively, are $650,000, $336,500, and $255,829 - an incredibly wide range with Mantys (the company tied to the Data Analyst posting) paying ~93% more for an entry-to-mid-level position over a senior management role with Meta (the company tied to the Director of Analytics role).
- **Desirable Compensation:** The median annual salary, based on these 10 results, was $224,711 / year - well above the median household income in 2023 of $59,540 / year (ref. [Bureau of Labor Statistics](https://www.bls.gov/opub/ted/2024/median-weekly-earnings-of-full-time-workers-were-1145-in-the-fourth-quarter-of-2023.htm)). Data analyst roles (and related positions) are an attractive job family to pursue.

### 2. Top Paying Skills for Data Analysts
I searched for and identified the 10 most highly-compensated skills that are tied to data analyst roles.
```sql
WITH top_paying_jobs AS (
SELECT
    job_id,
    job_title,
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
ORDER BY salary_year_avg DESC
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

The results presented the following:
- **Recurring Competencies:** Every one of the highest paying jobs listed SQL with 90% listing both SQL and Python as the most commonly-included skill(s).
- **Expanded Toolkit:** Alongside coding languages, the majority of these roles listed a form of data visualization tool (Tableau, Power BI, etc.) in addition to cloud computing proficiencies (AWS, Azure).
### 3. Top Demanded Skills for Data Analysts
I searched for and identified the 5 most sought-after skills for data analysts from the perspective of hiring companies, showcasing competencies that would serve potential data analysts well when building their resumes.
```sql
SELECT
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_work_from_home = 'TRUE'
GROUP BY
    skills
ORDER BY
    demand_count DESC
LIMIT 5;
```

The results presented the following:
- **Large Margin of Demand:** SQL returned with a count of 7,291 job postings specifically listing the skill as a desired competency - a ~58% increase from the 2nd-highest ranked skill in demand, Microsoft Excel, at 4,611 posting mentions.
- **To See, Speak, and Understand:** Coding languages SQL and Python are listed alongside visualization tools Tableau, Power BI, and Excel, highlighting a demand for candidates to not only have the ability to navigate through data, but convey a story out of their findings.
### 4. Top Paying Skills Based on Salary
I searched for and identified the top 25 skills that appear in the most highly-compensated data analyst roles, highlighting both niche and commonly-known competencies as they are found in available job postings.
```sql
SELECT
    skills,
    ROUND(AVG(salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
    AND job_work_from_home = 'TRUE'
GROUP BY
    skills
ORDER BY
    avg_salary DESC
LIMIT 25;
```

The results presented the following:
- **Unique Knowledge:** While there were some recognizeable skills I had found in my earlier queries, top-paying skills appear to be those that are sub-specialties of larger SaaS entities. Specifically, the skills I had become more familiar with as a result of Luke's SQL course or related content were rare in comparison to more novel tools under machine learning and online repository software.
### 5. Most Optimal Skills to Learn
Given our previous queries, I highlighted the 25 top skills that are in highest demand and are tied to high average salaried data analyst roles. This also included a concentration on remote positions with specified salaries (no null valued records in the database).
```sql
WITH skills_demand AS (
    SELECT
        skills_dim.skill_id,
        skills_dim.skills,
        COUNT(skills_job_dim.job_id) AS demand_count
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst' 
        AND salary_year_avg IS NOT NULL
        AND job_work_from_home = True 
    GROUP BY
        skills_dim.skill_id
), 
-- Skills with high average salaries for Data Analyst roles
-- Use Query #4
average_salary AS (
    SELECT 
        skills_job_dim.skill_id,
        ROUND(AVG(job_postings_fact.salary_year_avg), 0) AS avg_salary
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst'
        AND salary_year_avg IS NOT NULL
        AND job_work_from_home = True 
    GROUP BY
        skills_job_dim.skill_id
)

SELECT 
    skills_dim.skill_id,
    skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count,
    ROUND(AVG(job_postings_fact.salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
    AND job_work_from_home = True 
GROUP BY
    skills_dim.skill_id
HAVING
    COUNT(skills_job_dim.job_id) > 10
ORDER BY
    avg_salary DESC,
    demand_count DESC
LIMIT 25;
```

The results presented the following:
- **Programming Languages in High Demand:** Python and R top the demand count for programming languages, with average annual salaries tied to them at $101,397 and $100,499, respectively.
- **Need for Database Management:** The demand counts of Oracle, SQL Server, NoSQL represent a need for analysts that can manage the inflow and retention of data within databases, utilizing a number of tools in order to do so.
- **Visualization and Intelligence:** Analyst candidates would do well to maintain proficiency in business intelligence tools like Tableau and Looker, with respective demand counts of 230 and 49 - bridging the gap in understanding between hands-on analysts and interested stakeholders would appear to be an extremely desirable and well-compensated ability in the world of business intelligence.
# What I Learned
This course and related project instilled in me a greater understanding of the techincal aspects of querying data alongside the investigative practices of an analyst. I can move forward in my journey to apply for data analyst and related roles having achieved the following:
- **Familiarity with SQL** - this coding language is foundational to my ability to manipulate raw data and gleam from it the insights necessary to pursue a career in data analytics. I can now comfortably draft my own queries using a number of different commands.
- **Database Construction** - using PostgreSQL on my own, I have the framework to build my own databases with mock or live data that can be incorporated into my own research and resulting projects.
- **Real-world Data Analysis** - utilized a real and comprehensive data set across multiple tables to convey insights that are applicable to my desire to become a data analyst, crafting a comprehensive report that outlines my own insights alongside my upgraded skillset.

# Conclusions

## Data Insights
1. **Top-Paying Data Analyst Jobs**: Remote work is largely available to both tenured and entry-level candidates, offering competitive salaries that can go well into the 6-figure range.
2. **Skills for Top-Paying Jobs**: SQL is an absolute must-have for prospective analyst work - even if it's not the most advanced skill found in desired candidates, it is definitely a foundational competency to have at your disposal to move into higher-paying roles within data analytics.
3. **Most In-Demand Skills**: Desired skill families include both coding languages and visualization tools, with SQL being the standout pick for the most sought-after competency at ~58% higher demand count than the next-highest skill, Microsoft Excel.
4. **Skills with Higher Salaries:** The highest average salaries within our data tracked with specialized or niche skills; proficiency with company or industry-specific tools would be grounds for exceptional compensation.
5. **Optimal Skills for Job Market Value:** The job market is in demand for analysts (and their related roles) to have a suite of skills inclusive of database management (Oracle, NoSQL, PostreSQL), programming languages (like SQL and Python), and data visualization (Tableau, Power BI, Looker); gaining a familiarity and mastery of multiple skillsets would postion a potential candidate to be in high demand with the ability to be extremely well-compensated.

## My Take-away
Both this course and project educated me in the means of utilizing SQL within a real-world environment to analyze data and gleam from it actionable insights. I'm immensely proud of the work I've done to both familiarize myself with a suite of skills foundational to a career path I'm pursuing as well as apply it practically to produce a tangible showcase of said abilities. I have the confidence to recognize that I am able to continue learning with the myriad resources avaialble and I'm excited to move forward with this journey.
