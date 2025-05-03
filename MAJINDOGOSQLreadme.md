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

**FILLING IN NULL EMAIL COLUMN**  

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
-- IDENTIFY THE TOP WORKERS BASED ON THE NUMBER OF VISITS MADE
SELECT
	visits.assigned_employee_id,
    sum(visit_count)AS TOTAL_VISITS,
    employee.employee_name,
    employee.email,
    employee.phone_number
FROM visits
INNER JOIN employee on employee.assigned_employee_id=visits.assigned_employee_id
group by visits, assigned_employee_id;
```

### Data Manipulation of the Water Source Dataset 

The analysis of the `water_source` table reveals important insights into water access across urban and rural communities. Most water sources (60%) are located in rural areas. The dataset allows for exploration of various aspects, including the total population surveyed, the distribution of source types (wells, taps, rivers), the average users per source type, and the total number of users depending on each source type.
These metrics help assess access and dependency levels across different water infrastructure.s

```sql
-- How many people did we survey
select 
	sum(number_of_people_served)as sample_population
from water_source;
```
**_output_**
![sample population](https://github.com/user-attachments/assets/13c7e71c-aa6c-4a22-8dee-1412e21b764a)

```sql
--How many wells, rivers and taps are there?
select
	type_of_water_source,
	count(source_id)
from water_source
group by type_of_water_source;
```
**_output_**![sum of water sources](https://github.com/user-attachments/assets/efc37a7c-c5d5-4e9c-afeb-1ee851755f25)


```sql
--How many people share particular types of water sources on average?
select
	type_of_water_source,
	avg(number_of_people_served)as number_of_people
from water_source
group by type_of_water_source;

--output
|type_of_water_source|number_of_people|
|................... |................|
|river               |699.1844        |
|shared_tap          |2071.3147       |
|...                 |...             |
|tap_in_home_broken  |648.8593        |
|well                | 278.5321       |
```
In the tap_in_home analysis, it indicates that 644 individuals are sharing the tap. This seems inconsistent with the summary, which reports that, on average, only 6 people reside in a home. This discrepancy suggests a need for further investigation into the data collection methods or possible errors in reporting the number of individuals associated with each tap

```sql
--How many people are getting water from each type of source
select
	type_of_water_source,
	round(((sum(number_of_people_served)/'27628140')*100),0)as number_of_people--%of total people getting water rounded to 0 decimal places
from water_source
group by type_of_water_source;

--output
| type_of_water_source  |number_of_people |
|.......................|.................|
|tap_in_home            | 17              |
|tap_in_home_broken     | 14              |
|...                    |...              |
|shared_tap             | 43              |
|river                  | 9               |
```
Based on the number_of_people column, the highest percentage is for the shared tap, which accounts for 43%, while the lowest percentage is for the river, at just 9%..

### RANKING THE WATER SOURCE TO AMEND BASED ON THE NUMBER OF PEOPLE SERVED
To refine the analysis of water source effectiveness, tap_in_home was excluded from the rankings since it represents the highest standard of access and requires no further improvement. A window function was applied to rank the remaining water source types based on the total number of people served, enabling a clear prioritisation of sources that may need attention or investment.


```SQL
SELECT 
	*,
    row_number() over(order by number_of_people_served desc)as ranking
FROM md_water_services.water_source;
```
![RANKING](https://github.com/user-attachments/assets/de0a8747-8ab7-4bf5-9501-e1b16e57b783)

The ranking query for the column indicates the position  of investments or restructuring initiatives.

### ANALYSING QUEUES
The visits table was explored to analyse water access challenges in Maji Ndogo, focusing on queue durations and survey timing. Key insights included calculating the total survey period using DateTime functions, determining the average total queue time, and comparing queue patterns across different days of the week. The time_of_record column enabled aggregation by day and hour to better understand peak usage periods, ultimately helping to assess efficiency and inform strategies for reducing waiting times at water sources.

```SQL
select
	time_format(time_of_record,'%H:00'),--Format the time_of_record to H:00 FORMAT
    round(AVG(nullif(time_in_queue,0)),0)AS Average
from visits
group by time_format(time_of_record,'%H:00')
order by time_format(time_of_record,'%H:00') ;
```
the output shows that morning period,🕕6:00am-🕗8:00am,and the evening period,🕔17:00-🕖19:00,as the longest queues time

*pivot table mimicry*
A detailed breakdown of water queue times was created using SQL to mimic a pivot table. The analysis used the `hour` extracted from the `time_of_record` column as rows and each day of the week as separate columns. The `CASE` function was applied to conditionally aggregate queue times by day, enabling a clear hour-by-day view of when queues are longest. This method revealed peak water collection periods—particularly mornings and evenings—highlighting patterns shaped by people’s daily routines.

```SQL
--CREATING A MIMIC TABLE
select
	time_format(time_of_record,'%H:00') as Hour_of_Day,
    -- sunday
    round(avg(
		case
        when dayname(time_of_record)='sunday'then time_in_queue
        else null
	end
		),0) as Sunday,
	-- Monday
    round(avg(
		case
        when dayname(time_of_record)='Monday'then time_in_queue
        else null
	end
		),0) as Monday,
	-- tuesday
    round(avg(
    case 
    when dayname(time_of_record)='Tuesday'then time_in_queue
    else null
end 
	),0) as Tuesday,
    -- wednesday
    round(avg(
    case
    when dayname(time_of_record)='Wednesday'then time_in_queue
    else null
end
	),0) as Wednesday,
-- thursday
	round(avg(
    case 
    when dayname(time_of_record)='thursday' then time_in_queue
    else null
end
	),0) as Thursday,
-- Friday
round(avg(
	Case
    when dayname(time_of_record)='Friday' then time_in_queue
    else null
end
	),0) as Friday,
-- saturday
round(avg(
	case
    when dayname(time_of_record)='Saturday'then time_in_queue
    else null
end 
	),0) as Saturday
from visits
where time_in_queue != 0 -- exclude time_queue that are zero
group by Hour_of_Day
order by Hour_of_Day;
```
## insights
1.![rankinbaby](https://github.com/user-attachments/assets/66d8b79f-acba-40ca-9fc3-9312faa854e8)  
2. The total days taken to survey were 924 day5 years)  
3. The shared taps are recommended as a first-rank investment repair.  
4. 60% of the water sources are in rural areas.
