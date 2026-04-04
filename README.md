# Spotify-Data-Analysis-Advanced-SQL-Project-and-Query-Optimization-
This project is about more than just running queries; it’s about taking a messy, flat Spotify dataset and engineering it into a high-performance relational database. I handled the entire lifecycle—from initial normalization to executing complex analytical problems that uncover how tracks and artists actually perform in the real world.

Project Category: Advanced


## Overview
This project is all about turning raw Spotify data into meaningful insights using **SQL**. I worked with a rich dataset containing details on tracks, albums, and artists, taking it through a full end-to-end process — from cleaning and normalizing messy data to running SQL queries across different levels (easy, intermediate, and advanced).
Beyond just querying, I also focused on optimizing performance to make the analysis faster and more efficient.

At its core, this project was a hands-on way to sharpen my SQL skills while uncovering valuable insights hidden within the data.

```sql
-- create table
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);
```
## Project Steps

### 1. Data Exploration
Before diving into SQL, it’s important to understand the dataset thoroughly. The dataset contains attributes such as:
- `Artist`: The performer of the track.
- `Track`: The name of the song.
- `Album`: The album to which the track belongs.
- `Album_type`: The type of album (e.g., single or album).
- Various metrics such as `danceability`, `energy`, `loudness`, `tempo`, and more.

### 2. Querying the Data
After the data was inserted, various SQL queries were written to explore and analyze the data set. Queries were categorized into **easy**, **medium**, and **advanced** levels. This shows my SQL proficiency.

#### Easy Queries
- Simple data retrieval, filtering, and basic aggregations.
  
#### Medium Queries
- More complex queries involving grouping, aggregation functions, and joins.
  
#### Advanced Queries
- Nested subqueries, window functions, CTEs, and performance optimization.

### 3. Query Optimization
In advanced stages, the focus shifts to improving query performance. Some optimization strategies include:
- **Indexing**: Adding indexes on frequently queried columns.
- **Query Execution Plan**: Using `EXPLAIN ANALYZE` to review and refine query performance.
  
---

## 15 Practice Questions

### Easy Level
1. Retrieve the names of all tracks that have more than 1 billion streams.
```sql
select * from spotify
where stream >= 1000000000
```

2. List all albums along with their respective artists.
   ``` sql
   select artist, album from Spotify
   order by 2
   ``` 
3. Get the total number of comments for tracks where `licensed = TRUE`.

   ``` sql
    select sum(comments) as 'total comments'
    from spotify
   where  licensed = 'true'
   ```
   
4. Find all tracks that belong to the album type `single`.
   ```sql
   select * from spotify
   where album_type = 'single'
   ``` 

5. Count the total number of tracks by each artist.
   ```sql
    select artist, count(*) as 'total number of songs' from Spotify
   group by 1
   order by 2
   ```

### Medium Level
1. Calculate the average danceability of tracks in each album.
   ```sql
   select album, avg(danceability) as 'danceablitiy' from spotify
   group by 1
   ```

2. Find the top 5 tracks with the highest energy values.
   ```sql
   select track, max(energy) from spotify
   group  by 1
   order by 2 desc
   limit  5
   ```
3. List all tracks along with their views and likes where `official_video = TRUE`.
   ``` sql
   select track,
    sum(views) as 'total views',
   sum(likes) as 'total likes'
   from spotify
   where official_video = 'true'
   group by 1
   ```
4. For each album, calculate the total views of all associated tracks.
   ```sql
   select album, track, sum(views) as 'total view' from spotify
   group by 1, 2; 

5. Retrieve the track names that have been streamed more on Spotify than on YouTube.
   ```sql
   select * from
   (select track,
   -- most_played_on,
   coalesce(sum(case when most_played_on = 'Youtube' then stream end),0) as stream_on_youtube,
   coalesce(sum(case when most_played_on = 'spotify' then stream end),0) as stream_on_spotify
   from spotify
   group by 1) as t1
   where stream_on_spotify > stream_on_youtube
   and stream_on_youtube <> 0
   ``` 


### Advanced Level
1. Find the top 3 most-viewed tracks for each artist using window functions.
   ```sql
   (select artist, track, sum(views) as "total view",
   dense_rank() over (partition by artist
   order by sum(views)) as "rank" from spotify
   group by 1, 2
   order by 1, 3 desc
   )
   select * from ranking_artist
   where "rank" < 3
   ```

2. Write a query to find tracks where the liveness score is above the average.
   ```sql
   select * from spotify
   where liveness > (select round(avg(liveness), 2) from spotify)
    ```

3. **Use a `WITH` clause to calculate the difference between the highest and lowest energy values for tracks in each album.**
```sql
WITH cte
AS
(SELECT 
	album,
	MAX(energy) as highest_energy,
	MIN(energy) as lowest_energery
FROM spotify
GROUP BY 1
)
SELECT 
	album,
	highest_energy - lowest_energery as energy_diff
FROM cte
ORDER BY 2 DESC
```
   
4. Find tracks where the energy-to-liveness ratio is greater than 1.2.
   ``` sql
   select * from spotify
   where energyliveness > 1.2
   ```

5. Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions.
```sql
with CTE as 
(select track, views, sum(likes) from Spotify
group by 1, 2)
select * from CTE 
order by 2
```

---

## Query Optimization Technique 

To improve query performance, I carried out the following optimization process:

- **Initial Query Performance Analysis Using `EXPLAIN`**
    - I began by analyzing the performance of a query using the `EXPLAIN` function.
    - The query retrieved tracks based on the `artist` column, and the performance metrics were as follows:
        - Execution time (E.T.): **7 ms**
        - Planning time (P.T.): **0.17 ms**
    - Below is the **screenshot** of the `EXPLAIN` result before optimization:
      ![EXPLAIN Before Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_before_index.png)

- **Index Creation on the `artist` Column**
    - To optimize the query performance, I created an index on the `artist` column. This ensures faster retrieval of rows where the artist is queried.
    - **SQL command** for creating the index:
      ```sql
      CREATE INDEX idx_artist ON spotify_tracks(artist);
      ```

- **Performance Analysis After Index Creation**
    - After creating the index, I ran the same query again and observed significant performance improvements:
        - Execution time (E.T.): **0.153 ms**
        - Planning time (P.T.): **0.152 ms**
    - Below is the **screenshot** of the `EXPLAIN` result after index creation:
      ![EXPLAIN After Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_after_index.png)

- **Graphical Performance Comparison**
    - A graph illustrating the comparison between the initial query execution time and the optimized query execution time after index creation.
    - **Graph view** shows the significant drop in both execution and planning times:
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%203.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%202.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%201.png)

This optimization shows how indexing can drastically reduce query time, improving the overall performance of our database operations in the Spotify project.

- **Conclusion:**
This project highlights my ability to handle complex SQL queries and provides solutions to real-world business problems. The approach taken here demonstrates a structured problem-solving methodology, data manipulation skills, and the ability to derive actionable insights from data.
---
---
