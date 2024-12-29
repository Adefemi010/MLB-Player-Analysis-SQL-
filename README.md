# MLB-Player-Analysis-SQL-
For this project, I have access to a treasure trove of historical Major League Baseball player data, spanning decades of information. This dataset includes details like player statistics, schools attended, salaries, teams played for, height, weight, and more.

The primary goal of this project is to use advanced SQL querying techniques to analyze this data and uncover insights about how player statistics have evolved over time and across different teams in the league. Through this analysis, I’ll answer key questions that provide a deeper understanding of MLB player dynamics.#

### 1. In each decade, how many schools were there that produced players?

	SELECT 	FLOOR(yearID / 10) * 10 AS decade, COUNT(DISTINCT schoolID) AS num_schools
	FROM	schools
	GROUP BY decade
	ORDER BY decade;


###  2. What are the names of the top 5 schools that produced the most players?

	SELECT sd.name_full, COUNT(DISTINCT s.playerID) AS num_players
	FROM schools s #
	LEFT JOIN school_details sd
	ON s.schoolID = sd.schoolID
	GROUP BY s.schoolID
	ORDER BY num_players DESC
	LIMIT 5;

### 3. For each decade, what were the names of the top 3 schools that produced the most players? 

	WITH ds AS (SELECT FLOOR(s.yearID / 10) * 10 AS decade, sd.name_full, COUNT(DISTINCT s.playerID) AS num_players
		FROM schools s LEFT JOIN school_details sd
		ON s.schoolID = sd.schoolID
		GROUP BY decade, s.schoolID),
	rn AS (SELECT	decade, name_full, num_players,
		ROW_NUMBER() OVER (PARTITION BY decade ORDER BY num_players DESC) AS row_num
		FROM ds)
	SELECT	decade, name_full, num_players
	FROM rn
	WHERE row_num <= 3
	ORDER BY decade DESC, row_num;
 
 ### 4. Return the top 20% of teams in terms of average annual spending?
 
	WITH ts AS (SELECT teamID, yearID, SUM(salary) AS total_spend
		FROM salaries
		GROUP BY teamID, yearID
		ORDER BY teamID, yearID),
	sp AS (SELECT teamID, AVG(total_spend) AS avg_spend,
		NTILE(5) OVER (ORDER BY AVG(total_spend) DESC) AS spend_pct
		FROM ts
		GROUP BY teamID)
	SELECT teamID, ROUND(avg_spend / 1000000, 1) AS avg_spend_millions
	FROM sp
	WHERE spend_pct = 1;

 ### 5. For each team, show the cumulative sum of spending over the years?
 
	WITH ts AS (SELECT teamID, yearID, SUM(salary) AS total_spend
		FROM salaries
		GROUP BY teamID, yearID
		ORDER BY teamID, yearID) 
	SELECT	teamID, yearID,
	ROUND(SUM(total_spend) OVER (PARTITION BY teamID ORDER BY yearID) / 1000000, 1)
	AS cumulative_sum_millions
	FROM ts;
 
### 6. Return the first year that each team's cumulative spending surpassed 1 billion?

	WITH ts AS (SELECT teamID, yearID, SUM(salary) AS total_spend
			FROM salaries
			GROUP BY teamID, yearID
			ORDER BY teamID, yearID), 
            
	 cs AS (SELECT	teamID, yearID,
			SUM(total_spend) OVER (PARTITION BY teamID ORDER BY yearID)
			AS cumulative_sum
			FROM ts),
            
	 rn AS (SELECT	teamID, yearID, cumulative_sum,
			ROW_NUMBER() OVER (PARTITION BY teamID ORDER BY cumulative_sum) AS rn
			FROM cs
			WHERE cumulative_sum > 1000000000)
	SELECT	teamID, yearID, ROUND(cumulative_sum / 1000000000, 2) AS cumulative_sum_billions
	FROM	rn
	WHERE	rn = 1;


### 7. For each player, calculate their age at their first (debut) game, their last game, and their career length (all in years). Sort from longest career to shortest career.

	SELECT 	nameGiven,
	        TIMESTAMPDIFF(YEAR, CAST(CONCAT(birthYear, '-', birthMonth, '-', birthDay) AS DATE), debut)
		AS starting_age,
		TIMESTAMPDIFF(YEAR, CAST(CONCAT(birthYear, '-', birthMonth, '-', birthDay) AS DATE), finalGame)
		AS ending_age,
		TIMESTAMPDIFF(YEAR, debut, finalGame) AS career_length
	FROM players
	ORDER BY career_length DESC;

### 8. What team did each player play on for their starting and ending years? 

	SELECT 	p.nameGiven,
		s.yearID AS starting_year, s.teamID AS starting_team,
	        e.yearID AS ending_year, e.teamID AS ending_team
	FROM	players p INNER JOIN salaries s
		ON p.playerID = s.playerID
		AND YEAR(p.debut) = s.yearID
		INNER JOIN salaries e
		ON p.playerID = e.playerID
		AND YEAR(p.finalGame) = e.yearID;
 
 ### 9. How many players started and ended on the same team and also played for over a decade?
	
	SELECT 	p.nameGiven,
		s.yearID AS starting_year, s.teamID AS starting_team,
	        e.yearID AS ending_year, e.teamID AS ending_team
	FROM	players p INNER JOIN salaries s
		ON p.playerID = s.playerID
		AND YEAR(p.debut) = s.yearID
		INNER JOIN salaries e
		ON p.playerID = e.playerID
		AND YEAR(p.finalGame) = e.yearID
	WHERE	s.teamID = e.teamID AND e.yearID - s.yearID > 10;

 ### 10. Create a summary table that shows for each team, what percent of players bat right, left and both?

	SELECT	s.teamID,
		ROUND(SUM(CASE WHEN p.bats = 'R' THEN 1 ELSE 0 END) / COUNT(s.playerID) * 100, 1) AS bats_right,
	        ROUND(SUM(CASE WHEN p.bats = 'L' THEN 1 ELSE 0 END) / COUNT(s.playerID) * 100, 1) AS bats_left,
	        ROUND(SUM(CASE WHEN p.bats = 'B' THEN 1 ELSE 0 END) / COUNT(s.playerID) * 100, 1) AS bats_both
	FROM	salaries s LEFT JOIN players p
		ON s.playerID = p.playerID
	GROUP BY s.teamID;
	




 
