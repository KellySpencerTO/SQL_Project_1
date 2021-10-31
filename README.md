# **SQL Project**
**for Juno College of Technology Data Analytics Bootcamp, Cohort 1 (Fall 2021)**

*prepared by Kelly Spencer and Elena Froese*

This project tasked us to investigate player retention, concurrent with the one-year anniversary of a mobile game launch. We had access to four tables containing information about players, matches they played, items they could buy, and purchases they made. We needed to calculate the 30-day retention rate for each day in the year and produce a visualization of how the game performed on that metric over time. We were also asked to consider the available data and formulate another question related to retention, resulting in a second visualization. The tools available for use were SQL, specifically Google BigQuery, and a spreadsheet, in this case Google Sheets.

Our analysis of the 30-day retention rate over the course of the year showed that, after excluding players who joined in the last month for whom retention could not be determined, the retention rate for the game was remarkably stable, as seen in the chart below:

![retentionViz](https://github.com/KellySpencerTO/SQL_Project_1/blob/main/retention_viz.png?raw=true)


This is encouraging, but it would be even better if retention were increasing over time. We investigated a number of metrics at a high level to see if any appeared to impact retention. The most promising among them was winning percentage - the percentage of games won out of total games played by each player - so we took a deeper look at that. We aggregated the data into buckets in increments of 10% (ie players who won 0-10% of their games, 10-20% of their games, etc) and split each of those range buckets into players who were retained and not retained after 30 days.
