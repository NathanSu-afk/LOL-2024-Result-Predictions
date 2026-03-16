# League of Legends 2024 Sides Analysis
 
This is a final project for DSC 80 at UCSD that explores whether the side a professional League of Legends team is assigned to (Blue or Red) has a meaningful impact on their chances of winning. Using statistical analysis, hypothesis testing, missingness analysis, and predictive modeling, this project aims to uncover structural advantages in the game and use them to better understand what actually drives wins at the highest level of play.
 
**Author:** Nathan Su
 
---
 
## Introduction
 
League of Legends (LoL) is one of the most popular competitive video games in the world. Two teams of five players each compete on the same map, called Summoner's Rift, with the goal of destroying the other team's base structure, the Nexus. One of the most overlooked aspects of the game at the professional level is something that happens before the game even starts: which side of the map each team starts on. Every game, one team is assigned to Blue side and the other to Red side, and the debates have circulated for the longest time stating that the two sides are not perfectly symmetrical. Blue side gets to pick champions first in the draft phase, while Red side gets to see the full enemy team composition before making their last two picks. On the map itself, certain objectives like the Dragon, Rift Herald, and Baron Nashor are positioned in ways that naturally favor one side over the other. That being said, gold, which is the game's in-game currency, is also very important as it can improve your character and allow you and your team to secure objectives even if you are on the disadvantaged side for it. Therefore, there is also a certain level of mechanical skill and strategy that also accounts for a higher win rate.
 
With that in mind, the central question this project aims to answer is:
 
> **Is there a significant advantage to playing on the Blue side compared to the Red side in terms of win rate?**
 
This question matters because if Blue side or Red side has a consistent edge in professional play, it means the game itself has a structural imbalance, which has real implications for how teams prepare, how tournaments are organized, and how Riot Games balances the game. 
 
The dataset used to explore this comes from Oracle's Elixir, a trusted source of professional LoL match data. This project focuses specifically on 2024 professional match data. It contains important information on statistics and results that can be used by teams to understand both individual and overall team performance and how they fared against their opponents. It includes many features (objectives like dragon, rift herald, baron) and snapshots of how well the team is doing at specific intervals (Ex. goldat10, killsat10). The datasets has 122,388 rows representing thousands of games with many columns containing information on each of them.
 
The columns used in this project that were relevant to our data and their descriptions are listed below:
 
| Column | Description |
|---|---|
| `league` | The Esports tournament in which the game was played |
| `side` | Whether the team played on Blue or Red side |
| `result` | Whether the team won (1) or lost (0) the match |
| `team kpm` | Team kills per minute |
| `kills` | How many total kills a team got over the entiire match |
| `deaths` | How many total deaths a team had over the entire match |
| `earnedgold` | Total gold earned by the team during the match, excluding starting gold |
| `earned gpm` | Gold earned per minute, excluding starting gold |
| `firstdragon` | Whether the team secured the first dragon of the game, represented by 1 (True) and 0 (False) |
| `firstherald` | Whether the team secured the first Rift Herald, represented by 1 (True) and 0 (False) |
| `firstbaron` | Whether the team secured the first Baron Nashor, represented by 1 (True) and 0 (False) |
| `dragons` / `opp_dragons` | Total dragons secured by the team and by the opponent, respectively |
| `barons` / `opp_barons` | Total Baron Nashor kills secured by the team and by the opponent, respectively |
| `goldat10` / `goldat15` / `goldat20` / `goldat25` | Total gold the team had at the 10, 15, 20, and 25-minute marks of the match |
| `golddiffat10` / `golddiffat15` / `golddiffat20` / `golddiffat25` | Gold difference (team minus opponent) at the 10, 15, 20, and 25-minute marks |
| `opp_goldat10` / `opp_goldat15` / `opp_goldat20` / `opp_goldat25` | Total gold the opposing team had at the 10, 15, 20, and 25-minute marks |
 
---
 
## Data Cleaning and Exploratory Data Analysis
 
The raw Oracle's Elixir dataset includes rows for every individual player as well as summary rows for each team. Since this project is focused on team-level performance rather than individual player statistics, the first step was to keep only rows where `position == 'team'`. Additionally, only rows where `datacompleteness == 'complete'` were kept, which ensures that every row used in the analysis has fully recorded data from start to finish, so no partially tracked games were included.
 
