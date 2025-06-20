# FF15 or Winnable? Formulating League Of Legends Win Probability using Early Game Metrics

**Author**: Alvin Hatcher (alvinhat@umich.edu)

---

## Introduction
### Motivation
League of Legends (LoL) is a popular PC game played worldwide in the 5v5 MOBA genre. In this project, we explore the many different pre-game and early-game metrics that can accurately predict match outcomes in LoL games. Our objective is to understand how different in-game factors impact team performance in-game. Empirical insights offer fans and analysts a quantative view of optimal in-game strategy to win the game. Moreover, match outcome forecasting can help promote the video game by driving excitement among spectators, whether they are casual watchers or invested sportsbettors. 

In particular, we will be investigating this overarching question: 
**which early-game (and implicitly, pre-game) metrics determine game outcomes?**

### Raw Dataset Structure
The dataset we are using is data from professional League of Legends matches in 2022. Sourced from [OraclesElixir](https://oracleselixir.com/), the dataset spans 150588 rows and 163 columns. 

Each game is encoded as 12 rows.
<ul>
    <li>10 for the players (filtered out in data cleaning).</li>
    <li>2 for teams.</li>
</ul>

The 163 columns consists of:
<ul>
    <li>Team/player information (e.g., team names, region, gameid)</li>
    <li>Pre/post-game information (e.g., outcome, draft picks/bans, game duration)</li>
    <li>In-game metrics (e.g., kills, gold earned, deaths)</li>
</ul>

Relevant 31 columns to our stated question, in the context of only rows corresponding to teams:
<ul>
    <li>`gameid`: Unique idenfier for the game.</li>
    <li>`participantid`: Identifies players/teams given a `gameid`.
    <li>`teamname`: The name of the team. </li>
    <li>`result`: The outcome of the match (win/loss).</li>
    <li>`side`: Blue / Red</li>
    <li>`league`: The affiliated league the match was played in separated by region or tournament (e.g. LCK, LPL, Worlds).</li>
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
# Data Cleaning
<ol>
    <li>Selected only relevant rows.
        <ol>
            <li>Discarded player-level entries (any row with `participantid not in [100, 200]`)</li>
            <li>Discarded rows featuring games with a duration less than 15 minutes to fully capture only games that actually experienced early game; after all, we can't analyze early-game metrics without an early-game.</li>
        </ol>
    </li>
    <li>Selected only relevant features shown above.
        <ul>
            <li>Dropped columns that were all `NaN` or empty. These are a result of pruned rows (e.g., `positionname`, which is only relevant to players) & unreleased in-game content (e.g., void grubs, which were not in the game in 2022).</li>
            <li>Dropped columns with repetitive information. Each `gameid` has two rows corresponding to each team; columns with the `opp` prefix were double counted by both rows for a given `gameid`.</li>
            <li>Dropped columns containing strictly metadata (e.g. 'url', 'teamid', etc.).</li>
            <li>Dropped columns related to pre-game draft phase due to patch-to-patch meta fluctuations and frequent balance changes, which makes champion pick/ban less important in observing longer-term trends.</li>
            <li>Dropped any column containing summary metrics or metrics that are only present past 20 minutes since we are only concerned with *early* game metrics.</li>
        </ul>
    </li>
    <li>Handled missing value through pruning since there were a minimal amount of erroneous data collection.
        <ul>
            <li>2 rows had `firstmidtower` be `NaN` out of 21312 rows.</li>
            <li>9 rows had `teamname` be `NaN` out of 21312 rows.</li>
            <li>88 rows had `turretplates` be `NaN` out of 21312 rows.</li>
            <li>Total of 99 rows out of 21312 were erroneous and pruned (0.46% of the dataset)</li>
        </ul>
    </li>
</ol>

<b>Dataset Overview After Refinement</b>


| teamname | result | side | league | goldat10 | xpat10 | csat10 | golddiffat10 | xpdiffat10 | csdiffat10 | killsat10 | assistsat10 | deathsat10 | goldat15 | xpat15 | csat15 | golddiffat15 | xpdiffat15 | csdiffat15 | killsat15 | assistsat15 | deathsat15 | firstblood | firstdragon | firstherald | firsttower | firstmidtower | firsttothreetowers | turretplates | kdat10 | kdat15 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BRION Challengers | 0 | Blue | LCKC | 16218.0 | 18213.0 | 322.0 | 1523.0 | 137.0 | -8.0 | 3.0 | 5.0 | 0.0 | 24806.0 | 28001.0 | 487.0 | 107.0 | -1617.0 | -23.0 | 5.0 | 10.0 | 6.0 | 1.0 | 0 | 1 | 1 | 1 | 1 | 5.0 | 3.0 | -1.0 |
| Nongshim Esports Academy | 1 | Red | LCKC | 14695.0 | 18076.0 | 330.0 | -1523.0 | -137.0 | 8.0 | 0.0 | 0.0 | 3.0 | 24699.0 | 29618.0 | 510.0 | -107.0 | 1617.0 | 23.0 | 6.0 | 18.0 | 5.0 | 0.0 | 1 | 0 | 0 | 0 | 0 | 0.0 | -3.0 | 1.0 |
| T1 Esports Academy | 0 | Blue | LCKC | 14939.0 | 17462.0 | 317.0 | -1619.0 | -1586.0 | -27.0 | 1.0 | 1.0 | 3.0 | 23522.0 | 28848.0 | 533.0 | -1763.0 | -906.0 | -22.0 | 1.0 | 1.0 | 3.0 | 0.0 | 0 | 1 | 0 | 0 | 0 | 2.0 | -2.0 | -2.0 |
| Liiv SANDBOX Youth | 1 | Red | LCKC | 16558.0 | 19048.0 | 344.0 | 1619.0 | 1586.0 | 27.0 | 3.0 | 3.0 | 1.0 | 25285.0 | 29754.0 | 555.0 | 1763.0 | 906.0 | 22.0 | 3.0 | 3.0 | 1.0 | 1.0 | 1 | 0 | 1 | 1 | 1 | 3.0 | 2.0 | 2.0 |
| KT Rolster Challengers | 1 | Blue | LCKC | 15466.0 | 19600.0 | 368.0 | -103.0 | 813.0 | 13.0 | 0.0 | 0.0 | 1.0 | 24795.0 | 31342.0 | 560.0 | 1191.0 | 2298.0 | 15.0 | 3.0 | 8.0 | 1.0 | 0.0 | 1 | 0 | 1 | 1 | 1 | 1.0 | -1.0 | 2.0 |
| Gen.G Global Academy | 0 | Red | LCKC | 15569.0 | 18787.0 | 355.0 | 103.0 | -813.0 | -13.0 | 1.0 | 1.0 | 0.0 | 23604.0 | 29044.0 | 545.0 | -1191.0 | -2298.0 | -15.0 | 1.0 | 1.0 | 3.0 | 1.0 | 0 | 1 | 0 | 0 | 0 | 4.0 | 1.0 | -2.0 |




# Univariate Analysis
---
After refining our features 
 <iframe
 src="assets/uni1.html"
 width="600"
 height="600"
 frameborder="0"
 ></iframe>

<iframe
 src="assets/uni2.html"
 width="600"
 height="500"
 frameborder="0"
 ></iframe>