# **SQL Project**
**for Juno College of Technology Data Analytics Bootcamp, Cohort 1 (Fall 2021)**

*prepared by Kelly Spencer and Elena Froese*

This project tasked us to investigate player retention, concurrent with the one-year anniversary of a mobile game launch. We had access to four tables containing information about players, matches they played, items they could buy, and purchases they made. We needed to calculate the 30-day retention rate for each day in the year and produce a visualization of how the game performed on that metric over time. We were also asked to consider the available data and formulate another question related to retention, resulting in a second visualization. The tools available for use were SQL, specifically Google BigQuery, and a spreadsheet, in this case Google Sheets.

Our analysis of the 30-day retention rate over the course of the year showed that, after excluding players who joined in the last month for whom retention could not be determined, the retention rate for the game was remarkably stable, as evidenced by the trendline in the chart below:

![retentionViz](https://github.com/KellySpencerTO/SQL_Project_1/blob/main/retention_viz.png?raw=true)

This is encouraging, but it would be even better if retention were increasing over time.

We investigated a number of metrics at a high level to see if any appeared to impact retention. The most promising among them was winning percentage - the percentage of games won out of total games played by each player - so we took a deeper look at that. We aggregated the data into buckets in increments of 10% (ie players who won 0-10% of their games, 10-20% of their games, etc) and split each of those range buckets into players who were retained and not retained after 30 days.

![winPctViz](https://github.com/KellySpencerTO/SQL_Project_1/blob/main/win_pct_viz.png?raw=true)

From this chart we can see that players who either won or lost nearly all of their games - the 90-100% and 0-10% buckets - overwhelmingly were not retained. The trend held so that as winning percentage moved towards 50% from either more wins or more losses, more players were retained. This, combined with basic human psychology, suggested to us that players who lost all the time got frustrated and quit playing our game and that players who won all the time got bored and also quit. There appears to be a "sweet spot" where players who win about half the time feel appropriately challenged and keep playing.

If our assumptions are correct then we may be able to increase retention with a little in-game engineering. We could track players' win-loss records and provide those with poor records with opponents who are easier to beat, and those with excellent records with opponents who are harder to beat. Two possible options are to match players with actual players who appear to be close to their own skill level, or create "dumb-bots" and "smart-bots" and deploy them to play some of the matches against players in our target groups. The use of bots would have the additional advantage of not inadvertently affecting the win-loss record of other players in an attempt to influence the records of our target groups. The bots could be relied on not to quit, no matter how often they won or lost!

[You can view our SQL code here.](https://github.com/KellySpencerTO/SQL_Project_1/blob/main/queries.md)

[You can view our table for 30-day retention here.](https://docs.google.com/spreadsheets/d/1Dm69-vBBQL9hb2oH9sSDf_1s-0dof3c44qhVc_ZL2dM/edit?usp=sharing)

[You can view our table for winning percentage vs retention here.](https://docs.google.com/spreadsheets/d/1ws92aN1NKB2v0mcjKx3KvQE3ya2fdR3bHN2OLaMePGw/edit?usp=sharing)