After that, a few additional cleaning steps were applied. First, a new `kd_ratio` column was created by dividing `kills` by `deaths`, which were later dropped to reduce redundancy. Any team with 0 deaths had deaths replaced with 1 to avoid division by zero, giving a single numeric measure of how dominant a team was in combat. Second, columns like `firstdragon`, `firstbaron`, `firstherald`, and `result` were converted to True/False boolean values. Missing values in these columns were first filled with 0 and then converted, since a missing value for "first dragon" in a complete game means the team simply did not secure it. Third, in columns like `earned gpm` and `team kpm`, with 0s were replaced with NaN to properly represent any missingness in the column. Finally, out of the many columns in the original dataset, only the 26 columns relevant to this analysis were kept, reducing noise and keeping the DataFrame manageable.
 
Here is the head of the cleaned DataFrame:
 
| league | side | result | team kpm | firstdragon | firstherald | firstbaron | earnedgold | earned gpm | dragons | opp_dragons | barons | opp_barons | goldat10 | golddiffat10 | goldat15 | golddiffat15 | goldat20 | golddiffat20 | goldat25 | golddiffat25 | opp_goldat10 | opp_goldat15 | opp_goldat20 | opp_goldat25 | kd_ratio |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| TSC | Blue | True | 0.83 | True | True | True | 36396 | 1510.21 | 2.0 | 1.0 | 1.0 | 0.0 | 17004.0 | 1364.0 | 27034.0 | 2293.0 | 38246.0 | 4248.0 | 52523.0 | 12741.0 | 15640.0 | 24741.0 | 33998.0 | 39782.0 | 2.86 |
| TSC | Red | False | 0.29 | False | False | False | 23655 | 981.54 | 1.0 | 2.0 | 0.0 | 1.0 | 15640.0 | -1364.0 | 24741.0 | -2293.0 | 33998.0 | -4248.0 | 39782.0 | -12741.0 | 17004.0 | 27034.0 | 38246.0 | 52523.0 | 0.35 |
| TSC | Blue | True | 0.88 | False | False | False | 49333 | 1394.90 | 2.0 | 3.0 | 1.0 | 1.0 | 15788.0 | -88.0 | 25949.0 | -75.0 | 36104.0 | 777.0 | 45691.0 | 1459.0 | 15876.0 | 26024.0 | 35327.0 | 44232.0 | 1.55 |
| TSC | Red | False | 0.57 | True | True | True | 43943 | 1242.50 | 3.0 | 2.0 | 1.0 | 1.0 | 15876.0 | 88.0 | 26024.0 | 75.0 | 35327.0 | -777.0 | 44232.0 | -1459.0 | 15788.0 | 25949.0 | 36104.0 | 45691.0 | 0.65 |
| TSC | Blue | True | 0.69 | True | True | True | 45438 | 1298.85 | 2.0 | 3.0 | 1.0 | 0.0 | 14550.0 | -2583.0 | 24170.0 | -561.0 | 33386.0 | -1528.0 | 43051.0 | 1092.0 | 17133.0 | 24731.0 | 34914.0 | 41959.0 | 3.00 |
 
---
 
### Univariate Analysis
 
<iframe
  src="assets/eda_gold.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>
 
The distribution of `golddiffat25` is approximately normal and centered at 0, which is expected since gold difference is a zero-sum measure, where every positive gold lead for one team is an equal negative for their opponent. This shows that the data is balanced and clearly follows a predictable trend.
 
<iframe
  src="assets/eda_gpm.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>
 
The distribution of `earned gpm` (earned gold per minute) is roughly normal and bimodal. The two peaks likely represent the gold effiency of winning and losing teams. It makes sense because teams that win usually have a better gold rate than losing teams, suggesting that gold is an important factor in who wins or loses.
 
---
 
### Bivariate Analysis
 
<iframe
  src="assets/eda_winrate.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>
 
This bar chart compares the average win rate between Blue side and Red side. Blue side wins about **52.9%** of games while Red side wins about **47.1%**. While this might seem like a small difference, across thousands of games it is a consistent and notable gap. This sets up the core question of the project of whether this difference real and statistically significant, or is it just noise from random variation?
 
