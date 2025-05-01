## Project Introduction: Maji Ndogo Water Crisis - Data-Driven Solutions Part 1 and 2

This project is a **SQL data analytics** initiative aimed at addressing the persistent water crisis in Maji Ndogo.
The overarching goal is to transform a vast repository of data into actionable insights that will guide our solutions. 
By analysing this data, we aim to identify key trends, correlations, and root causes, which will then inform effective interventions.

![majindogo](https://github.com/user-attachments/assets/c8bafd61-c236-47a1-8c4b-84852720ff25)
---

### 1. **Getting to Know Our Data**

In this phase of the project, we focus on exploring the foundational tables and their structure to understand the data better.
By utilising a **data dictionary**, we gain a clear overview of the dataset’s tables, including their columns and descriptions.
This process ensures that we have a well-organised understanding of the data’s components.    

```sql
SELECT *
FROM md_water_services.data_dictionary;
```
**Data dictionary**
```
 | table_name  | column_name           | description                                                                    | datatype         | related_to                    |
 |-------------|-----------------------|--------------------------------------------------------------------------------|------------------|-------------------------------|
 | employee    | assigned_employee_id  | Unique ID assigned to each employee                                            | INT              | visits                        |
 | employee    | employee_name         | Name of the employee                                                           | VARCHAR(255)     |                               |
 | employee    | phone_number          | Contact number of the employee                                                 | VARCHAR(15)      |                               |
 | ...         | ...                   | ...                                                                            | ...              | ...                           |
 | visits      | record_id             | Unique ID assigned to each visit                                               | INT              | water_quality, water_source   |
 | visits      | location_id           | ID of the location visited                                                     | VARCHAR(255)     | location                      |
```
Through the application of select distinct, we were able to gain valuable insights into the diverse types of data present in Maji Ndogo. 
This approach has enhanced our understanding and will guide our further analysis.

```sql
Select distinct type_of_water_source
From water_source;
```
***different types of water sources***
|type_of_water_source|
|--------------------|
| tap_in_home        |
|...                 |
|shared_tap          |
    
Using the filter condition, we uncovered the excessive waiting time that the people of Maji Ndogo experienced while waiting for water, with more scenarios exceeding 500 minutes.

```sql
select *
from visits
where visits.time_in_queue>500;
```
```
output:
105 rows returned
```

### Assessing the Data Quality
**Data Scrutiny**
The task involved checking the quality of water sources by querying a table for records where the subject_quality_score is 10 (indicating good home taps) and a second visit was made. This revealed 218 rows of data, suggesting potential errors.
A suggestion was made to appoint an auditor for an independent data review to ensure accuracy.

```sql
SELECT 
	*
FROM md_water_services.water_quality
where subjective_quality_score=10 and visit_count = 2
```

A query was written to generate email addresses by transforming the `employee_name` field—replacing spaces with dots, converting the string to lowercase, and appending the domain `@ndogowater.gov`. This was first tested using a `SELECT` statement to confirm the format,
followed by an `UPDATE` statement to populate the email addresses in the employee table accordingly.
*output*
***_private data_***

**Pollution Issues**  
Data integrity checks revealed mislabeled water quality records showing “Clean” despite biological contamination levels exceeding 0.01. Descriptions such as “Clean Bacteria: E. coli” and “Clean Bacteria: Giardia Lamblia” were corrected to reflect accurate contamination.
Results marked as “Clean” were updated to “Contaminated: Biological” based on actual contamination values to prevent future misclassification.

```sql
--case 1-change 'Clean Bacteria: E. coli' to 'Bacteria: E. coli'
update well_pollution
set
  description='Bacteria: E. coli'
where
  description='Clean Bacteria: E. coli'

--case 2 - change 'clean Bacteria: Giardia Lamblia' to 'Bacteria: Giardia Lamblia'
update well_pollution
set description='Bacteria: Giardia Lamblia'
where
  description='clean Bacteria: Giardia Lamblia'

---case 3-update the result of clean where there is bacteria
update well_pollution
set description ='clean'
where biological>0.01 and results='clean'
```

**output**
| source_id     | date                | description                     | pollutant_ppm | biological | results |
|---------------|---------------------|----------------------------------|----------------|-------------|---------|
| AkRu06489224  | 2021-01-10 09:44:00 | Clean Bacteria: Giardia Lamblia | 0.0897904     | 38.467      | Clean   |
| KiRu25473224  | 2021-02-07 15:51:00 | Clean Bacteria: Giardia Lamblia | 0.0630094     | 24.4536     | Clean   |
| AkRu08749224  | 2021-11-19 13:37:00 | Clean Bacteria: E. coli         | 0.0888194     | 9.28085     | Clean   |

### Honouring the Workers
Employee residential data was analysed to count how many workers live in each town, highlighting the presence of staff in rural communities of Maji Ndogo. 
To recognise outstanding contributions, the top three field surveyors were identified based on the number of location visits. Their employee IDS were used to retrieve names, emails, and phone numbers to appreciate them
for their exceptional efforts during the survey.

```SQL
SELECT
	visits.assigned_employee_id,
    sum(visit_count)AS TOTAL_VISITS,
    employee.employee_name,
    employee.email,
    employee.phone_number
FROM visits
INNER JOIN employee on employee.assigned_employee_id=visits.assigned_employee_id
group by visits.assigned_employee_id;
```

### Summary: Exploring the Water Source Dataset

An analysis of the `water_source` table reveals important insights into water access across urban and rural communities. The majority of water sources (60%) are located in rural areas. The dataset allows for exploration of various aspects, including total population surveyed, distribution of source types (wells, taps, rivers), average users per source type, and the total number of users depending on each source type.
These metrics help assess access and dependency levels across different water infrastructures.
