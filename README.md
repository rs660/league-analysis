# FF15 or Winnable? Formulating League Of Legends Win Probability using Early Game Metrics

**Author**: Alvin Hatcher (alvinhat@umich.edu)

---

# 👋 Introduction 👋

### Motivation
League of Legends (LoL) is a popular PC game played worldwide in the 5v5 MOBA genre. In this project, we analyze the many different pre/early-game performance metrics in LoL games and their impact. Our ultimate objective is to find a combination of in-game factors that help us accurately predict game outcomes. These empirical insights offer fans and analysts a quantative view of optimal in-game strategy to win the game. Moreover, match outcome forecasting can help promote the video game by driving excitement among spectators, whether they are casual watchers or invested sportsbettors. 

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

The relevant 31 columns to our stated question, in the context of only rows corresponding to teams:
- `gameid`: Unique idenfier for the game.
- `participantid`: Identifies players/teams given a `gameid`.
- `teamname`: The name of the team. 
- `result`: The outcome of the match (win/loss).
- `side`: Blue / Red.
- `league`: The affiliated league the match was played; either separated by region or tournament (e.g. LCK, LPL, Worlds).
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

# 🔎 Cleaning & Exploratory Data Analysis 🔎

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

### Univariate Analysis

After refining our features, we can finally do analysis. The plot below shows the *distribution of CS/min @ 10 minutes* for each team. High CS/min is a sought after metric by many players as CS directly translates to a gain in gold and XP as well.

