# **League of Legends Position Predictor**

## **Introduction:**

**The Dataset:** This project analyzes a dataset of competitive League of Legends esports matches from 2022. League of Legends is a very complex multiplayer online battle arena (MOBA) game in which ten players (five per team) compete in lengthy strategic matches. Each player on the team is designated one of five roles: Top, Jungle, Mid, Bot, or Support. This dataset contains a row for each of the 10 players per match, as well as a row for each of the teams per match containing summary statistics for their performance in the game. These rows capture a wide range of statistics including gold, kills, assists, wards placed, etc. The dataset contains **150,588** rows constituting **12,549 competitive matches**.

**The Question:** Initially the question was asking whether we could predict a players **role** (Top, Jungle, Mid, Bot, Support) based on various traits including the name of the champion they played. However, since most champions are played exclusively in one specific role, the question eventually changed to: *Can we predict a players role based on their end-game performance statistics **without** knowing which champion they played?*

**Why this is important:** Relying on champion selection makes our predictions trivial. The aim of this project is to see if we can pinpoint the statistical differences in how different roles in League of Legends are played: Who achieves more assists vs. kills? Who gets the most monster kills? Who places the most wards? All of these define the playstyles for different roles and will help us in determining who played what role.

**Relevant Columns:**
- `position`: This is the target variable we are trying to predict and represents what role was assigned to the player.
- `damageshare`: This represents the percentage of the team's total damage this player inflicted.
- `earnedgpm`: This column represents the gold earned per minute of a certain player. This distinguishes higher earning roles (Bot/Mid) from low earners (Support)
- `damagetakenperminute`: This represents the average amount of damage absorbed by a player per minute. This helps separate more "tank-y" roles such as Jungle and Top
- `monsterkills`: The amount of monsters the player killed. Most useful in identifying Jungle players.
- `wpm` **(Wards Per Minute)**: This is a statistic that deals with map control. This identifies the support role as they primarily place down wards.
- `damagetotowers`: A critical statistic in the game representing damage inflicted on in-game structures. Useful for distinguishing Top/Mid-lane players.
- `cspm` **(Creep Score Per Minute)**: A statistic representing the average amount of minion kills a player has.
- `result`: Whether the player was on the winning (1) or losing (0) team. Used mostly in our fairness analysis for determining possible bias in our model.

## **Data Cleaning and Exploratory Data Analysis:**

### Data Cleaning:

The given dataset for League of Legends competitive esports matches isn't very messy as it is– however we do need to reformat our dataset so it only features data from individual players rather than teams as a whole since the aim of our project is to predict the roles of individual players. To do this, we redefined a dataset `lol_individual` that only features rows where the `position` column **isn't** "team". These specific rows feature aggregate data for the team's statistics as a whole.

After doing this cleaning, some of our relevant columns still contained missing datapoints. After calculating the value counts of missing values in each of these columns, we found that multiple of these columns only featured 10 rows with missing data. Since this is such a trivial value, we dropped these rows as it wouldn't have much impact on any findings.

Here are the first 5 rows of our dataset (with a selection of relevant columns) after conducting our cleaning:

| gameid                | position   | champion   |   earned gpm |   damageshare |   monsterkills |    wpm |   result |
|:----------------------|:-----------|:-----------|-------------:|--------------:|---------------:|-------:|---------:|
| ESPORTSTMNT01_2690210 | top        | Renekton   |      250.928 |     0.278784  |             11 | 0.2802 |        0 |
| ESPORTSTMNT01_2690210 | jng        | Xin Zhao   |      188.021 |     0.208009  |            115 | 0.2102 |        0 |
| ESPORTSTMNT01_2690210 | mid        | LeBlanc    |      208.231 |     0.252086  |             16 | 0.6655 |        0 |
| ESPORTSTMNT01_2690210 | bot        | Samira     |      239.405 |     0.196358  |             18 | 0.4203 |        0 |
| ESPORTSTMNT01_2690210 | sup        | Leona      |      101.856 |     0.0647631 |              0 | 1.0158 |        0 |


### Exploratory Data Analysis:

Below is a histogram representing the earned gold per minute (`earned gpm`) for all positions. It seems like there is a wide spread of values included with a right tail. Since the range of gold earned is so high with a wide spread, it's possible that different roles have different gold-earning habits with many being high earners and many not earning much gold at all.

<iframe src="assets/univariate_plot.html" width="800" height="600" frameborder="0"></iframe>

Here is box-plot representing the spread of monster kills (`monsterkills`) divided across all the different possible positions in the game. This shows us that Jungle players have a significantly higher number of monster kills compared to the other positions. This will be an advantageous feature in distinguishing Junglers from other roles in our model down the line!

<iframe src="assets/bivariate_plot.html" width="800" height="600" frameborder="0"></iframe>

Below is a pivot table representing the average of multiple statistics across the various positions. This allows us to see multiple macro trends across the roles. For example, Support players have a significantly higher `wpm` and many more `assists` than the other roles on average. Bottom lane players have a significantly higher amount of `damagetotowers`, and bottom, top, and mid-lane players all have relatively high `earned gpm`.

| position   |   assists |   damagetotowers |   earned gpm |   monsterkills |      wpm |
|:-----------|----------:|-----------------:|-------------:|---------------:|---------:|
| bot        |   5.37467 |         5247.85  |      299.917 |      22.5505   | 0.431304 |
| mid        |   5.89495 |         2761.66  |      267.468 |      17.102    | 0.413005 |
| top        |   5.02403 |         4105.14  |      257.898 |      15.1288   | 0.38049  |
| jng        |   6.85714 |         1483.36  |      212.393 |     143.216    | 0.357516 |
| sup        |   9.19001 |          539.651 |      110.589 |       0.440145 | 1.45195  |

## **Assessment of Missingness:**

### NMAR Analysis:

I believe the missingness in columns `ban1`, `ban2`, `ban3`, `ban4`, and `ban5` are examples of NMAR (Not Missing At Random) data. These values are likely missing because either the players didn't choose to ban anyone, or they ran out of time to ban a champion. If obtained data claiming whether players ran out of time or skipped banning someone, we would explicitly know why a champion wasn't picked for the ban. Then these columns would have MAR (Missing At Random) missingness. 

### Missingness Dependency:

To investigate the missingness mechanism of the `teamid` column (which had over 1700 missing values), we performed permutation tests against other columns in the dataset.

#### Test 1: Dependency on Game Result
The `teamid` column in our dataset has a significant amount of missing data. Below we will run a permutation test to determine whether the missingness of this column depends on the `result` column (win/loss)

- **Null Hypothesis:** The missingness of `teamid` is independent of `result`; that is, the average win rate for rows with `teamid` data present is the same as the average win rate for rows with `teamid` data missing.
- **Alternative Hypothesis:** The missingness of `teamid` depends on `result`; that is, the average win rate for rows with `teamid` data present is *different* than the average win rate for rows with `teamid` data missing.
- **Test Statistic:** The difference in win rate (Win Rate with data present - Win Rate with data missing)
- **Result:** Our observed test statistic was 0.05 (50% win rate with data, 45% win rate missing data). After 500 permutations, our test resulted in a p-value of 0.0. The below plot shows our simulated distribution with the red line representing our observed difference. Visually, this value is much more extreme than our simulations.

<iframe src="assets/missingness_permutation.html" width="800" height="600" frameborder="0"></iframe>

- **Interpretation:** In conclusion, we **reject** the null hypothesis. The missingness is dependent on game result (MAR – Missing At Random). Specifically, this suggests that losing teams are significantly more likely to have a missing `teamid` than winning teams.

#### Test 2: Independence from Position:
Below we will run a permutation test to see whether the missingness of `teamid` depends on a player's `position`.

- **Null Hypothesis:** The missingness of `teamid` is independent of `position`
- **Alternative Hypothesis:** The missingness of `teamid` depends on `position`
- **Test Statistic:** The Total Variation Distance (TVD) between `position` distributions
- **Result**: The observed TVD was 0.0 (meaning the distribution was identical in both groups). After running our permutation test we achieved a p-value of 1.0.
- **Interpretation:** We **fail to reject** the null hypothesis. Our missingness is independent of `position`, meaning it's MCAR (Missing Completely at Random) with respect to `position`. A support player is no more likely to have a missing `teamid` than a top-lane player, etc. This is likely because when a team identifier is missing for one player in a team, it's missing for everyone on the team.