<iframe
  src="assets/eda_kd_gpm.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>
 
This scatter plot looks at the relationship between `earned gpm` and `kd_ratio`, with points colored by side. There is a clear positive trend teams that earn more gold per minute also tend to have higher kill-to-death ratios, which makes sense since getting kills both earns gold directly and denies the enemy from farming safely. Importantly, the Blue and Red side distributions look almost identical here, meaning neither side is consistently outperforming the other in terms of raw economic or combat efficiency. This suggests the win rate gap between the two sides may be driven by structural map factors rather than one side simply playing better.
 
---
 
### Interesting Aggregates
 
The pivot table below shows the average performance statistics for Blue and Red side teams across all 2024 professional games:
 
| side | avg_earned gpm | avg_firstbaron | avg_firstdragon | avg_firstherald | avg_kd_ratio | avg_result |
|---|---|---|---|---|---|---|
| Blue | 1209.92 | 0.50 | 0.39 | 0.61 | 1.72 | 0.53 |
| Red | 1182.26 | 0.45 | 0.61 | 0.38 | 1.51 | 0.47 |
 
Blue side has a higher average win rate (0.53 vs 0.47) and a noticeably higher average KD ratio (1.72 vs 1.51), suggesting Blue side teams tend to come out ahead in fights more often, which makes sense if they have the lead/advantage. Blue side also secures the first Herald at a much higher rate (61% vs 38%), which is consistent with Rift Herald spawning on the top side of the map, closer to Blue side's starting position. On the flip side, Red side secures the first dragon at a much higher rate (61% vs 39%), since Dragon spawns on the bottom side, which is closer to Red side. Despite Red side's clear dragon advantage, Blue side still wins more often overall. This suggests that herald and baron control, which Blue side leans toward, may be more impactful to winning than early dragon control in professional play.
 
---
 
## Assessment of Missingness
 
### MNAR Analysis
 
Within this dataset, the column `goldat25` has a significant amount of missing rows. I believe this column is likely **MNAR** (Missing Not At Random). The reasoning is that `golddiffat25` can only be recorded for games that actually last at least 25 minutes. If a game ends early, then this value simply does not exist. That means the missingness is tied to the actual value of the data, where games with the most extreme gold differences are also the games most likely to end before 25 minutes, are exactly the games where `golddiffat25` would be missing. The missing values are not random because they represent a specific type of game outcome. To fully verify this, we would use a `gamelength` column for every match, which would be a direct predictor of whether this value is missing because there is no data to record when the match is already finished.
 
---
 
### Missingness Dependency
 
To formally test missingness, `golddiffat25` was selected as the column to analyze. A new column called `gold_missing` was created, taking all the rows `golddiffat25` was missing for. Permutation tests were then run using Total Variation Distance (TVD) as the test statistic to see whether the missingness pattern of `golddiffat25` was related to other columns.
 
**Does the missingness of `golddiffat25` depend on `side`?**

**Null Hypothesis**: The distribution of `side` when `golddiffat25` is missing is the same as when it is not missing.
**Alternative Hypothesis**: The distribution of `side` when `golddiffat25` is missing is not the same as when it is not missing.

After running the permutation test, the observed TVD came out to 0.0000 with a p-value of 1.0000. We fail to reject the null hypothesis, meaning the missingness of `golddiffat25` does **not** depend on `side`. This makes sense because both Blue and Red side play the same number of games, and a game ending before 25 minutes is not inherently tied to which side won. The missing values are perfectly split between Blue and Red.

<iframe
  src="assets/missingness_side_dist.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>

<iframe
  src="assets/missingness_side_tvd.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>
 
**Does the missingness of `golddiffat25` depend on `league`?**
 
**Null Hypothesis**: The distribution of `league` when `golddiffat25` is missing is the same as when it is not missing. 
**Alternative Hypothesis**: The distribution of `league` when `golddiffat25` is missing is not the same as when it is not missing.

