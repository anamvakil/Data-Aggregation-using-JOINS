# Data Aggregation using Joins and Subqueries

The primary objectives of this post are to summarize the use of JOINS in SQL, assert the importance of aliases and showcase the use of Subqueries. To do that I have imported the dataset from BIGQUERY marketplace named ***International Education***. The tables I have used in this dataset are ***international education*** and ***country summary***. Let us start by looking into the schema of both tables to see which column can be used to combine both tables. 

## Use of Joins and Aliases

![image](https://github.com/user-attachments/assets/776d21cf-c14f-424d-af22-662d330aa85c)

Both the tables share a field name,```country_code```, that can be used as a primary key and a foreign key for the JOIN.

To practice the use of JOINs and explore how aliasing can help develop complex queries we will consider the below-mentioned scenarios:

***Scenario 1: JOIN that does not use any aliasing***
```
SELECT `bigquery-public-data.world_bank_intl_education.international_education`.country_name,
    `bigquery-public-data.world_bank_intl_education.country_summary`.country_code,
    `bigquery-public-data.world_bank_intl_education.international_education`.value
FROM `bigquery-public-data.world_bank_intl_education.international_education`
INNER JOIN `bigquery-public-data.world_bank_intl_education.country_summary`
ON `bigquery-public-data.world_bank_intl_education.country_summary`.country_code = `bigquery-public-data.world_bank_intl_education.international_education`.country_code
```

![image](https://github.com/user-attachments/assets/8aa01156-ec0f-42e4-a4fd-61860cc2709b)

***Scenario 2: JOIN that uses aliasing***
```
SELECT
    edu.country_name,
    summary.country_code,
    edu.value
FROM
    `bigquery-public-data.world_bank_intl_education.international_education` AS edu
INNER JOIN
    `bigquery-public-data.world_bank_intl_education.country_summary` AS summary
ON edu.country_code = summary.country_code
```

![image](https://github.com/user-attachments/assets/6166f080-5cb1-4a53-a081-a9e408b55022)

After exploring the above two scenarios, let us consider a situation where we want to know how many people were at the official age for secondary education in 2015, broken down by region. To answer this we can preview the tables to see which columns do we need.

![image](https://github.com/user-attachments/assets/5c7bf8b3-1b2b-4056-ab3a-b2ef5d73964e)

We will run the below-mentioned query to get the desired results.
```
SELECT
summary.region,/* to select region column from country_summary table*/
SUM(edu.value) secondary_edu_population /*to calculate the number of population and setting alias name*/
FROM
    `bigquery-public-data.world_bank_intl_education.international_education` AS edu /*setting alias for the table name international_education*/
INNER JOIN
    `bigquery-public-data.world_bank_intl_education.country_summary` AS summary /*setting alias for the table name country_summary*/
ON edu.country_code = summary.country_code --country_code is our key
    WHERE summary.region IS NOT NULL
    AND edu.indicator_name = 'Population of the official age for secondary education, both sexes (number)'
    AND edu.year = 2015
GROUP BY summary.region
ORDER BY secondary_edu_population DESC
```
![image](https://github.com/user-attachments/assets/c171ddd3-00b0-44f9-8951-f0e58486605c)

![image](https://github.com/user-attachments/assets/1b7f9254-9d44-40f9-ad72-8e1894edb6e1)

To explore what results would a Left Join give in the same scenario and also to see how does WHERE clause affects these results, I have used the following code:

```
SELECT
summary.region,/* to select region column from country_summary table*/
SUM(edu.value) secondary_edu_population /*to calculate the number of population and setting alias name*/
FROM
    `bigquery-public-data.world_bank_intl_education.international_education` AS edu /*setting alias for the table name international_education*/
LEFT JOIN
    `bigquery-public-data.world_bank_intl_education.country_summary` AS summary /*setting alias for the table name country_summary*/
ON edu.country_code = summary.country_code --country_code is our key
    WHERE 
    edu.indicator_name = 'Population of the official age for secondary education, both sexes (number)'
    AND edu.year = 2015
GROUP BY summary.region
ORDER BY secondary_edu_population DESC
```

![image](https://github.com/user-attachments/assets/b12a58b3-8ceb-4296-a786-62324bc7658b)

As we can see a total of eight rows were returned instead of seven. Also, one of the cells consists of NULL values. Hence it is important to use the correct JOIN to get the results. Whereas, if you do not change the WHERE clause in Line 9 you will get the following result.

![image](https://github.com/user-attachments/assets/e2997764-a846-4e2c-90fc-3d4a47a6ba01)

We will now consider another dataset from the explore pane named ***ncaa_basketball*** to learn more about LEFT JOIN which is a type of OUTER JOIN. Consider a scenario where you want to feature a sports article on NCAA basketball in the 1990s about which team mascots were winning the most from Division 1. The following query will help us get the right result:

```
SELECT
 seasons.market AS university,
 seasons.name AS team_name,
 mascots.mascot AS team_mascot,
 AVG(seasons.wins) AS avg_wins,
 AVG(seasons.losses) AS avg_losses,
 AVG(seasons.ties) AS avg_ties
FROM `bigquery-public-data.ncaa_basketball.mbb_historical_teams_seasons` AS seasons
LEFT JOIN `bigquery-public-data.ncaa_basketball.mascots` AS mascots
ON seasons.team_id = mascots.id
WHERE seasons.season BETWEEN 1990 AND 1999
 AND seasons.division = 1
 GROUP BY 1,2,3
ORDER BY avg_wins DESC, university
```
Query Results:

![image](https://github.com/user-attachments/assets/6fb37ca0-f3d9-48a5-90c0-6fa3e6a1fb20)

As we can observe under the query results, we found that the number of rows in our joined table is 320. Now, if we return the query with an INNER JOIN instead of a LEFT JOIN, how many rows does it return?

![image](https://github.com/user-attachments/assets/1c7d484e-f98b-4edf-a89e-64d5e6462b21)

The number turns out to be 317. When we want to retrieve only the rows where there is a match in both tables based on the JOIN condition, we use INNER JOIN. Whereas, when we want to retrieve all rows from the left table and the matched rows from the right table along with NULL values for non-matching rows, we use the LEFT JOIN. Hence the use of different types of JOIN can have a significant impact on our results. 

## Use of subqueries for a transportation system

To learn more about subqueries we will create three different subqueries, which will allow us to gather information such as:

### 1. The average trip duration by station

![image](https://github.com/user-attachments/assets/3736d712-726c-4248-b22d-f74cf3ce8a5f)

```
SELECT 
 subquery.start_station_id,
 subquery.avg_duration,
FROM
    (
    SELECT
        start_station_id,
        AVG(tripduration) AS avg_duration
        FROM bigquery-public-data.new_york_citibike.citibike_trips
GROUP BY start_station_id) as subquery
ORDER BY avg_duration DESC;
```

![image](https://github.com/user-attachments/assets/a1a1a52b-ec9e-4379-8252-e7f8fdf3d384)

In the above case, there are 882 lines of data that you can use to analyze and compare the average trip durations between each station.

### 2. Compare trip duration by station

We will continue to work with the average trip duration per station in this query.

```
SELECT
    starttime,
    start_station_id,
    tripduration,
    (
        SELECT ROUND(AVG(tripduration),2)
        FROM bigquery-public-data.new_york_citibike.citibike_trips
        WHERE start_station_id = outer_trips.start_station_id
    ) AS avg_duration_for_station,
    ROUND(tripduration - (
        SELECT AVG(tripduration)
        FROM bigquery-public-data.new_york_citibike.citibike_trips
        WHERE start_station_id = outer_trips.start_station_id), 2) AS difference_from_avg
FROM bigquery-public-data.new_york_citibike.citibike_trips AS outer_trips
ORDER BY difference_from_avg DESC
LIMIT 25;
```

![image](https://github.com/user-attachments/assets/5a3fffc9-04a8-4274-a1a4-01bb1162c8af)

### 3. Determine the five stations with the longest mean trip durations.

Composing a new query to filter the data to include only the trips from the five stations with the longest mean trip duration.

```
SELECT
    tripduration,
    start_station_id
FROM bigquery-public-data.new_york_citibike.citibike_trips
WHERE start_station_id IN
    (
        SELECT
            start_station_id
        FROM
        (
            SELECT
                start_station_id,
                AVG(tripduration) AS avg_duration
            FROM bigquery-public-data.new_york_citibike.citibike_trips
            GROUP BY start_station_id
        ) AS top_five
        ORDER BY avg_duration DESC
        LIMIT 5
    );
```

NOTE: 
`LINE5` uses the IN operator to filter records based on whether the start_station_id is in a list of start_station_ids produced by the subquery and if it is not there then that record is filtered out.

![image](https://github.com/user-attachments/assets/a60bb553-d8d9-45e1-bc8f-67d5b98f2e74)
