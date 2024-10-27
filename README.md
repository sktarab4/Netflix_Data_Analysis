# Netflix Movies and TV Shows Data Analysis using SQL

![](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

## Business Problems and Solutions

```sql
SELECT * FROM netflix;

--1. Count the Number of Movies vs TV Shows

SELECT type, COUNT(*) AS total_content
FROM netflix
GROUP BY type;

--2. Find the Most Common Rating for Movies and TV Shows

with rating_counts as (SELECT type, rating, COUNT(*) AS rating_count
FROM netflix
GROUP BY type, rating),

ranked_ratings as (SELECT type, rating, rating_count,
RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rnk
FROM rating_counts)

SELECT type, rating AS most_frequent_rating
FROM ranked_ratings
WHERE rnk = 1;

--3. List All Movies Released in a Specific Year (e.g., 2020)

SELECT * FROM netflix
WHERE type = 'Movie'
AND release_year = 2020

--4. Find the Top 5 Countries with the Most Content on Netflix

SELECT unnest(string_to_array(country, ', ')) AS country,
COUNT(*) AS total_content
FROM netflix
GROUP BY country
ORDER BY total_content DESC
LIMIT 5;

--5. Identify the Longest Movie

SELECT 
title,
SPLIT_PART(duration,' ',1)::INT AS duration_in_mins
FROM netflix
WHERE type = 'Movie'
AND duration IS NOT NULL
ORDER BY 2 DESC
LIMIT 1

--6. Find Content Added in the Last 5 Years

SELECT * FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'

--7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

SELECT * FROM 
(SELECT title,unnest(string_to_array(director,', ')) as director FROM netflix) temp
WHERE director = 'Rajiv Chilaka'

--8. List All TV Shows with More Than 5 Seasons

SELECT type,title,release_year,duration FROM netflix
WHERE type = 'TV Show'
and SPLIT_PART(duration, ' ',1)::INT > 5

--9. Count the Number of Content Items in Each Genre

SELECT unnest(string_to_array(listed_in,',')) as genre,
COUNT(*) as num_of_content 
FROM netflix
GROUP BY 1;

--10. List All Movies that are Documentaries

SELECT * FROM netflix
WHERE listed_in LIKE '%Documentaries';

--11. Find All Content Without a Director

SELECT * 
FROM netflix
WHERE director IS NULL;

--12. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

SELECT * FROM netflix
WHERE casts LIKE '%Salman Khan%'
AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;

--13. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;

--14. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords

with cte as (SELECT *,
CASE WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
ELSE 'Good' END AS category
FROM netflix)

SELECT category, COUNT(*) AS content_count
FROM cte
GROUP BY 1

--15. Find each year and the average numbers of content release in India on netflix.

SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;

--16. Average Duration of Movies by Rating

SELECT rating, AVG(CAST(SPLIT_PART(duration, ' ', 1) AS INTEGER)) AS avg_duration
FROM netflix
WHERE type = 'Movie' AND duration IS NOT NULL
GROUP BY rating
ORDER BY avg_duration DESC;

--17. Rank Top 3 Movies by Year Based on Duration

SELECT title, release_year, duration,
       RANK() OVER (PARTITION BY release_year ORDER BY CAST(SPLIT_PART(duration, ' ', 1) AS INTEGER) DESC) AS rank
FROM netflix
WHERE type = 'Movie'
ORDER BY release_year, rank
LIMIT 100;

--18.

```

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.