After running the permutation test, the observed TVD came out to 0.3310 with a p-value of 0.0000. We reject the null hypothesis, meaning the missingness of `golddiffat25` **does** depend on `league`. Looking at the data, leagues like PCL, ESLOL, and PRMP account for a disproportionate share of missing values. These tend to be more regional or developing leagues where games may end earlier on average, or where data collection practices differ slightly. This confirms that `golddiffat25` is MAR (Missing At Random) with respect to `league`, where its missingness can be partially explained by which league the game was played in.
 
<iframe
  src="assets/missingness_league_dist.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>
 
<iframe
  src="assets/missingness_league_tvd.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>
 
---
 
## Hypothesis Testing
 
The goal of this hypothesis test is to determine whether the observed win rate difference between Blue and Red side is statistically significant, or if it could have just as easily occurred by chance.

**Null Hypothesis**: The probability of winning is the same for both the Blue side and the Red side, and any observed difference in win rate is due to random chance. 
**Alternative Hypothesis**: The probability of winning is higher for the Blue side than for the Red side.

The **test statistic** chosen is the difference in win rates between Blue side and Red side (Blue win rate − Red win rate). This is a good choice because it directly measures the quantity we care about, which is how much more often Blue wins, and it is easy to interpret. A **significance level of 0.05** was used.
 
A permutation test with 500 repetitions was used. In each repetition, the `side` labels were randomly shuffled across all rows while the `result` column was kept fixed. This simulates a world where side assignment has no effect on the outcome. The observed difference was then compared to the distribution of shuffled differences to compute a p-value.

<iframe
  src="assets/hypothesis_test.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>
 
The observed difference came out to **0.0576** with a **p-value of 0.0000**. With a p-value of essentially 0, we reject the null hypothesis at the 0.05 significance level. The observed 5.76 percentage point win rate gap between Blue and Red side is extremely unlikely to occur by random chance alone. This is strong evidence that Blue side has a real structural advantage in professional 2024 League of Legends. That said, we cannot conclude with absolute certainty that Blue side always wins more as this is an observational study, not a controlled experiment, and other factors could be contributing to the pattern.
 
---
 
## Framing a Prediction Problem
 
Building on the patterns found in the EDA, the prediction problem is:
 
> **Can we predict whether a professional League of Legends team will win or lose a match based on the gold state of the game at some point in time and the objectives secured throughout the game?**
 
This is a **binary classification** problem, predicting `result` where 1 = Win and 0 = Loss. The response variable `result` was chosen because it is the only outcome that truly matters in a game of League of Legends, with every game has exactly one winner and one loser, making the classes perfectly balanced with no need to worry about any class imbalance.
 
For the baseline model, I predict thje outcome based on the 25 minute mark. However, for the final model, I also end up predicting the outcome based on the total dragons and barons taken on each side. The times that these happen vary depending on the game, making it potentially biased. However, this is still a useful model in interpreting what features determine a win the most.
---
 
## Baseline Model
 
The baseline model is a **Random Forest Classifier** implemented in a single `sklearn` Pipeline.
 
**Features used:**
 
`golddiffat25` is a **quantitative** feature representing the gold difference between the team and its opponent at the 25-minute mark. It was passed through without any transformation as a raw numeric feature. `firstbaron` is a **nominal (boolean)** feature indicating whether the team secured the first Baron Nashor. It was encoded using `OneHotEncoder`.
 
`GridSearchCV` with 5-fold cross-validation was used to find the best `max_depth` from the values [2, 4, 5, 6, 7, 9, 10]. The best `max_depth` was **7**, with a best CV accuracy of **0.8513** and a test accuracy of **0.8491**.
 
For a two-feature model, 84.9% test accuracy is a great starting point. Permutation importance tests confirmed that `golddiffat25` is by far the more important feature because shuffling it caused accuracy to drop all the way to **58.96%**, essentially near chance. Shuffling `firstbaron` only dropped accuracy to **79.27%**, meaning it adds value but is largely secondary to gold advantage. This makes sense because gold difference captures a broad picture of who is winning across the whole game, while first baron is just one binary event and is also somewhat tied to golddiff because it gives the team buffs and more gold.
 
---
 
## Final Model
 
The final model keeps the same Random Forest Classifier framework but introduces two new engineered features and applies a feature transformation to `golddiffat25`.
 
