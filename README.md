# FF15 or Winnable? Formulating League Of Legends Win Probability using Early Game Metrics

**Author**: Alvin Hatcher (alvinhat@umich.edu)

---

# Introduction

### Motivation
League of Legends (LoL) is a popular PC game played worldwide in the 5v5 MOBA genre. In this project, we explore the many different pre-game and early-game metrics that can accurately predict match outcomes in LoL games. Our objective is to understand how different in-game factors impact team performance in-game. Empirical insights offer fans and analysts a quantative view of optimal in-game strategy to win the game. Moreover, match outcome forecasting can help promote the video game by driving excitement among spectators, whether they are casual watchers or invested sportsbettors. 

In particular, we will be investigating this overarching question: 
**which early-game (and implicitly, pre-game) metrics determine game outcomes?**

### Raw Dataset Structure
The dataset we are using is data from professional League of Legends matches in 2022. Sourced from [OraclesElixir](https://oracleselixir.com/), the dataset spans 150588 rows and 163 columns. 

Each game is encoded as 12 rows.
- 10 for the players (filtered out in data cleaning).
- 2 for teams.

The 163 columns consists of:
- Team/player information (e.g., team names, region, gameid)
- Pre/post-game information (e.g., outcome, draft picks/bans, game duration)
- In-game metrics (e.g., kills, gold earned, deaths)

Relevant 31 columns to our stated question, in the context of only rows corresponding to teams:
- `gameid`: Unique idenfier for the game.
- `participantid`: Identifies players/teams given a `gameid`.
- `teamname`: The name of the team. 
- `result`: The outcome of the match (win/loss).
- `side`: Blue / Red
- `league`: The affiliated league the match was played in separated by region or tournament (e.g. LCK, LPL, Worlds).
- `goldat10`: Team's total gold @ 10 minutes into the game.
- `xpat10`: Team's total XP @ 10 minutes into the game.
- `csat10`: Team's total creep score (CS) @ 10 minutes into the game.
- `killsat10`: Team's total kills @ 10 minutes into the game.
- `assistsat10`: Team's total assists @ 10 minutes into the game.
- `deathsat10`: Team's total deaths @ 10 minutes into the game.
- `golddiffat10`: Team's gold difference with respect to the enemy team's @ 10 minutes into the game.
- `xpdiffat10`: Team's XP difference with respect to the enemy team's @ 10 minutes into the game.
- `csdiffat10`: Team's CS difference with respect to the enemy team's @ 10 minutes into the game.
- `goldat15`: Team's total gold @ 15 minutes into the game.
- `xpat15`: Team's total XP @ 15 minutes into the game.
- `csat15`: Team's total creep score (CS) @ 15 minutes into the game.
- `killsat15`: Team's total kills @ 15 minutes into the game.
- `assistsat15`: Team's total assists @ 15 minutes into the game.
- `deathsat15`: Team's total deaths @ 15 minutes into the game.
- `golddiffat15`: Team's gold difference with respect to the enemy team's @ 15 minutes into the game.
- `xpdiffat15`: Team's XP difference with respect to the enemy team's @ 15 minutes into the game.
- `csdiffat15`: Team's CS difference with respect to the enemy team's @ 15 minutes into the game.
- `firstblood`: Indicates if corresponding team achieved first blood (first to kill another player).
- `firstdragon`: Indicates if corresponding team slained first dragon.
- `firstherald`: Indicates if corresponding team slained first rift herald.
- `firsttower`: Indicates if corresponding team was first to destroy tower.
- `firsttothreetowers`: Indicates if corresponding team was first to destroy 3 towers.
- `turretplates`: The number of turret plates destroyed by corresponding team.

---

# Cleaning and Exploratory Data Analysis

### Data Cleaning

- Selected only relevant rows.
    - Discarded player-level entries (any row with `participantid not in [100, 200]`)
    - Discarded rows featuring games with a duration less than 15 minutes to fully capture only games that actually experienced early game; after all, we can't analyze early-game metrics without an early-game.
- Selected only relevant features shown above.
    - Dropped columns that were all `NaN` or empty. These are a result of pruned rows (e.g., `positionname`, which is only relevant to players) & unreleased in-game content (e.g., void grubs, which were not in the game in 2022).
    - Dropped columns with repetitive information. Each `gameid` has two rows corresponding to team blue/red; columns with the `opp` prefix were double counted by both rows for a given `gameid`.
    - Dropped columns containing strictly metadata (e.g. `url`, `teamid`, etc.).
    - Dropped columns related to pre-game draft phase due to patch-to-patch meta fluctuations and frequent balance changes, which makes champion pick/ban less important in observing longer-term trends.
    - Dropped any column containing summary metrics or metrics that are only present past 20 minutes since we are only concerned with *early* game metrics.
- Handled missing value through pruning due to minimal amount of erronenous data.
    - 2 rows had `firstmidtower` be `NaN` out of 21312 rows.
    - 9 rows had `teamname` be `NaN` out of 21312 rows.
    - 88 rows had `turretplates` be `NaN` out of 21312 rows.
    - Total of 99 rows out of 21312 were erroneous and thus pruned (0.46% of the dataset)

### Dataset Overview After Refinement

<iframe src="assets/cleaned.html" class='table-wrapper' width="800" height="400" frameborder="0">
</iframe>

## Univariate Analysis

After refining our features, we can finally do analysis. The plot below shows the *distribution of CS/min @ 10 minutes* for each team within the dataset. We generated this plot by taking the `csat10` feature and dividing it by 10 to give us CS/min, an often mentioned metric in LoL gameplay. High CS/min is a sought after metric by many players as CS directly translates to a gain in gold and XP as well.

<iframe src="assets/uni1.html" width="900" height="450" frameborder="0"></iframe>

 Team CS/min seem to cluster around `30-35` CS/min. This averages to around `7.5-8.5` CS/min for each non-support player (since supports don't farm), showcasing prioritization of early, consistent farming in professional play. Outliers in the distribution highlights varying game dynamics; low early CS/min likely corresponds to games with a lot of early skirmishes or games where one team is dominated in lane, whereas high CS games indicate high lane dominance or a relatively low action early game.

 Below we have another distribution plot, but this time, it is the *distribution of kills @ 10 minutes*.

<iframe src="assets/uni2.html" width="900" height="450" frameborder="0"></iframe>

We can see that a small fraction of games see 4+ kills, and beyond 7 kills is extremely rare. This shows that early-game aggression is relatively muted in professional play and that teams rarely collect more than 3 kills in laning phase. A high kill game early on by a team potentially signals a very dominant early game and possible snowball, but for the most part, teams will "scale" (get gold -> buy items and get XP -> level up) through farming in the early stages.

## Bivariate Analysis

Below is the *distribution of gold difference among game winners vs. losers*. I suspect gold difference would be a powerful predictive early-game metric.

<iframe src="assets/bivar1.html" width="900" height="450" frameborder="0"></iframe>

As expected, there's a clear relationship between positive gold difference and winning, indicating that teams with a gold lead are more likely to win the game. However, there are small overlaps between the two boxes and the handful of extreme outliers showcases that winning is more than just early gold.
