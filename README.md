# OTT_data-analysis-
Problem Statement/Hypothetical Scenario:

As the demand for streaming content continues to grow, understanding patterns in content type, release trends, and viewer-relevant attributes has become crucial for OTT platforms. This project aims to perform a comprehensive data analysis of a OTT platform to uncover insights related to content distribution, popularity, and regional presence.

The analysis focuses on answering key questions such as:
-Understanding content type distribution (Movies vs TV Shows)
-Identifying common content ratings for audience targeting
-Filtering content by release year for trend analysis
-Finding top content-producing countries for regional strategy
-Detecting long-duration content for platform optimization
-Assessing content freshness and recent additions
-Tracking content created by specific directors
-Identifying long-running TV shows for engagement strategies
-Measuring content diversity across genres
-Analyzing annual content trends in India for strategic planning
-Isolating documentaries for niche content promotion
-Detecting incomplete metadata (missing director info)
-Tracking actor-specific content output over time
-Identifying top contributing actors for a region 
-Categorizing content tone using keyword-based sentiment analysis (Good vs Bad)

1. Data base setup and table formation
```sql
CREATE DATABASE OTT;
```

```sql
CREATE TABLE ott
(
	show_id	VARCHAR(5),
	type    VARCHAR(10),
	title	VARCHAR(250),
	director VARCHAR(550),
	casts	VARCHAR(1050),
	country	VARCHAR(550),
	date_added	VARCHAR(55),
	release_year	INT,
	rating	VARCHAR(15),
	duration	VARCHAR(15),
	listed_in	VARCHAR(250),
	description VARCHAR(550)
);
```
-- OTT Data Analysis using SQL

-- 1. Count the number of Movies vs TV Shows

```sql
SELECT 
	type,
	COUNT(*)
FROM ott
GROUP BY 1
```
-- 2. Find the most common rating for movies and TV shows

```sql
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM ott
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```

-- 3. List all movies released in a specific year (e.g., 2020)

```sql
SELECT * 
FROM ott
WHERE release_year = 2020;
```

-- 4. Find the top 5 countries with the most content on OTT


```sql
SELECT * 
FROM
(
	SELECT 
		-- country,
		UNNEST(STRING_TO_ARRAY(country, ',')) as country,
		COUNT(*) as total_content
	FROM ott
	GROUP BY 1
)as t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```

-- 5. Identify the longest movie

```sql
SELECT 
	*
FROM ott
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```

-- 6. Find content added in the last 5 years

```sql
SELECT
*
FROM ott
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

-- 7. Find all the movies/TV shows by director 'Bill Condon'!

```sql
SELECT *
FROM
(

SELECT 
	*,
	UNNEST(STRING_TO_ARRAY(director, ',')) as director_name
FROM 
ott
)
WHERE 
	director_name = 'Bill Condon';
```


-- 8. List all TV shows with more than 5 seasons


```sql
SELECT *
FROM ott
WHERE 
	TYPE = 'TV Show'
	AND
	SPLIT_PART(duration, ' ', 1)::INT > 5;
```

-- 9. Count the number of content items in each genre


```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
	COUNT(*) as total_content
FROM ott
GROUP BY 1;
```

-- 10. Find each year and the average numbers of content release by United States on OTT. 
-- return top 5 year with highest avg content release !


```sql
SELECT 
	country,
	release_year,
	COUNT(show_id) as total_release,
	ROUND(
		COUNT(show_id)::numeric/
								(SELECT COUNT(show_id) FROM ott WHERE country = 'United States')::numeric * 100 
		,2
		)
		as avg_release
FROM ott
WHERE country = 'United States' 
GROUP BY country, 2
ORDER BY avg_release DESC 
LIMIT 5;
```

-- 11. List all movies that are documentaries


```sql
SELECT * FROM ott
WHERE listed_in LIKE '%Documentaries';
```


-- 12. Find all content without a director

```sql
SELECT * FROM ott
WHERE director IS NULL;
```

-- 13. Find how many movies actor 'Denzel Washington' appeared in last 10 years!


```sql
SELECT * FROM ott
WHERE 
	casts LIKE '%Denzel Washington%'
	AND 
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

-- 14. Find the top 10 actors who have appeared in the highest number of movies produced in Unites States.


```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(casts, ',')) as actor,
	COUNT(*)
FROM ott
WHERE country = 'Unites States'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```

--15:Categorize the content based on the presence of the keywords 'superhero' and '' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.


```sql
SELECT 
    category,
	TYPE,
    COUNT(*) AS content_count
FROM (
    SELECT 
		*,
        CASE 
            WHEN description ILIKE '%superhero%' THEN 'Good'
            ELSE 'Bad'
        END AS category
    FROM ott
) AS categorized_content
GROUP BY 1,2
ORDER BY 2;
```


