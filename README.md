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

### 3. 


