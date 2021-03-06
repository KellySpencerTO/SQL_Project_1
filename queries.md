```sql
/*
create a table to calculate the 30-day retention for each day in the year
*/
SELECT
    joined AS day_joined,
    COUNT (player_id) AS players_joined,
        -- # players joined on a given day
    SUM (retained_flag) AS players_retained,
        -- since flag set to 1/0 we can sum the flag = 1 to get the # players
        -- who joined on a given day who were retained
    (SUM (retained_flag) / COUNT (player_id)) AS fractional_retention
        -- 30-day retention = retained/joined
FROM
    (
/*
using the results of the nested calc below, set a flag for whether each player
was retained or not; use 1/0 so mathematical sums can be performed on the flag column
*/
    SELECT
        player_id,
        joined,
        CASE
            WHEN diff_last_vs_joined > 30 THEN 1
                -- if diff > 30 then 1/TRUE ie retained
            ELSE 0
                -- ELSE 0/FALSE ie not retained
            END
            AS retained_flag
    FROM
        (
/*
using joined player and match tables, for each player calculate
the number of days betwen their most recent match and the day they joined
*/
        SELECT
            player_info.player_id AS player_id,
            player_info.joined AS joined,
            IFNULL (MAX (matches_info.day), player_info.joined) AS last_match_day,
                -- day index of most recent match; IFNULL sets last match day =
                -- joined day where no matches played
            (MAX (matches_info.day) - player_info.joined) AS diff_last_vs_joined,
                -- days between most recent match and joining;
                -- prior IFNULL sets no matches played diff = 0
        FROM `thematic-scope-329421.game_co_data.player_info` AS player_info
        LEFT JOIN `thematic-scope-329421.game_co_data.matches_info` AS matches_info
            -- without the LEFT we lose any player who joined but never played
            ON matches_info.player_id = player_info.player_id
        GROUP BY -- required to combine fields with fields containing aggregations
            player_info.player_id,
            player_info.joined
        ) as calc_days
    ) as retained_calc
/*
and make the results ready for export to a visualization tool
*/
GROUP BY day_joined -- aggregation to one record for each day
ORDER BY day_joined ASC -- put it in date order
```
------------------
```sql
/*
create a table to analyze the relationship between win-loss record and 30-day retention

we have two calculations - 30 day retention and winning percentage - that need to be
made to feed up to the analysis, and they are not initially connected so we use a
WITH statement to create separate temporary tables to hold those results

we will exclude players who joined but did not play any games as their retention was
not affected by their experience within matches
*/
WITH
-- new temp table
    calc_days AS
    (
/*
set up a flag for 30-day retention for each player
includes players who joined but did not play for consistency with the other query in
project scope
*/
    SELECT
        player_id,
        joined,
        last_match_day,
        diff_last_vs_joined,
        CASE WHEN diff_last_vs_joined > 30 THEN 1
            -- if diff > 30 then 1/TRUE ie retained
            ELSE 0 -- ELSE 0/FALSE ie not retained
            END
            AS retained_flag
    FROM
        (
        SELECT
            player_info.player_id AS player_id,
            player_info.joined AS joined,
            IFNULL (MAX (matches_info.day), player_info.joined) AS last_match_day,
                -- day index of most recent match
                -- IFNULL sets last match day = joined where no matches played
            (MAX (matches_info.day) - player_info.joined) AS diff_last_vs_joined,
                -- days between most recent match and joining
                -- prior IFNULL sets no matches played diff = 0
        FROM `thematic-scope-329421.game_co_data.player_info` AS player_info
        LEFT JOIN `thematic-scope-329421.game_co_data.matches_info` AS matches_info
                -- without the LEFT we lose any player who joined but never played
            ON matches_info.player_id = player_info.player_id
        GROUP BY
            player_info.player_id,
            player_info.joined
        )
    ),
-- new temp table
    match_record AS
    (
 /*
calculate the total number of matches, matches won, and winning pct for each player
and include a column to allow aggregation to each 10% band
*/
    SELECT
        player_id,
        matches_played,
        matches_won,
        win_pct,
        CASE -- CASE WHEN sets win % range eg 0-10% for each player
            WHEN win_pct <= 0.1 THEN "0-10%"
            WHEN win_pct <= 0.2 THEN "10-20%"
            WHEN win_pct <= 0.3 THEN "20-30%"
            WHEN win_pct <= 0.4 THEN "30-40%"
            WHEN win_pct <= 0.5 THEN "40-50%"
            WHEN win_pct <= 0.6 THEN "50-60%"
            WHEN win_pct <= 0.7 THEN "60-70%"
            WHEN win_pct <= 0.8 THEN "70-80%"
            WHEN win_pct <= 0.9 THEN "80-90%"
            ELSE "90-100%"
            END
            AS win_pct_range
    FROM
        (
        SELECT
            player_id AS player_id,
            COUNT (match_id) AS matches_played, -- total matches played by player
            COUNTIF (outcome = 'win') as matches_won, -- total matches won by player
            (COUNTIF (outcome = 'win') / COUNT (match_id)) AS win_pct
                -- wins / total = winning pct; player_id is from matches table
                -- so there will be no count = 0 and no need to correct for DIV0
        FROM `thematic-scope-329421.game_co_data.matches_info`
        GROUP BY player_id
        )
    )
/*
join the temp tables into a table aggregated by pct range for export to Sheets
*/
SELECT
    match_record.win_pct_range AS win_pct_range,
    COUNT (match_record.player_id) AS num_players,
      -- total players with win_pct in range
    SUM (calc_days.retained_flag) AS num_retained,
      -- total players retained with win_pct in range
    SUM (calc_days.retained_flag) / COUNT (match_record.player_id) AS pct_retained,
        -- percentage of players in range who were retained after 30 days
    (1 - (SUM (calc_days.retained_flag) / COUNT (match_record.player_id))) AS pct_not_retained
        -- percentage of players in range who were not retained after 30 days
FROM
    match_record AS match_record
        -- based on matches_info only, naturally excludes joiners who never played
    JOIN calc_days AS calc_days
        -- no LEFT join: only want players who played
        ON calc_days.player_id = match_record.player_id
/*
and make the results ready for export to a visualization tool
*/
GROUP BY win_pct_range -- aggregation to one record for each pct range
ORDER BY win_pct_range ASC -- put it in range order
```
