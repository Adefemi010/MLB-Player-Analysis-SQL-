# MLB-Player-Analysis-SQL-
For this project, I have access to a treasure trove of historical Major League Baseball player data, spanning decades of information. This dataset includes details like player statistics, schools attended, salaries, teams played for, height, weight, and more.

The primary goal of this project is to use advanced SQL querying techniques to analyze this data and uncover insights about how player statistics have evolved over time and across different teams in the league. Through this analysis, I’ll answer key questions that provide a deeper understanding of MLB player dynamics.#

### 1. In each decade, how many schools were there that produced players?

	SELECT 	FLOOR(yearID / 10) * 10 AS decade, COUNT(DISTINCT schoolID) AS num_schools
	FROM	schools
	GROUP BY decade
	ORDER BY decade;


###  2. What are the names of the top 5 schools that produced the most players?

	SELECT	 sd.name_full, COUNT(DISTINCT s.playerID) AS num_players
	FROM	 schools s LEFT JOIN school_details sd
			 ON s.schoolID = sd.schoolID
	GROUP BY s.schoolID
	ORDER BY num_players DESC
	LIMIT 	 5;

### 3. For each decade, what were the names of the top 3 schools that produced the most players? 

	WITH ds AS (SELECT	 FLOOR(s.yearID / 10) * 10 AS decade, sd.name_full, COUNT(DISTINCT s.playerID) AS num_players
				FROM	 schools s LEFT JOIN school_details sd
						 ON s.schoolID = sd.schoolID
				GROUP BY decade, s.schoolID),
	            
		 rn AS (SELECT	decade, name_full, num_players,
						ROW_NUMBER() OVER (PARTITION BY decade ORDER BY num_players DESC) AS row_num
				FROM	ds)
	            
	SELECT	decade, name_full, num_players
	FROM	rn
	WHERE	row_num <= 3
	ORDER BY decade DESC, row_num;
 
 ### 4. Return the top 20% of teams in terms of average annual spending?
 
	WITH ts AS (SELECT 	teamID, yearID, SUM(salary) AS total_spend
				FROM	salaries
				GROUP BY teamID, yearID
				ORDER BY teamID, yearID), -- ORDER BY in CTE is not needed and can be omitted
	            
		 sp AS (SELECT	teamID, AVG(total_spend) AS avg_spend,
						NTILE(5) OVER (ORDER BY AVG(total_spend) DESC) AS spend_pct
				FROM	ts
				GROUP BY teamID)
	            
	SELECT	teamID, ROUND(avg_spend / 1000000, 1) AS avg_spend_millions
	FROM	sp
	WHERE	spend_pct = 1;

 ### 5. For each team, show the cumulative sum of spending over the years?
 
	WITH ts AS (SELECT	 teamID, yearID, SUM(salary) AS total_spend
				FROM	 salaries
				GROUP BY teamID, yearID
				ORDER BY teamID, yearID) -- ORDER BY in CTE is not needed and can be omitted
	            
	SELECT	teamID, yearID,
			ROUND(SUM(total_spend) OVER (PARTITION BY teamID ORDER BY yearID) / 1000000, 1)
				AS cumulative_sum_millions
	FROM	ts;


