![Netflix Data Analysis](https://raw.githubusercontent.com/obshake/netflix/main/netflix.jpg)
# Netflix Data Analysis Project

## Overview
This project provides an in-depth analysis of Netflix's extensive catalog of movies and TV shows using PostgreSQL. The analysis focuses on uncovering key trends and insights from the data to answer business-critical questions. By exploring the distribution, ratings, release years, and content origins, this project aims to offer actionable recommendations for decision-makers in the streaming industry. PostgreSQL is used for data querying and manipulation to extract meaningful insights.

## Objectives
- Examine the distribution of content types (movies vs. TV shows) to understand platform composition.
- Identify the most prevalent ratings across different content categories.
- Analyze content based on release years, countries, and durations to spot trends in global entertainment.
- Classify and categorize content based on keywords and specific criteria for deeper insights.

## Database Setup

```sql
DROP TABLE IF EXISTS netflix_data;
CREATE TABLE netflix_data (
    show_id VARCHAR(10) PRIMARY KEY,
    type VARCHAR(50),
    title VARCHAR(255),
    director VARCHAR(255),
    "cast" TEXT,
    country VARCHAR(600),
    date_added DATE,
    release_year INT,
    rating VARCHAR(10),
    duration VARCHAR(20),
    listed_in TEXT,
    description TEXT
);

COPY netflix_data
FROM 'D:\AssignMents\Sql Projects\Netflix Project\netflix_titles.csv'
DELIMITER ','
CSV HEADER;

```
## Data Analysis

**1. Count the Number of Movies and Tv shows.**
```sql
SELECT
	type,
	COUNT(*)
FROM
	netflix_data
GROUP BY
	type;
```
**2. Find the Most Common Rating for Movies and TV Shows.**


WITH rating_table AS(
	SELECT 
		type,
		rating, 
		COUNT(rating) AS rating_cnt
	FROM
		netflix_data
	GROUP BY
		1 , 2
	ORDER BY 3,1
),

	ranked_table AS(
	SELECT
		type,
		rating,
		ROW_NUMBER()OVER(PARTITION BY type ORDER BY rating_cnt DESC) AS row_num
	FROM
		rating_table
	)
SELECT
	type,rating
FROM
	ranked_table
WHERE
row_num = 1;

**3. Find the Top 5 Countries with the Most Content on Netflix.**
```sql
SELECT 
	country,
	COUNT(show_id) AS total_shows
FROM
	netflix_data
WHERE
	country IS NOT NULL
GROUP BY 
	country
ORDER BY
	total_shows DESC
LIMIT
	5;
```
**4. Identify the Longest Movie.**
```sql
SELECT
	title,
	country,
	duration
FROM
	netflix_data
WHERE
 	type = 'Movie' AND duration IS NOT NULL
ORDER BY
	SPLIT_PART(duration,' ', 1)::INT DESC
LIMIT
	1;
```
**5. Find All the Movies directed by Rajiv Chilaka.**
```sql
SELECT
	title,
	UNNEST(STRING_TO_ARRAY(director,',')) AS director_name
FROM 
	netflix_data
WHERE
	director = 'Rajiv Chilaka';
```
**6. Count the Number of Movies and TV Shows in Each Genre.**
```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(listed_in,',')) AS genre,
	COUNT(*) FILTER(WHERE type = 'Movie') AS Movie_count,
	COUNT(*) FILTER(WHERE type = 'TV Show') AS TV_Shows_count
FROM
	netflix_data
GROUP BY
	genre;
```
**7. Find Number of Content Releases in India.**
```sql
SELECT
	EXTRACT(YEAR FROM date_added) AS year,
	COUNT(*)
FROM
	netflix_data
WHERE
	country ILIKE '%india%'
GROUP BY
	year;
```
**8. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies released in India.**
```sql
SELECT 
	UNNEST(STRING_TO_ARRAY("cast", ',')) AS actor_name,
	COUNT(*) AS movie_count
FROM
	netflix_data
WHERE
	country ILIKE '%india%' AND type = 'Movie'
GROUP BY
	actor_name
ORDER BY
	movie_count DESC
LIMIT 10;
```
**9. Find Shows with Longest and shortest duration.**
```sql
WITH DurationStats AS (
    SELECT 
        title, 
        CAST(SUBSTRING(duration, 1, POSITION(' ' IN duration) - 1) AS INTEGER) AS duration_minutes
    FROM netflix_data
    WHERE duration ~ '^[0-9]+ min$'
)
(
SELECT
	'Longest Movie' AS length,
    title, 
    duration_minutes
FROM DurationStats
ORDER BY duration_minutes DESC
LIMIT 1 
)
UNION
(
SELECT
	'Shortest Movie' AS length,
    title, 
    duration_minutes
FROM DurationStats
ORDER BY duration_minutes ASC
LIMIT 1
);
```
**10. Top 5 Countries with the Most Unique genre Types.**
```sql
SELECT 
    TRIM(UNNEST(STRING_TO_ARRAY(country,',')))AS unique_country,
    COUNT(DISTINCT(STRING_TO_ARRAY(listed_in,','))) AS unique_genre
FROM 
	netflix_data
GROUP BY 
	unique_country
ORDER BY 
	unique_genre DESC
LIMIT 5;
```
**11. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 .**
```sql
SELECT
	title
FROM
	netflix_data
WHERE
	"cast" ILIKE '%salman khan%'
	AND release_year +10 >= EXTRACT(YEAR FROM CURRENT_DATE);

```
**12. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords.**
```sql
SELECT
	title,
	CASE 
		WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
		ELSE 'Good' END AS category
FROM 
	netflix_data;
```