## **Hypothesis Testing:**

For our hypothesis test, we wanted to see whether mid-lane players deal a higher percentage (`damageshare`) of damage than top-lane players. This is a useful test because top and mid-lane players tend to have relatively similar statistics and this could be a useful distinguishing feature between those two categories.

- **Null Hypothesis:** There is no difference between the average `damageshare` of top lane players and the average `damageshare` of midlane players
- **Alternative Hypothesis:** The average `damageshare` of midlane players is larger than the average `damageshare` of top lane players
- **Test Statistic:** The difference in means (Mean Top Damage Share - Mean Mid Damage Share)
- **Significance Level:** 0.05

- **Justification:** We chose a difference in means for our test statistic because `damageshare` is a numerical feature and this is a natural measure of the center of both categories. We choose to conduct a permutation test because we don't know the distribution of `damageshare`, and permutations don't rely on any parametric assumptions.

### Results:

- **Observed Value:** -0.026 (Mid-lane players dealt ~2.6% more of their team's damage than Top-lane players on average)
- **P-value:** 0.0
- **Interpretation:** We **reject** the null hypothesis. This suggests that top-lane players deal significantly less damage than mid-lane players on average. This confirms `damageshare` is a valuable feature we can potentially use in our model to distinguish between top and mid-lane players.

## **Framing a Prediction Problem:**

**Prediction Problem:** We are building a model to predict the `position` (Top, Mid, Bot, Jungle, Support) of a player in League of Legends based on their in-game performance statistics.

 **Problem Type:** This is a Multi-Class Classification problem. We have five distinct categories that we are trying to predict and sort different players into.

 **Response Variable:** Our response variable is `position`. We chose this because we want to understand how different roles are defined statistically and how we can quantify a person's "playstyle". We are essentially testing whether the roles in League of Legends are distinct enough to be identified based on statistical reasoning.

 **Evaluation Metric:** In League of Legends there is strictly one player per role in each game. That means there is no class imbalance as each role represents exactly 20% of our data. Since we don't have class imbalance, accuracy is the best and most intuitive way to measure the success of our model. F1 scores are useful when classes are imbalanced, however that is not the case with this problem.

 All of the metrics we will train our model on are post-game features collected once the game has finished. Our dataset has multiple features like `killsat15`, `xpat20`, etc. representing statistics throughout the game, however they will not be used in our model. So the "time of prediction" for our model is after a game has been completed.

## **Baseline Model:**

**Model Description:** For our baseline model, we used a Random Forest Classifier to predict a player's role. We used a pipeline that preprocessed features before using the classifier with default hyperparameters. 

**Model Features:**
- `champion` **(Nominal)**: This column represents the name of the champion played. Since this is categorical data without order, we used a One-Hot Encoding creating a binary column for each champion in the game.

- `cspm` **(Quantitative)**: This measures how many minions a player kills. We scaled using a StandardScaler.

- `damageshare` **(Quantitative)**: This is the percentage of the team's total damage dealt by a particular player. We used StandardScaler to transform this feature.

**Performance:**
- **Training Accuracy:** ~90.7%
- **Test Accuracy:** ~89.9%

**Evaluation:** While numerically this model seems great, an accuracy of 90% seems too good to be true. These numbers represent a conceptual flaw in building this model: in League of Legends, most champions are played exclusively in one position. Our model wasn't learning based off of playstyles; it was memorizing which champion was played in which role and sorting them as such. While it was excellent at predicting, it didn't answer any questions about the behavioral differences between roles. This led us to remove the `champion` column from our Final Model moving forward so we could answer a more interesting question.

## **Final Model:**

**Features Added:** To build a more robust model that predicts based on *behavior* rather than champion name, we removed the `champion` column to prevent data leakage. To compensate for this loss in information, we added five important features:
- `monsterkills`: This feature was added to identify Junglers. As seen in our EDA, Junglers have a significantly higher number of monster kills than any other role.
- `damagetotowers`: This was added to distinguish between top and mid-lane players. In a typical game of League of Legends, Top laners stay in a side lane and hit towers while Mid laners roam the map. We expect Top laners to have significantly higher tower damage.
- `assists`: This was added to distinguish Support and Mid laners. Supports and Mid lane players roam the map and typically assist in kills with one another. On the contrary Jungle, Bot lane, and Top lane are more isolated roles leading to a smaller number of assists.
- `earned gpm`: This was added to separate Bot/Mid lane roles from Support roles. The distribution of gold is right-skewed as we saw in our univariate EDA plot. Because of this skew we applied a QuantileTransformer to normalize the feature.
- `damagetakenperminute`: This was added to identify tank players who absorb a lot of damage. This is common in isolated roles such as Top lane and Jungle who need to defend for themselves with little support.

**Modeling Algorithm and Hyperparameters:** Again, we chose a Random Forest Classifier becuase it handles categorical predictions and non-linear relationships. To optimize our model we used GridSearchCV with 5-fold cross validation. We tuned three different hyperparameters to balance our model and prevent overfitting:
- `n_estimators`: Tuned to 100
- `max_depth`: Tuned to 15, we limited the depth to prevent our model from overfitting and memorizing the patterns of specific games
- `min_samples_leaf`: Tuned to 10, this forced the model to only create rules that applied to at least 10 players, ensuring we based our predictions on actual patterns in the dataset rather than specific games and outliers.

#### **Performance Improvement:**
- **Baseline Accuracy (with `champion`):** ~90%
- **Baseline Accuracy (without `champion`):** ~72%
- **Final Model Accuracy:** ~82.9% on our Training set and ~78.9% on our Test set (after removing data leakage from `champion` and tuning features/variables)

**Confusion Matrix:** The below Confusion Matrix shows that our model is nearly perfect at predicting Junglers and Support players. This is likely because they have more distinct identifiers (`monsterkills` for Junglers and high `assists`/low `earned gpm` for support). However, our model struggles with distinguishing Mid Laners from Top/Bot Laners. This is likely because all of these roles have similarly high gold and low damage.

<iframe src="assets/confusion_matrix.html" width="800" height="600" frameborder="0"></iframe>

**Conclusion:** While our raw accuracy dropped from our Baseline to our Final Model, our Final Model is a qualitative improvement and answers the question of the project in a more meaningful way. Our original model was essentially memorizing the names of champions which is trivial. When removing `champion`, our accuracy initially dropped to 72%, however by adding features such as `monsterkills` and `assists`, we recovered 7% accuracy. Our Final Model is more successful at predicting player positions based off of playstyles rather than by sorting characters. 


## **Fairness Analysis:**

**Groups:** We analyzed whether or not our model was biased based on whether a team won/lost a game:
- *Group A:* Winning team (`result == 1`)
- *Group B:* Losing team (`result == 0`)


**Evaluation Metric:** We chose to evaluate our model based on accuracy. We are mostly trying to measure our model's correctness across the five possible roles. Since all of the classes in our Multi-class Classification are balanced, accuracy provides a clear summary of whether our model is biased towards a certain group.

**Hypotheses/Test Statistic & Significance Level:**
- **Null Hypothesis:** Our model is fair: the accuracy is roughly the same for both winning and losing players.
- **Alternative Hypothesis:** The model is unfair: its accuracy for players on the winning team is significantly higher than its accuracy for players on the losing team.

- **Test Statistic:** Difference in Accuracy (Accuracy of Winners - Accuracy of Losers)
- **Significance Level:** 0.05

**Results:**
- **Observed Difference:** 0.028 (Our model was 2.8% more accurate predicting roles for winning teams)
- **P-value:** 0.0

**Conclusion:** Since our p-value (0.0) is less than our significance level (0.05), we **reject** the null hypothesis. Our model is definitely significantly biased towards winning teams. This bias likely exists because players on winning teams will likely exhibit the "ideal" traits of their roles. Meanwhile players on losing teams will have deflated stats (lower gold, lower cspm, etc.) which can cause them to be classified incorrectly.


