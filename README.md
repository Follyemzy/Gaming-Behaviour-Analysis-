
# Decoding Gaming Behavior Using PostgreSQL

## Project Overview

The primary goal of this project is to decode gaming behavior by analyzing player and level data using SQL queries. By leveraging PostgreSQL, we aim to gain insights into player performance, login behavior, and game statistics across different levels and devices.

## Objectives

1. **Data Cleaning**: Performed data cleaning by removing unnamed or unwanted columns and standardizing date names using Google Sheets.
2. **Database Creation**: Created a database named "Gaming Behavior Analysis/mentorness".
3. **Data Integrity Enhancement**: Altered the `level_details` table by adding primary key constraints to address duplicate entries in the `l_id` column.
4. **Query Writing**: Wrote SQL queries to address specific problem statements related to gaming behavior.

## Process Explanation

### 1. Table Creation and Primary Key Addition

- Created a table named `level_details` with columns `p_id`, `Dev_ID`, `start_datetime`, etc.
- Added a primary key constraint to the table on columns `P_ID`, `Dev_id`, and `start_datetime`.

### 2. SQL Queries

**Query 1**:
```sql
SELECT p_id, Dev_ID, PName, difficulty, level 
FROM level_details 
JOIN player_details ON level_details.p_id = player_details.p_id 
WHERE level = 0;
```
- **Objective**: Retrieve player details for those at level 0.

**Query 2**:
```sql
SELECT Level1_code, lives_earned, stages_crossed, AVG(Kill_Count) as avg_kill_count
FROM level_details 
JOIN player_details ON level_details.p_id = player_details.p_id 
WHERE lives_earned = 2 AND stages_crossed >= 3 
GROUP BY Level1_code, lives_earned, stages_crossed;
```
- **Objective**: Calculate the average `Kill_Count` grouped by specific attributes.

**Query 3**:
```sql
SELECT difficulty, SUM(stages_crossed) as total_stages
FROM level_details 
JOIN player_details ON level_details.p_id = player_details.p_id 
WHERE Level = 2 AND Dev_ID LIKE 'zm%' 
GROUP BY difficulty 
ORDER BY total_stages DESC;
```
- **Objective**: Calculate total `stages_crossed` for devices starting with 'zm'.

**Query 4**:
```sql
SELECT p_id, COUNT(DISTINCT start_datetime) as play_days
FROM player_details 
JOIN level_details ON player_details.p_id = level_details.p_id 
GROUP BY p_id 
HAVING COUNT(DISTINCT start_datetime) > 1;
```
- **Objective**: Find players who played on more than one day.

**Query 5**:
```sql
WITH AvgKill AS (
    SELECT AVG(kill_count) as avg_kill
    FROM level_details 
    WHERE difficulty = 'Medium'
)
SELECT p_id, level, SUM(kill_count) 
FROM level_details, AvgKill
WHERE kill_count > avg_kill 
GROUP BY p_id, level;
```
- **Objective**: Find players with kill counts greater than the average for `Medium` difficulty.

**Query 6**:
```sql
SELECT Level, SUM(lives_earned)
FROM player_details 
JOIN level_details ON player_details.p_id = level_details.p_id 
WHERE Level <> 'Level0' 
GROUP BY Level 
ORDER BY Level;
```
- **Objective**: Sum of `lives_earned` for each level excluding `Level0`.

**Query 7**:
```sql
SELECT Dev_ID, difficulty, score
FROM (
    SELECT Dev_ID, difficulty, score, ROW_NUMBER() OVER (PARTITION BY Dev_ID ORDER BY score DESC) as rank
    FROM level_details
) as ranked_scores
WHERE rank <= 3;
```
- **Objective**: Rank and select top 3 scores for each `Dev_ID`.

**Query 8**:
```sql
SELECT Dev_ID, MIN(start_datetime) as first_login
FROM level_details
GROUP BY Dev_ID;
```
- **Objective**: Find the first login time for each `Dev_ID`.

**Query 9**:
```sql
SELECT Dev_ID, score, RANK() OVER (PARTITION BY difficulty ORDER BY score DESC) as rank
FROM level_details
WHERE rank <= 5;
```
- **Objective**: Rank and select top 5 scores for each difficulty.

**Query 10**:
```sql
SELECT p_id, Dev_ID, MIN(start_datetime)
FROM level_details 
JOIN player_details ON level_details.p_id = player_details.p_id 
GROUP BY p_id, Dev_ID;
```
- **Objective**: Find the first login device for each player.

**Query 11a**:
```sql
SELECT p_id, start_datetime, SUM(kill_count) OVER (PARTITION BY p_id ORDER BY start_datetime) as cumulative_kill_count
FROM level_details;
```
- **Objective**: Calculate cumulative `kill_count` for each player.

**Query 11b**:
```sql
SELECT p_id, start_datetime, SUM(stages_crossed) OVER (PARTITION BY p_id ORDER BY start_datetime ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING) as cumulative_stages
FROM level_details;
```
- **Objective**: Calculate cumulative `stages_crossed` excluding the most recent.

**Query 13**:
```sql
WITH ScoreRank AS (
    SELECT Dev_ID, p_id, score, RANK() OVER (PARTITION BY Dev_ID ORDER BY score DESC) as rank
    FROM level_details
)
SELECT Dev_ID, p_id, score
FROM ScoreRank
WHERE rank <= 3;
```
- **Objective**: Rank and select top 3 scores for each `Dev_ID`.

**Query 14**:
```sql
SELECT p_id, PName, AVG(score) as avg_score
FROM level_details 
JOIN player_details ON level_details.p_id = player_details.p_id 
GROUP BY p_id, PName
HAVING AVG(score) > 0.5 * SUM(score);
```
- **Objective**: Find players who scored more than 50% of their total score in any game.

### 3. Stored Procedure Creation

**GetTopNHeadshots**:
```sql
CREATE OR REPLACE PROCEDURE GetTopNHeadshots(n INT)
LANGUAGE plpgsql
AS $$
BEGIN
    CREATE TEMP TABLE TopHeadshots AS
    SELECT Dev_ID, headshots_count, difficulty, ROW_NUMBER() OVER (PARTITION BY Dev_ID ORDER BY headshots_count DESC) as rank
    FROM level_details;
    
    SELECT Dev_ID, headshots_count, difficulty
    FROM TopHeadshots
    WHERE rank <= n;
END;
$$;
```
- **Objective**: Find the top `n` `headshots_count` for each `Dev_ID` using a CTE and `Row_Number()`.

## Conclusion

This comprehensive set of SQL queries provides detailed insights into various aspects of gaming behavior, including player performance, login patterns, and game statistics across different levels and devices.