<iframe src="assets/uni1.html" width="900" height="450" frameborder="0"></iframe>

 Team CS/min seem to cluster around `30-35` CS/min. This averages to around `7.5-8.5` CS/min for each non-support player (since supports don't farm), showcasing prioritization of early, consistent farming in professional play. Outliers in the distribution highlights varying game dynamics. Low early CS/min likely corresponds to games with a lot of early skirmishes or games where one team is dominated in lane; high CS games indicate high lane dominance or a relatively low action early game.

 Below we have another distribution plot, but this time, it is the *distribution of kills @ 10 minutes*.

<iframe src="assets/uni2.html" width="900" height="450" frameborder="0"></iframe>

We can see that a small fraction of games see 4+ kills, and beyond 7 kills is extremely rare. This shows that early-game aggression is relatively muted in professional play and that teams rarely collect more than 3 kills in laning phase. A high kill game early on by a team potentially signals a very dominant early game and possible snowball, but for the most part, teams will "scale" (get gold -> buy items and get XP -> level up) through farming in the early stages.

### Bivariate & Aggregate Feature Analysis

Below is the *distribution of gold difference among game winners vs. losers*. I suspect gold difference would be a powerful predictive early-game metric, as gold is arguably the most important resource in the game.

<iframe src="assets/bivar1.html" width="900" height="450" frameborder="0"></iframe>

As expected, there's a clear relationship between positive gold difference and winning, indicating that teams with a gold lead are more likely to win the game. However, the visible overlaps between the two boxes' upper/lower quartiles and the handful of extreme outliers highlight that winning is more than just getting an early gold lead. Nonetheless, since gold is so seemingly important, let's look at other features' relationship with gold difference. 

<iframe src="assets/bivar2.html" width="900" height="450" frameborder="0"></iframe>

Here, we can see XP is also moderately with wins. Which shouldn't be a surprised at all since securing gold usually comes with XP as well (CS, kills, etc.).

Using my own heuristic, another metric that would lead to wins would be map/objective control. Although less quantifiable, securing early river objectives is by definition having control of the map and the objectives in it. Below is an attempt of illustrating map control in terms of early control of river objectives: a pivot table of winrates for teams that secure first river objectives.

|                |  No 1st Herald |  1st Herald |
| -------------- | -------------- | ----------- |
|  No 1st Dragon |   0.323357     |   0.504889  |
|    1st Dragon  |   0.495114     |   0.676933  |

As shown above, securing both river objectives correlates with a positive win rate. Although securing first herald seem to have a slight edge over securing first dragon. This is likely due to the mechanics of the herald (being able to take down towers and plating) which can facilitate early snowballs. However, the utility of herald falls off after its use, whereas the stat-boosts obtained from slaining dragon remains the entire game. So I suspect the longer the game lasts, the more useful first dragon as a predictive metric becomes.

---

# ❓ Framing a Prediction Problem ❓

Our objective now is to **PREDICT** a team's win probability based on early-game metrics (pre-15 minutes). We will use knowledge of the features gained in the analysis above in order to build our build our prediction model.
- Prediction Task Type: Non-binary Classification
- Response Variable: win probability `[0, 1]`
    - Motivation: win probability is linked to a lot of predictive tasks not just limited to League of Legends. For example, it has wide-ranging applications in betting and gambling.
- Evaluation Metric: ROC-AUC
    - Motivation: Mainly because my response variable is already lies on the codomain needed for ROC-AUC. Although it's true that naive accuracy & confusion matrices provide a more easily interpretable performance metric, my predictive task doesn't have a fixed threshold. Therefore, ROC-AUC is still more robust/accurate for this use case and allows for more refined adjustments when developing my models.
- All features used will be pre/early-game metrics that can only be strictly observed before 'result', therefore this problem has to be prediction and not inference.

---

# 🚲 Baseline Model 🚲

Our baseline models uses **logistic regression** to predict the win probability of a given record. The objective is to build a simple model which highlights basic prediction relationships between the data and our target.

### Features
- `killsat15` (*Quantitative*): The # of kills a team has at the 15 minute mark.
    - Preprocessing: N/A 
    - Motivation: We chose this feature because of the potential snowball effect kills can have, and as inferred earlier in our exploratory data analysis, high-kill early games potentially lead to wins as it implies early game dominance. Additionally, it might not be as correlated to gold difference as CS is, and since we don't want any collinearity, this makes the metric perfect for simple logistic regression. This feature, and the following 2 features as well, doesn't require any encoding as they are already numeric. Standardization does not improve performance, so we will save `StandardScaler` for our final model. 
- `golddiffat15` (*Quantitative*): The gold difference a team has at the 15 minute mark with respect to the enemy team.
    - Preprocessing: N/A
    - Motivation: We observed a strong, positive relationship between gold difference and winning outcomes earlier.
- `xpdiffat15` (*Quantitative*): The gold difference a team has at the 15 minute mark with respect to the enemy team.
    - Preprocessing: N/A
    - Motivation: From personal experience, XP leads dictate the pace of a lane. If a team has an overwhelming XP lead, they can gain control of the lanes and pressure turrets and objectives far easier than if XP was even. And unlike gold, XP doesn't have to be redeemed at base, so a team with an XP lead will always be able to push the advantage.

### Model Results (ROC-AUC Accuracies):

|  Model Name     |   Train Acc.   |  Test Acc.  |
| --------------- | -------------- | ----------- |
|  BaselineLogReg |   0.828189     |   0.826455  |

### Model Analysis
Training accuracy and testing accuracy being so tightly coupled suggests that our baseline model is underfitted. However, this makes perfect sense as it was made to be simple by design. As such, it might not have captured all of the feature vs. response relationships. Considering the intentional simplicity of the model and how we are restricted to just early-game metrics, the model in my opinion achieved a relatively strong performance. 

---

# 🏎️ Final Model 🏎️

The final model is called `UltLogReg` and remains a **logistic regression** algorithm, albeit a far improved version compared to `BaselineLogReg` due to improved feature engineering and hyperparameter tuning.

### Feature Selection & Engineering
- `side` (*nominal*): Blue / Red.
    - Preprocessing: `OneHotEncoder`
    - Motivation: Summoner's Rift (competitive map) is not symmetric so I figured there would be some inbalance in a large dataset.
- `golddiffat15` (*quantitative*): Gold difference for corresponding team @ 15 minutes.
    - Preprocessing: `PolynomialFeatures` -> `StandardScaler`
    - Motivation: Based on EDA above, gold difference seems to be strongly predictive of match outcome.
- `xpdiffat15` (*quantitative*): XP difference for corresponding team @ 15 minutes.
    - Preprocessing: `StandardScaler`
    - Motivation: I extrapolated results from my notebook analysis, and also the `BaselineLogReg` results seems promising with XP difference despite my initial doubts. Might be slightly collinear with gold difference though.
- `gold_delta` (*quantitative*): Difference between gold difference @ 10 and 15 minutes.
    - Preprocessing: `golddiffat10`, `golddiffat15` -> `PolynomialFeatures` -> `StandardScaler`
    - Motivation: I wanted to capture some of the signal from building gold lead spikes. These spikes (and drops) captures in-game momentums that can't be seen in other metrics.
- `xp_delta` (*quantitative*): Difference between XP difference @ 10 and 15 minutes.
    - Preprocessing: `xpdiffat10`, `xpdiffat15` -> `StandardScaler` 
    - Motivation: Same as the above feature, although likely not as strong as a predictor. Additionally, this feature is probably multicollinear with a few others, haha.
- `river` (*quantitative*): Custom formulated function that combines `firstherald` & `firstdragon`; denotes river control.
    - Preprocessing: `1.25*firstherald` + `firstdragon` -> `StandardScaler`  
    - Motivation: Found through EDA that river objective control correlates moderately with winning outcomes, with `firstherald` being slightly more impactful. So I came up with that equation.
- `lane` (*quantitative*): Custom formulated function that combines `firsttower` & `firstblood`; denotes lane control.
    - Preprocessing: `2.5*firsttower` + `firstblood` -> `StandardScaler`
    - Motivation: Once again, EDA showed me that `firsttower` is an impactful predictor. `firstblood` is kind of just there, although both of them combined become a extremely strongly correlated with wins even if `firstblood` doesn't seem to be any good as a predictor.
- `teamname` (*nominal*): The name of the team corresponding to the current record.
    - Preprocessing: `TargetEncoder` -> `PolynomialFeatures`

## Model Results (ROC-AUC Accuracies & Optimal Parameters) 

|  Model Name  | Train Acc. | Test Acc.  | LogReg λ | gold_diff_poly | gold_delta_poly | teamname_poly |         
| ------------ | ---------- | ---------- | -------- | -------------- | --------------- | ------------- |
|  UltLogReg   |  0.844237  |  0.834024  |   10.0   |       1        |       2         |       3       |

## Model Analysis
At a quick glance, `UltLogReg` improved in both training and test ROC-AUC accuracy. Although it might be a tiny bit more overfitted than before (test acc. > train acc. by a bit), it's still within our comfortable "best fit" margin. Based on the optimal parameters and hyperparameters, `teamname` and metrics related to gold difference seem to the best predictive metrics. Gold difference being a strong predictor makes tons of sense, especially after all of the previous data analysis we did confirming it. But how did `teamname` manage to sneak in there? The answer is `TargetEncoder`. This transformation effectively encodes the win rate with respect to each category in `teamname` (e.g. each team) without response leakage. So in the end, the **most predictive pre/early-game metrics for match outcomes are team winrates and gold difference**. 

## Comparison to BaselineLogReg (ROC-UAC Accuracies)

- Training accuracy gain: `+0.016048`
- Testing accuracy gain: `+0.007569`

However, for the features given to `BaselineLogReg`, it was better bit relative compared to `UltLogReg`. The final model seems to be a tad bit overfitted, likely due to the cubic term. More hyperparameter adjustment could certainly help, although the gain would still be pretty minimal. Overall, we achieved great improvement over with the final model. And in my opinion, close to the best possible results given the self-constraint features and dataset. How am I so sure? It is because I built a model that had a target variable as encoded as a feature and it performed only a bit better than this, haha.