The first new feature is `baron_diff`, computed as `barons - opp_barons`. Rather than just knowing who got the first baron, this captures how thoroughly a team controlled the Baron objective across the entire game. Baron Nashor is the most powerful late-game objective, as it gives a team a massive minion buff that pushes towers and structures. A team that secured 2 barons while the opponent had 0 is in a completely different position than a team with just 1 baron against 1. The second new feature is `dragon_diff`, computed as `dragons - opp_dragons`. Dragon stacks provide permanent stacking buffs to a team, and teams with more dragons can end up gaining a bigger lead throughout the game.
 
The reason these features improve the model is that the baseline model only knew whether a team got the first baron which is a single binary event. The new features tell the model how the entire objective game played out over the course of the match. Objectives secured later in the game are often the deciding factor in professional matches, where early leads can be erased but a 2-baron advantage almost always closes out a game. These differential features also account for the opponent's performance simultaneously, which raw counts do not.
 
For feature transformations, `golddiffat25` was passed through a `StandardScaler` to normalize it so the model weights it appropriately relative to the integer-scale objective features. `baron_diff` and `dragon_diff` were passed through as-is since they are already on a natural integer scale with a small range.
 
I used `GridSearchCV` with 5-fold CV to tune `rf__max_depth` across [2, 5, 7, 10], `rf__min_samples_split` across [2, 5, 10], and `rf__criterion` across ['gini', 'entropy']. The best parameters found were `{'rf__criterion': 'gini', 'rf__max_depth': 5, 'rf__min_samples_split': 10}`, with a final train accuracy of **0.9003** and a test accuracy of **0.9021**. While we use information from a later game state, I still think the tradeoff is justifiable because it encompasses as much of the game as possible to determine the effects of features throughout an entire game.
 
Test accuracy improved from **84.91%** to **90.21%**, with a gain of about 5.3 percent. Permutation importance on the final model revealed that `baron_diff` is now the single most impactful feature (shuffling it dropped accuracy to **79.20%**), followed by `golddiffat25` (dropping to **82.40%**) and then `dragon_diff` (dropping to **86.10%**). In the baseline model, gold was dominant, but in the final model with more features, baron control becomes the strongest individual predictor in the later stages of the game, which aligns with what most players have argued: Baron Nashor is the most game-deciding objective in League of Legends. Also, since Blue has an advantage of defeating the Baron Nashor, it does help explain why there could be an inherent advantage to playing on the Blue side.
 
---
 
## Fairness Analysis
 
The final model uses `baron_diff` and `dragon_diff` as two of its three features. These two objectives are physically located on opposite sides of the map, with Baron Nashor is on the top side, which is closer to Blue side's base, while Dragon spawns on the bottom side, which is closer to Red side. This creates a natural concern: does the model predict wins more accurately for one side than the other, because the features it uses are inherently biased toward the side that has easier access to them?
 
To investigate this, a fairness analysis was conducted comparing model accuracy for **Group X (Blue side teams)** versus **Group Y (Red side teams)**. Accuracy is appropriate here because an equal number of biased features were used in determining the outcome of the match. Even though barons seem to be more important than dragons, dragons still contribute a decent amount in determining the outcome of the game and this inherent disadvantage is already reflected in the tests we did above, accurately representing game design.
 
**Null Hypothesis**: The model is fair, meaning its accuracy for Blue side teams and Red side teams is roughly the same, and any observed difference is due to random chance. The **Alternative Hypothesis**: The model is unfair, meaning its accuracy for Blue side teams is higher than for Red side teams. 

The **test statistic** is the difference in accuracy (Blue accuracy − Red accuracy), where a positive value means the model is more accurate for Blue side. A **significance level of 0.05** was used.

A permutation test with 1,000 repetitions was run. In each repetition, the `side` labels on the test set were randomly shuffled, and the accuracy difference was recalculated. The observed difference (Blue − Red) came out to **0.0024** with a **p-value of 0.4450**. We fail to reject the null hypothesis. The tiny 0.24 percentage point accuracy gap between Blue and Red side is well within the range of normal random variation. Even though the model's features are structurally tied to objectives that favor different sides of the map, the model ends up being just as accurate for Red side teams as it is for Blue side teams. This suggests the model generalizes fairly across both sides despite the asymmetry baked into the game.

<iframe
  src="assets/fairness_analysis.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>
