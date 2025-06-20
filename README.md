# FF15 or Winnable? Formulating League Of Legends Win Probability using Early Game Metrics

**Author**: Alvin Hatcher (alvinhat@umich.edu)

---

## Introduction
### Motivation
League of Legends (LoL) is a popular PC game played worldwide in the 5v5 MOBA genre. In this project, we explore the many different pre-game and early-game metrics that can accurate predict match outcomes in LoL games. Our objective is to understand how different in-game factors impacts team performance in-game. Empirical insights offer fans and analysts a quantative view of optimal in-game strategy to win the game. Moreover, match outcome forecasting can help promote the video game by driving excitement among spectators, whether they be casual watchers or sportsbettors. 

In particular, we will be investigating this overarching question: *which early-game (and implicitly, pre-game) metrics determine game outcomes?*

### Raw Dataset Structure
The dataset we are using is data from professional League of Legends matches in 2022. Sourced from [OraclesElixir](https://oracleselixir.com/), the dataset spans `150588` rows and `163` columns. 

Each game is encoded as 12 rows.
<ul>
    <li> 10 for the players (filtered out in data cleaning).
    <li> 2 for teams.
</ul>

The `163` columns consists of:
<ul>
    <li>Team/player information (e.g., team names, region, gameid)</li>
    <li>Pre/post-game information (e.g., outcome, draft picks/bans, game duration)</li>
    <li>In-game metrics (e.g., kills, gold earned, deaths)</li>
</ul>

Relevant `31` columns to our stated question, in the context of only rows corresponding to teams:
<ul>
    <li>`gameid`: Unique idenfier for the game.</li>
    <li>`participantid`: Identifies players/teams given a `gameid`.
    <li>`teamname`: The name of the team. </li>
    <li>`result`: The outcome of the match (win/loss).</li>
    <li>`side`: Blue / Red</li>
    <li>`league`: The affiliated league the match was played in separated by region or tournament. (e.g. LCK, LPL, Worlds)</li>
    <li>`goldat10`: Team's total gold @ 10 minutes into the game.</li>
    <li>`xpat10`: Team's total XP @ 10 minutes into the game.</li>
    <li>`csat10`: Team's total creep score (CS) @ 10 minutes into the game.</li>
    <li>`killsat10`: Team's total kills @ 10 minutes into the game.</li>
    <li>`assistsat10`: Team's total assists @ 10 minutes into the game.</li>
    <li>`deathsat10`: Team's total deaths @ 10 minutes into the game.</li>
    <li>`golddiffat10`: Team's gold difference with respect to the enemy team's @ 10 minutes into the game.</li>
    <li>`xpdiffat10`: Team's XP difference with respect to the enemy team's @ 10 minutes into the game.</li>
    <li>`csdiffat10`: Team's CS difference with respect to the enemy team's @ 10 minutes into the game.</li>
    <li>`goldat15`: Team's total gold @ 15 minutes into the game.</li>
    <li>`xpat15`: Team's total XP @ 15 minutes into the game.</li>
    <li>`csat15`: Team's total creep score (CS) @ 15 minutes into the game.</li>
    <li>`killsat15`: Team's total kills @ 15 minutes into the game.</li>
    <li>`assistsat15`: Team's total assists @ 15 minutes into the game.</li>
    <li>`deathsat15`: Team's total deaths @ 15 minutes into the game.</li>
    <li>`golddiffat15`: Team's gold difference with respect to the enemy team's @ 15 minutes into the game.</li>
    <li>`xpdiffat15`: Team's XP difference with respect to the enemy team's @ 15 minutes into the game.</li>
    <li>`csdiffat15`: Team's CS difference with respect to the enemy team's @ 15 minutes into the game.</li>
    <li>`firstblood`: Indicates if corresponding team achieved first blood (first to kill another player).</li>
    <li>`firstdragon`: Indicates if corresponding team slained first dragon.</li>
    <li>`firstherald`: Indicates if corresponding team slained first rift herald.</li>
    <li>`firsttower`: Indicates if corresponding team was first to destroy tower.</li>
    <li>`firsttothreetowers`: Indicates if corresponding team was first to destroy 3 towers.</li>
    <li>`turretplates`: The number of turret plates destroyed by corresponding team.</li>
</ul>
---

## Cleaning and Exploratory Data Analysis


---

## Assessment of Missingness

Here's what a Markdown table looks like. Note that the code for this table was generated _automatically_ from a DataFrame, using

```py
print(counts[['Quarter', 'Count']].head().to_markdown(index=False))
```

| Quarter     |   Count |
|:------------|--------:|
| Fall 2020   |       3 |
| Winter 2021 |       2 |
| Spring 2021 |       6 |
| Summer 2021 |       4 |
| Fall 2021   |      55 |

---

## Hypothesis Testing


---