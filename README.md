# League of Legends Position Predictor

**The Dataset:** This project analyzes a dataset of competitive League of Legends esports matches from 2022. League of Legends is a very complex multiplayer online battle arena (MOBA) game in which ten players (five per team) compete in lengthy strategic matches. Each player on the team is designated one of five roles: Top, Jungle, Mid, Bot, or Support. This dataset contains a row for each of the 10 players per match, as well as a row for each of the teams per match containing summary statistics for their performance in the game. These rows capture a wide range of statistics including gold, kills, assists, wards placed, etc. The dataset contains **150,588** rows constituting **12,549 competitive matches**.

**The Question:** Initially the question was asking whether we could predict a players **role** (Top, Jungle, Mid, Bot, Support) based on various traits including the name of the champion they played. However, since most champions are played exclusively in one specific role, the question eventually changed to: Can we predict a players role based on their end-game performance statistics **without** knowing which champion they played?

**Why this is important:** Relying on champion selection makes our predictions trivial. The aim of this project is to see if we can pinpoint the statistical differences in how different roles in League of Legends are played: Who achieves more assists vs. kills? Who gets the most monster kills? Who places the most wards? All of these define the playstyles for different roles and will help us in determing who played what role.

**Relevant Columns:**
- `position`: This is the target variable we are trying to predict and represents what role was assigned to the player.
- `damageshare`: This represents the percentage of the team's total damage this player inflicted.
- `earnedgpm`: This column represents the gold earned per minute of a certain player. This distinguishes higher earning roles (Bot/Mid) from low earners (Support)
- `damagetakenperminute`: This represents the average amount of damage absorbed by a player per minute. This helps separate more "tank-y" roles such as Jungle and Top
- `monsterkills`: The amount of monsters the player killed. Most useful in identifying Jungle players.
- `damagetotowers`: A critical statistic in the game represented damage inflicted on in-game structures. Useful for distinguishing Top/Mid-lane players.
- `cspm` **(Creep Score Per Minute)**: A statistic representing the average amount of minion kills a player has.
- `result`: Whether the player was on the winning (1) or losing (0) team. Used mostly in our fairness analysis for determining possible bias in our model.