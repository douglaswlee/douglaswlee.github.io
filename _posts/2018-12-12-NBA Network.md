---
layout: post
title: The two-man game
---

Previously, I [wrote](https://douglaswlee.github.io/NBA-Performance/) a little about a preliminary study to understand what contributes to underperformance and overperformance by NBA teams in terms of the difference between their expected (Pythagorean) winning percentage and their actual winning percentage. One thing that I believed severely limited my efforts (among others) was the absence of lineup-level data (see below) to potentially characterize potential inefficiencies in allocation of minutes on the court between different players.

<p align="center">
  <img src="../assets/img/Lineups.png">
</p>

For example, say, the 11th lineup -- the most frequently-played negative lineup by PTS (net points per 100 possessions) for this year's current best team, the Toronto Raptors -- was a bit closer to the top lineup by minutes played (MP). That *might* be something that hinders an otherwise high-performing team from winning as much as it should.

Ultimately, I didn't explore (yet) anything related to these kinds of potential playing time/net scoring inefficiencies and their connection to team under/overperformance. But I did want to go deeper than just the five-man lineups to understand how pairs of players impact team performance while on court together as a starting point.

For a given team during a given season, I used BeautifulSoup to scrape a table of five-man lineups as above to estimate (crudely) the *Net Points Contributed* by a given pair of players across all such lineups included in the table. To do so,for  each time a unique pair of players appeared in a given row, I made sure to increment the total minutes played (MP) and net points per 100 possesions (PTS) contributed by the pair. After collecting this data, I constructed a network graph to visualize each player pair's performance, doing the following: 

<p align="center">
  <img src="../assets/img/Pace.png">
</p>

1. Scraped the "front matter" of the page for the team (see above) to collect the team's pace for the season (number of possessions played per 48 minutes)
2. Using the team's pace and lineup-level data, calculated the estimated *Net Points Contributed* for each pair of players which appeared at least once in one of the table of five-man lineups. This estimate is essentially a product of the given team's pace (converted to possessions per minute), each player pair's total minutes played and the total net points (converted to per possession).
3. Created an undirected network graph using [NetworkX](https://networkx.github.io) from the player pair *Net Points Contributed*, where:
* Each node corresponds to each individual player on the team
* Each edge reflects that a pair of players were on the court together during the season. The weight of each edge corresponds to the pair's *Net Points Contributed*. The color indicates if this quantity for the pair are negative (red) or positive (black)

