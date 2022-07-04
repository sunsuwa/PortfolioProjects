# PortfolioProjects
The first step of data exploration required me to create an identical table on PostgreSQL before data importation from a CSV file. 
PostgreSQL doesn’t make an automatic table. 
It only allows for importation to a matching table.
-------
CREATE TABLE covid_table(
continent VARCHAR(50),
location VARCHAR(50),
date DATE,
population BIGINT,
new_cases BIGINT,
total_deaths BIGINT)
--------
The query above, named covid_table, contains columns detailing the continent, location, date, population, new cases and total deaths due to covid-19.
I imported the data from a matching CSV file containing relevant and accurate information with the same columns.
------
	I used the same format to create a table named vaccination_table.
This new table contains relevant information regarding covid vaccination.

	
------	
		
		
	CREATE TABLE vaccination_table(
continent VARCHAR(50),
location VARCHAR(50),
date DATE,
total_vaccinations BIGINT,
people_vaccinated BIGINT,
people_fully_vaccinated BIGINT,
	new_vaccinations BIGINT)	
-------
SELECT * FROM vaccination_table
ORDER BY 1,2
----
To derive the INCIDENCE RATE, 
I had to query the new cases per population from the covid table.
----
The query below also required me to cast the data type as a float which is a decimal 
on PostgreSQL.
Casting to float data type enabled me to avoid an error when dividing the integer data types, 
which are new cases and population.
------

SELECT continent,location,date,
MAX((cast (new_cases as float))/(cast (population as float))) * 100
AS incidence_rate
FROM covid_table
WHERE continent IS NOT NULL
AND total_deaths IS NOT NULL
AND location IS NOT NULL
and date IS NOT NULL
AND new_cases IS NOT NULL
GROUP BY continent,location,date
ORDER BY incidence_rate  DESC
---
To create a view of the incidence 

CREATE VIEW incidence_rate AS
SELECT continent,location,date,
MAX((cast (new_cases as float))/(cast (population as float))) * 100
AS incidence_rate
FROM covid_table
WHERE continent IS NOT NULL
AND total_deaths IS NOT NULL
AND location IS NOT NULL
and date IS NOT NULL
AND new_cases IS NOT NULL
GROUP BY continent,location,date
ORDER BY incidence_rate  DESC
------
filtering the data by location
----
SELECT continent,location,date,population,new_cases, total_deaths, (cast (new_cases as float))/(cast (population as float)) * 100
AS incidence_rate
FROM covid_table
WHERE location IN ('Australia', 'Canada') 
AND date = '2020-12-31'
ORDER BY 3,4

----
filtering by location, I was able to draw valuable insight that by the end of 2020,
the incidence rate in Australia was significantly less than the 
the incidence rate in Canada.

----
I also explored the mortality rate of Covid in Australia.
I derived the mortality rate  by querying:
the total death by the population.
Like the incidence rate above, I cast the data type as a float.

----
MORTALITY RATE
-----
The query below indicates that as of 30th of April, 2021,
Hungary had the worst mortality rate per population at 0.2%.
----


SELECT continent, location, date,population, total_deaths,
(cast(total_deaths as float))/(cast(population as float)) * 100
AS mortality_rate
FROM covid_table
WHERE total_deaths IS NOT NULL
AND population IS NOT NULL
and continent IS NOT NULL
AND location IS NOT NULL
ORDER BY mortality_rate DESC
--
Alternatively, to also confirm Hungary as the country with the highest mortality rate
as of the time of the data,
I selected only the maximum total death figures from each country,
and the results were the same, excluding the null values.

--


SELECT location,population, MAX(total_deaths),
MAX((cast(total_deaths as FLOAT))/(cast(population as FLOAT))) * 100
AS max_mortality_rate
FROM covid_table
GROUP BY location, population
ORDER BY max_mortality_rate DESC
---
To create a view of the mortality rate
---
CREATE VIEW mortality_rate as
SELECT location,population, MAX(total_deaths),
MAX((cast(total_deaths as FLOAT))/(cast(population as FLOAT))) * 100
AS max_mortality_rate
FROM covid_table
GROUP BY location, population
ORDER BY max_mortality_rate DESC
-----
Exploring the vaccination table.
to grab the full scope of the data and arrive at valuable insight,
I joined the vaccination table to the covid table on 
location and date.

----

SELECT * FROM vaccination_table INNER JOIN covid_table ON
vaccination_table.location = covid_table.location
and
vaccination_table.date = covid_table.date

---
 To assess the total vaccination per population
 
 --
 
 SELECT covid_table.continent,covid_table.location,covid_table.date,
 covid_table.population,
 vaccination_table.new_vaccinations,
 MAX((cast(vaccination_table.new_vaccinations as FLOAT))/ 
	 (cast(covid_table.population as FLOAT))) * 100
 AS vaccination_rate
FROM vaccination_table INNER JOIN covid_table ON
vaccination_table.location = covid_table.location
and
vaccination_table.date = covid_table.date
WHERE new_vaccinations IS NOT NULL
	  GROUP BY covid_table.continent,covid_table.location,covid_table.date,
 covid_table.population,
 vaccination_table.new_vaccinations
 ORDER BY vaccination_rate DESC
---

The query above indicates that as of Late March 2021, the country of Bhutan,
had the highest vaccination rate in the World.
I verified this information by fact-checking on reliable resources.
--------
To create a view of the long query above
--------
CREATE VIEW country_vaccinations as
SELECT covid_table.continent,covid_table.location,covid_table.date,
 covid_table.population,
 vaccination_table.new_vaccinations,
 MAX((cast(vaccination_table.new_vaccinations as FLOAT))/ 
	 (cast(covid_table.population as FLOAT))) * 100
 AS vaccination_rate
FROM vaccination_table INNER JOIN covid_table ON
vaccination_table.location = covid_table.location
and
vaccination_table.date = covid_table.date
WHERE new_vaccinations IS NOT NULL
	  GROUP BY covid_table.continent,covid_table.location,covid_table.date,
 covid_table.population,
 vaccination_table.new_vaccinations
 ORDER BY vaccination_rate DESC
 
 --
 SELECT * FROM country_vaccinations
 ----
 Creating a temporary table to store the result
 ---
 CREATE TABLE country_vaccinations_rate(
 continent VARCHAR(250),
	 location VARCHAR(250),
	 date DATE,
	 population BIGINT,
	 new_vaccinations BIGINT,
	 vaccination_rate FLOAT
 )
 -- Then insert into the new table
 ---
 INSERT INTO country_vaccinations_rate(continent,location,date,
population,new_vaccinations, vaccination_rate)
 SELECT covid_table.continent,covid_table.location,covid_table.date,
 covid_table.population,
 vaccination_table.new_vaccinations,
 MAX((cast(vaccination_table.new_vaccinations as FLOAT))/ 
	 (cast(covid_table.population as FLOAT))) * 100
 AS vaccination_rate
FROM vaccination_table INNER JOIN covid_table ON
vaccination_table.location = covid_table.location
and
vaccination_table.date = covid_table.date
WHERE new_vaccinations IS NOT NULL
	  GROUP BY covid_table.continent,covid_table.location,covid_table.date,
 covid_table.population,
 vaccination_table.new_vaccinations
 ORDER BY vaccination_rate DESC
 -----
 
 SELECT * FROM country_vaccinations_rate
 
