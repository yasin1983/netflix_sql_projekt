# Exploring Netflix Movies and TV Shows with SQL
![Logo](Netflix_Logo.jpg)
## Overview
In this project, I performed an SQL-based analysis of Netflix's movie and TV show dataset. The main purpose was to explore the data, gain useful insights, and answer relevant business questions. This README explains the project's objectives, the questions explored, the methods used, and the key findings.

## Objectives
- Examine how content is distributed between movies and TV shows.

- Determine the most frequent ratings assigned to both movies and TV shows.

- Analyze content based on release year, country of origin, and duration.

- Investigate and classify titles using specific keywords and custom criteria.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)
## Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows

```sql
SELECT 
	type, COUNT(*) as total_count
FROM dbo.netflix
GROUP BY type
```

**Objective:** Determine the distribution of content types on Netflix.

### 2. Find the Most Common Rating for Movies and TV Shows

```sql
WITH rating_counts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS total
    FROM dbo.netflix
    GROUP BY type, rating
),
ranked AS (
    SELECT 
        type,
        rating,
        total,
        RANK() OVER(PARTITION BY type ORDER BY total DESC) AS ranking
    FROM rating_counts
)
SELECT 
    type,
    rating,
    total,
    ranking
FROM ranked
WHERE ranking = 1
ORDER BY type;
```

**Objective:** Identify the most frequently occurring rating for each type of content.

### 3. List All Movies Released in a Specific Year (e.g., 1999)

```sql
SELECT *
FROM netflix
WHERE 
	type = 'Movie'
	AND
	release_year = 1999;
```

**Objective:** Retrieve all movies released in a specific year.

### 4. Find the Top 3 Countries with the Most Content on Netflix

```sql
SELECT new_country
FROM netflix
CROSS APPLY dbo.SplitString2(country, ',') )

SELECT TOP 3
	 new_country,
	COUNT(*) as total_content
FROM CountryList
GROUP BY  new_country
ORDER BY total_content DESC;
```

**Objective:** Identify the top 3 countries with the highest number of content items.

### 5. Identify the 3 Longest Movies

```sql
SELECT TOP 3 * FROM netflix
WHERE
	type = 'Movie'
	AND 
	duration = (SELECT MAX(duration) FROM netflix)
```

**Objective:** Find the 3 movies with the longest duration.

### 6. Find Content Added in the Last 4 Years

```sql
SELECT *
FROM netflix
WHERE
	TRY_CAST(date_added AS DATE) >= DATEADD(YEAR, -4, GETDATE())
```

**Objective:** Retrieve content added to Netflix in the last 4 years.

### 7. Find All Movies/TV Shows by Director 'Gary Andrews'

```sql
SELECT *
FROM netflix
WHERE 
	director LIKE '%Gary Andrews%'
```

**Objective:** List all content directed by 'Gary Andrews'.

### 8. List All TV Shows with More Than 5 Seasons

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
AND
	duration > '5 Seasons'
```

**Objective:** Identify TV shows with more than 5 seasons.

### 9. Count the Number of Content Items in Each Genre

```sql
SELECT 
    TRIM(value) as category,
    COUNT(*) as category_count
FROM netflix
    CROSS APPLY string_split(listed_in, ',')
GROUP BY TRIM(value)
ORDER BY category_count DESC
```

**Objective:** Count the number of content items in each genre.

### 10.Find each year and the average numbers of content release in Austria on netflix. 
return top 5 year with highest avg content release!

```sql

SELECT 
	YEAR(date_added) AS year_only, country, COUNT(*) as total_count_yearly,
	CAST((COUNT(*) *1.0/(SELECT COUNT(*) FROM netflix WHERE country  = 'India'))*100 AS DECIMAL(5,2) ) AS ratio 
FROM netflix 
WHERE country = 'Austria'
GROUP BY YEAR(date_added), country
```

**Objective:** Calculate and rank years by the average number of content releases by Austria.

### 11. List All Movies that are International Movies

```sql
SELECT *
FROM netflix
WHERE 
	listed_in LIKE '%International Movies%'
```

**Objective:** Retrieve all movies classified as international movies.

### 12. Find All Content Without a Director

```sql
SELECT *
FROM netflix
WHERE director IS NULL
```

**Objective:** List content that does not have a director.

### 13. Find How Many Movies Actor 'Bill Hader' Appeared in the Last 10 Years

```sql
SELECT type, cast, date_added,release_year
FROM netflix
CROSS APPLY STRING_SPLIT(cast, ',') AS actor
WHERE 
	LTRIM(RTRIM(actor.value)) = 'Bill Hader'
AND 
	TRY_CAST(date_added AS DATE) >= DATEADD(YEAR, -10, GETDATE())
AND 
	type = 'Movie'
ORDER BY date_added DESC;
```

**Objective:** Count the number of movies featuring 'Bill Hader' in the last 10 years.

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in United States

```sql
SELECT TOP 10 TRIM(actor.value) AS actor,
COUNT(*) as count
FROM netflix
CROSS APPLY STRING_SPLIT(cast, ',') AS actor
--WHERE country = 'India'
WHERE LOWER(country) LIKE '%United States%' --case insensitive search
GROUP BY TRIM(actor.value)
ORDER BY count DESC
```

**Objective:** Identify the top 10 actors with the most appearances in United States-produced movies.

### 15.Categorize Content Based on the Presence of 'kill, murder, violence, crime, blood' and 'fight, war, death, gun'  Keywords

```sql
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
              WHEN LOWER(description) LIKE '%kill%' 
              OR LOWER(description) LIKE '%murder%' 
              OR LOWER(description) LIKE '%violence%' 
              OR LOWER(description) LIKE '%crime%' 
              OR LOWER(description) LIKE '%blood%' THEN 'Very Bad'

            WHEN LOWER(description) LIKE '%fight%' 
              OR LOWER(description) LIKE '%war%' 
              OR LOWER(description) LIKE '%death%' 
              OR LOWER(description) LIKE '%gun%' THEN 'Bad'
			
			ELSE 'Good'

        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category
ORDER BY content_count DESC;
 
```

**Objective:** Categorize content as 'Very Bad' if it contains 'kill, murder, violence, crime, blood', as 'Bad' if it contains 'fight, war, death, gun' and 'Good' otherwise. Count the number of items in each category.

## Findings and Conclusion

- **Content Distribution:** The dataset showcases a wide variety of movies and TV shows, reflecting diverse genres and rating types.  
- **Audience Targeting:** Identifying the most frequent ratings offers insights into the intended audiences for Netflix content.  
- **Regional Trends:** Analysis of content by country, especially India, reveals regional patterns and production trends.  
- **Content Classification:** Using keyword-based categorization sheds light on the thematic focus and diversity of Netflix’s offerings.

Overall, this analysis offers a well-rounded perspective on Netflix’s content library and can support strategic decisions related to content planning and development.

## Author - Yasin Karadag 
This project is part of my portfolio, showcasing the MSQL skills essential for data analyst roles. Through this work, I've explored data effectively and derived meaningful insights. If you have any questions, feedback, or would like to collaborate on similar projects, feel free to reach out!
