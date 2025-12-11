# **League of Legends Position Predictor**

## **Introduction:**

**The Dataset:** This project analyzes a dataset of competitive League of Legends esports matches from 2022. League of Legends is a very complex multiplayer online battle arena (MOBA) game in which ten players (five per team) compete in lengthy strategic matches. Each player on the team is designated one of five roles: Top, Jungle, Mid, Bot, or Support. This dataset contains a row for each of the 10 players per match, as well as a row for each of the teams per match containing summary statistics for their performance in the game. These rows capture a wide range of statistics including gold, kills, assists, wards placed, etc. The dataset contains **150,588** rows constituting **12,549 competitive matches**.

**The Question:** Initially the question was asking whether we could predict a players **role** (Top, Jungle, Mid, Bot, Support) based on various traits including the name of the champion they played. However, since most champions are played exclusively in one specific role, the question eventually changed to: Can we predict a players role based on their end-game performance statistics **without** knowing which champion they played?

**Why this is important:** Relying on champion selection makes our predictions trivial. The aim of this project is to see if we can pinpoint the statistical differences in how different roles in League of Legends are played: Who achieves more assists vs. kills? Who gets the most monster kills? Who places the most wards? All of these define the playstyles for different roles and will help us in determing who played what role.

**Relevant Columns:**
- `position`: This is the target variable we are trying to predict and represents what role was assigned to the player.
- `damageshare`: This represents the percentage of the team's total damage this player inflicted.
- `earnedgpm`: This column represents the gold earned per minute of a certain player. This distinguishes higher earning roles (Bot/Mid) from low earners (Support)
- `damagetakenperminute`: This represents the average amount of damage absorbed by a player per minute. This helps separate more "tank-y" roles such as Jungle and Top
- `monsterkills`: The amount of monsters the player killed. Most useful in identifying Jungle players.
- `wpm` **(Wards Per Minute)**: This is a statistic that deals with map control. This identifies the support role as they primarily place down wards.
- `damagetotowers`: A critical statistic in the game represented damage inflicted on in-game structures. Useful for distinguishing Top/Mid-lane players.
- `cspm` **(Creep Score Per Minute)**: A statistic representing the average amount of minion kills a player has.
- `result`: Whether the player was on the winning (1) or losing (0) team. Used mostly in our fairness analysis for determining possible bias in our model.

## **Data Cleaning and Exploratory Data Analysis:**

### Data Cleaning:

The given dataset for League of Legends competitve esports matches isn't very messy as it is– however we do need to reformat our dataset so it only features data from individual players rather than teams as a whole since the aim of our project is to predict the roles of individual players. To do this, we redefined a dataset `lol_individual` that only features rows where the `position` column **isn't** "team". These specific rows feature aggregate data for the team's statistics as a whole.

After doing this cleaning, some of our relevant columns still contained missing datapoints. After calculating the value counts of missing values in each of these columns, we found that multiple of these columns only featured 10 rows with missing data. Since this is such a trivial value, we dropped these rows as it wouldn't have much impact on any findings.

Here is the first 5 rows of our dataset (with a selection of relevant columns) after conducting our cleaning:

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


