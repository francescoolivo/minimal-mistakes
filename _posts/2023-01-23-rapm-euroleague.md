---
title: "Calculating RAPM for Euroleague"
excerpt: "Euroleague Regularized Adjusted Plus Minus from 2017 to 2022"
header:
    teaser: "/assets/images/articles/rapm/players.png"
categories:
- "Analyses"
---

Or, in other words, a practical example of how pivoting turns out usefeul.
## Abstract

Plus Minus (+/-) is a basketball statistic which tries to evaluate the performance of a team when a player is on the court, by summing the score differential across all actions when a player is on court. Nonetheless, Plus Minus ( +/-) is known to have several shortcomings.
The main reason is that since basketball is a team sport, a player alone is not entirely accountable for the performance of the team. In fact, if an average player constantly shares the court with very good teammates, he will generally have a high +/-, despite having a reduced impact on the game. On the contrary, a good player in a bad team will likely have a negative +/-, despite bringing positive value to his team.

Adjusted Plus Minus (APM) is an advanced metric that tries to evaluate a player's impact regardless of the teammates and opponents he shares the court with. It was first developed by Dan Rosenbaum in [Measuring How NBA Players Help Their Teams Win](http://www.82games.com/comm30.htm). It does so by using a regression considering all the players on court in a given part of the game and the result of that specific part of game. Nonetheless, it has a major problem, which is the tendency to overfit data. To overcome this, Regularized Adjusted Plus Minus (RAPM) was developed, which regularizes data to avoid overfitting and smoothen outliers.

RAPM is a spread and common statistic in NBA, both for the bigger interest that NBA draws, both for the easiness to fetch NBA data. Yet, it is not very common in European basketball, for a combination of factors, such as the absence of a unified stats API, fewer fans and way smaller datasets.
In fact, on average an NBA season has more than 4 times the number of games than a EL season; also, games are longer (48 minutes VS 40 minutes), and are played at a higher pace (in the current season the average possession lasts 16.9 seconds in EL, 15.2 in NBA).

The goal of this project is to compute APM and RAPM for the last 5 seasons of Euroleague, and to see whether it can be a reliable metric for European basketball too. In such a case, it could be used to adjust other advanced stats based on RAPM (such as Box Plus Minus, RAPTOR etc.) to Euroleague, which has different rules and playing style than NBA.

## Basic concepts

We compute APM by building a matrix with stints as rows and players as columns.

A stint is a set of actions where the same ten players are on court. For every stint we record the number of possessions (computed as the average between the home team possessions and the away team possessions) and the net differential among the two teams, computed as the score difference of the stint per 100 possessions. The net differential is positive if the home team outscores the away team, negative otherwise.

For each player we place 1 if the player is on court for the home team in that stint, -1 if he is on the court for the away team, 0 otherwise.

Let's make a simple example on a 3 vs 3 game, where players A, B and C are on court for the home team H, and players D, E and F are on court for the visitor team V, G is on the bench for team H and J is on the bench for team V.
The teams play 5 possessions each, and the home team outscores the visitor team by 2 points, Thus, the home team has a +40 net rating over 100 possessions. Our one row vector to represent this stint will look like this:

| difference | possessions | net rating | A   | B   | C   | D   | E   | F   | G   | J   |
|------------|-------------|------------|-----|-----|-----|-----|-----|-----|-----|-----|
| 2          | 5           | 40         | 1   | 1   | 1   | -1  | -1  | -1  | 0   | 0   |

After this stint, player G replaces player B and player J replaces player F. They play 10 possessions each, and the visitor team outscores the home team by 3 points. Our matrix will now look like:

| difference | possessions | net rating | A   | B   | C   | D   | E   | F   | G   | J   |
|------------|-------------|------------|-----|-----|-----|-----|-----|-----|-----|-----|
| 2          | 5           | 40         | 1   | 1   | 1   | -1  | -1  | -1  | 0   | 0   |
| -3         | 10          | -30        | 1   | 0   | 1   | -1  | -1  | 0   | 1   | -1  |

Finally, player B replaces player G. In this stint, the home team outscores the opponents by 6 points, in 10 possessions each. Out matrix becomes:

| difference | possessions | net rating | A   | B   | C   | D   | E   | F   | G   | J   |
|------------|-------------|------------|-----|-----|-----|-----|-----|-----|-----|-----|
| 2          | 5           | 40         | 1   | 1   | 1   | -1  | -1  | -1  | 0   | 0   |
| -3         | 10          | -30        | 1   | 0   | 1   | -1  | -1  | 0   | 1   | -1  |
| 6          | 10          | 60         | 1   | 1   | 1   | -1  | -1  | 0   | 0   | -1  |

More in detail, this is the individual performance of every player:

| team | player | +/- | possessions | net rating |
|------|--------|-----|-------------|------------|
| H    | A      | 5   | 25          | 20         |
| H    | B      | 8   | 15          | 53.3       |
| H    | C      | 5   | 25          | 20         |
| H    | G      | -3  | 10          | -30        |
| V    | D      | -5  | 25          | -20        |
| V    | E      | -5  | 25          | -20        |
| V    | F      | -2  | 5           | -40        |
| V    | J      | -3  | 20          | -15        |

Just from this table, we can notice that normalizing across possessions leads players with the same +/- to have different on court net rating. Both player G and player J had a -3 +/-, but since G got it in half the possessions, his net rating is twice as negative.

![Correlogram between some performance metrics](/assets/images/articles/rapm/correlogram.png)

It is visible how +/- and on court net rating have a strong correlation with the team record, and this makes sense, since we expect good teams to have good players, and good players to create good teams. Yet, not all players in a good team are good, and the same applies for bad teams.

APM and RAPM should ideally filter this out, since a player can have a positive impact despite playing in a bad team, which would usually lead to a bad plus minus.

## Data

I will not delve into details of how data was scraped and cleaned, since it is out of the scope of the project. You can download the datatset [here](/assets/zips/data.zip)

Data consists of 5 dataframes for each of the 5 seasons, containing play-by-play logs, boxscores and information about players, teams and games.
The dataframes will be concatenated in order to have only five dataframes instead of twentyfive, and this will be dealt with by the utility function.

In particular, these are the columns of the play-by-play logs:
- season_id, edition_id, game_id: to identify the game
- action_number: the number of the action within the game
- period, remaining_period_time: temporal information about the action
- home_score, away_score: score of the game at the end of the action
- type: the event of the action, such as 2FGM for a 2P scores, AST (assist), STL (steal) et cetera.
- player_id: the id of the player who made the action
- team_id, opponent_id: the id of the team who made the action and of the opponent, who received the action
- x, y: coordinates (only in shots)
- details: additional flags about the action, separated by ':'
- linked_action_number: the number of the action that the current action is referring. For instance, a rebound is always linked to a missed shot
- h1:h5: the ids of the 5 players on court for the home team
- a1:a5: the ids of the 5 players on court for the away team

```R
library(tidyverse)

source('/home/francescoolivo/PycharmProjects/sdeng/readers/stats/stats.R')
source('/home/francescoolivo/PycharmProjects/sdeng/readers/import/R/csv.R')

setwd('/home/francescoolivo/PycharmProjects/Virtus')

data <- get_data('analyses/RAPM/data')
ign <- list2env(data, envir = .GlobalEnv)

head(actions)
```

In order to calculate APM, we need to transform this dataframe to the format of the previous example. This will be done by using the `tidyverse` library:

The first step requires to join the play-by-play logs (`actions` dataframe) with information about the game (`games` dataframe), in order to know whether the team in an action is the home team or the away team.

We also compute the score difference for every team at every action, and add suffix to free throws that end a possession, namely free throws containing a 2/2 or a 3/3 flag in the `details` column.
```R
df <-
	actions %>%
		inner_join(games, by = c('season_id', 'edition_id', 'game_id'), suffix = c('', '_game')) %>%
		group_by(game_id) %>%
		mutate(
			team = case_when(
				team_id == home_team_id ~ 'home',
				team_id == away_team_id ~ 'away',
				TRUE ~ NA_character_),
			   FT_PE = grepl('.*2/2.*', details) | grepl('.*3/3.*', details),
			   type = ifelse(type %in% c('FTA', 'FTM') & FT_PE, paste0(type, '_PE'), type),
			   home_difference = home_score - lag(home_score, 1, 0),
			   away_difference = away_score - lag(away_score, 1, 0),
		)
```

The next step is to count the number of possessions, computed as the average between the home team possessions and the away team possessions.
```R
df <- select(df, game_id, type, team, home_difference, away_difference, h1:a5) %>%
	group_by(h1, h2, h3, h4, h5, a1, a2, a3, a4, a5) %>%
	summarise(
		home_possessions = sum(team == 'home' & (type %in% c('2FGA', '2FGM', '3FGA', '3FGM', 'TOV') | (type %in% c('FTA_PE', 'FTM_PE')))) - sum(team == 'home' & type == 'OREB'),
		away_possessions = sum(team == 'away' & (type %in% c('2FGA', '2FGM', '3FGA', '3FGM', 'TOV') | (type %in% c('FTA_PE', 'FTM_PE')))) - sum(team == 'away' & type == 'OREB'),
		possessions = (home_possessions + away_possessions) / 2,
		home_points = sum(home_difference),
		away_points = sum(away_difference),
		home_PPP = home_points / home_possessions,
		away_PPP = away_points / away_possessions,
		net_rating = 100 * (home_PPP - away_PPP)
	) %>%
	ungroup()
```

Now we have to pivot vertically in order to have one player per row in every stint. This will allow us to place a 1/-1 to track that he is on court for a specific team. We do this after filtering for stints with at least two possessions per team, in order to reduce variance.
```R
df <- filter(df, possessions >= 2) %>%
	mutate(stint = row_number()) %>%
	select(possessions, stint, h1:a5, net_rating) %>%
	pivot_longer(cols = c(h1:a5), values_to = 'player_id', names_to = 'position') %>%
	mutate(value = ifelse(startsWith(position, 'h'), 1, -1))
```

The final step is to simpy pivot horizontally the matrix, in order to have one column per player. We filter out stints with a non-finite net rating, to avoid errors and outliers. We fill missing values with 0, which is exactly what we need to place 0 on rows where a player is not on court.

```R
df <- select(df, -position, -contains('PPP')) %>%
		filter(is.finite(net_rating)) %>%
		distinct() %>%
		pivot_wider(names_from = player_id, values_fill = 0) %>%
		select(-stint)
```

Eventually, we separate our data:
- net_rating is the target value
- possessions are the weight
- players are the coefficients to find

```R
possessions <- df$possessions
net_rating <- df$net_rating
players_matrix <-
	df %>%
		select(-net_rating, -possessions) %>%
		data.matrix()
```

Let's also, by using a utility function, compute the stats of the players. We will use them to compare differences among indicators, and to see the played minutes for every player.

```R
stats <- get_stats_from_actions(actions, groups = 'player_id', stat_types = c('traditional', 'advanced'), per = 'total') %>%
	mutate(record = W / GAMES) %>%
	select(player_id, GAMES, record, MIN, POSS, PM)

head(stats)
```

| **player_id**    | **GAMES** | **record** | **MIN** | **POSS** | **PM** |
|------------------|-----------|------------|---------|----------|--------|
| Aaron Craft      | 4         | 0          | 68.37   | 122      | -51    |
| Aaron Doornekamp | 58        | 0.41       | 1120.07 | 1930     | 95     |
| Aaron Harrison   | 27        | 0.48       | 472.47  | 826      | -35    |
| Aaron Jackson    | 12        | 0.67       | 210.1   | 376      | -42    |
| Aaron White      | 136       | 0.47       | 2867.95 | 5662     | -142   |
| Achille Polonara | 90        | 0.46       | 1989.78 | 3650     | 77     |


## APM

Adjusted Plus Minus is simply a linear model, which we can compute as:

```R
apm <- lm(formula = net_rating ~ . -possessions, data=df, weights = possessions)

apm_data <-
	broom::tidy(apm) %>%
		rename(
			player_id = term,
			APM = estimate
		) %>%
		mutate(player_id = gsub("`", "", player_id)) %>%
		slice(-1)

head(apm_data)
```

| **player_id**  | **APM** | **std.error** | **statistic** | **p.value** |
|----------------|---------|---------------|---------------|-------------|
| Aaron Craft    | 22.43   | 96.88         | 0.23          | 0.82        |
| Alen Omic      | 53.38   | 94.91         | 0.56          | 0.57        |
| Coty Clarke    | 57.42   | 95.05         | 0.6           | 0.55        |
| Earl Clark     | 50.3    | 95.14         | 0.53          | 0.6         |
| Nemanja Gordic | 48.64   | 94.95         | 0.51          | 0.61        |
| Alex Tyus      | 57.82   | 94.8          | 0.61          | 0.54        |


As we can see, there is an extremely high variance and a very high p-value, which means that we cannot reject the null hypothesis, and the results are likely due to chance.

As I mentioned before, APM is known for this high variance, even in the NBA where the sample size is way greater.

This is the reason why Regularized Adjusted Plus Minus was developed.

## RAPM

Regularized Adjusted Plus Minus is an improvement of APM based on a regularized linear regression, in particular on Ridge Regression: it is a technique which overcomes the problem of multicollinearity between variables and prevents overfitting, which means that it is perfect for our goal.

It is an extension of the traditional linear regression method by adding a penalty term, known as the shrinkage parameter, to the least squares objective function. The shrinkage parameter, represented by lambda, controls the magnitude of the coefficients by shrinking the coefficients of less important variables towards zero. This results in a model more robust to noise in the data.

We can find a suitable value for lambda by using cross validation, in particular by trying to minimize deviance. It is fundamental to set `alpha=0` in order to obtain ridge regression.

```R
library(glmnet)
lambda <- cv.glmnet(players_matrix, net_rating, alpha = 0, weights = possessions, nfolds = 10, type.measure = 'deviance')

lambda.min <- lambda$lambda.min
lambda.min
# 300.589
```

Once we find a suitable value for lamda, we can apply our regression:

```R
ridge <- glmnet(players_matrix, net_rating, weights = possessions, alpha = 0,lambda = lambda.min)

rapm_data <-
	coef(ridge,s=lambda.min) %>%
		as.matrix() %>%
		as.data.frame() %>%
		rownames_to_column('player_id') %>%
		as_tibble() %>%
		slice(-1) %>%
		rename(RAPM = s1)

head(rapm_data)
```

| **player_id**  | **RAPM** |
|----------------|----------|
| Aaron Craft    | -8.68    |
| Alen Omic      | -2.05    |
| Coty Clarke    | -1.56    |
| Earl Clark     | -3.44    |
| Nemanja Gordic | -2.41    |
| Alex Tyus      | -0.67    |

## Results

Now, we can join all of our data and draw some insights:

```R
result <-
	stats %>%
		inner_join(apm_data, by='player_id') %>%
		inner_join(rapm_data, by='player_id') %>%
		select(player_id, MIN, PM, record, POSS, APM, RAPM)

head(result)
```

| **player_id**    | **MIN** | **PM** | **record** | **POSS** | **APM** | **RAPM** |
|------------------|---------|--------|------------|----------|---------|----------|
| Aaron Craft      | 68.37   | -51    | 0          | 122      | 22.43   | -8.68    |
| Aaron Doornekamp | 1120.07 | 95     | 0.41       | 1930     | 67.05   | 1.59     |
| Aaron Harrison   | 472.47  | -35    | 0.48       | 826      | 41.84   | -3.36    |
| Aaron Jackson    | 210.1   | -42    | 0.67       | 376      | 49.8    | -1.02    |
| Aaron White      | 2867.95 | -142   | 0.47       | 5662     | 54.55   | -0.07    |
| Achille Polonara | 1989.78 | 77     | 0.46       | 3650     | 55.05   | 0.83     |


![Funnel effect between time and APM](/assets/images/articles/rapm/correlogram2.png)

As we can see, the plot with APM vs minutes played shows a very strong funnel shape, which is extremely reduced in the RAPM case. Nonetheless, let's filter out players who played less than 1000 minutes, and look at the best players:

It is also interesting to note how RAPM is less correlated with record than +/-. As I mentioned in the introduction, it is fair that a good player makes a team good, and similarly a good team has good players, which leads to a certain correlation.

Now, let's look at the results:

```R
result %>%
	filter(MIN >= 1000) %>%
	arrange(desc(RAPM)) %>%
	head(10)
```

| **player_id**    | **MIN** | **PM** | **record** | **POSS** | **APM** | **RAPM** |
|------------------|---------|--------|------------|----------|---------|----------|
| Sertac Sanli     | 1248.58 | 355    | 0.74       | 2371     | 68.67   | 3.26     |
| Dogus Balbay     | 1048.42 | 141    | 0.59       | 1768     | 68.48   | 2.79     |
| Rolands Smits    | 1501.85 | 277    | 0.68       | 2600     | 70.26   | 2.51     |
| Facundo Campazzo | 2416.03 | 586    | 0.72       | 4355     | 65.63   | 2.41     |
| Walter Tavares   | 3558.57 | 794    | 0.68       | 6284     | 72      | 2.38     |
| Alex Abrines     | 1286.68 | 246    | 0.71       | 2253     | 62.03   | 2.32     |
| Marko Guduric    | 2518.07 | 414    | 0.65       | 4305     | 67.5    | 2.22     |
| Nikola Mirotic   | 2559.63 | 524    | 0.73       | 4595     | 69.64   | 2.15     |
| Alec Peters      | 2014.32 | 292    | 0.69       | 3897     | 60.75   | 2.03     |
| Chris Singleton  | 4127.42 | 657    | 0.65       | 7707     | 68.75   | 2        |

The results are interesting: in the top 10 we have three of the best players in the recent history of Euroleague, such as Campazzo, Tavares and Mirotic, and a lot of "solid" role players. This actually makes sense, since a player does not provide value to his team only by filling his boxscore, but also through effort and other actions that do not fall in the tracked statistics.

## Improvements of the model

This model works fine, but I think that we could add a few tweaks to improve his power: in fact, at the moment each possession has the same weight, which is not the case in basketball.

At the same time, having play-by-plays allows to segment offensive and defensive RAPM, to evaluate the impact each player has on both sides of the court separately.

We can do this by recreating our dataframe:

```R
df2 <-
	actions %>%
		inner_join(games, by = c('season_id', 'edition_id', 'game_id'), suffix = c('', '_game')) %>%
		group_by(game_id) %>%
		mutate(
			team = case_when(
				team_id == home_team_id ~ 'home',
				team_id == away_team_id ~ 'away',
				TRUE ~ NA_character_),
		    FT_PE = grepl('.*2/2.*', details) | grepl('.*3/3.*', details),
		    type = ifelse(type %in% c('FTA', 'FTM') & FT_PE, paste0(type, '_PE'), type),
		    home_difference = home_score - lag(home_score, 1, 0),
		    away_difference = away_score - lag(away_score, 1, 0),
			score_difference = abs(lag(home_score - away_score)),
			game_type = ifelse(type_game == 'PO', 2, 1),
			game_phase = case_when(
				period == 4 & score_difference >= 20 ~ 0.5,
				(period == 4 & remaining_period_time <= 180 & score_difference <= 5) | period >= 5 ~ 1.5,
				TRUE ~ 1
			)
		) %>%
		select(game_type, game_phase, game_id, type, team, home_difference, away_difference, h1:a5) %>%
		group_by(game_type, game_phase, h1, h2, h3, h4, h5, a1, a2, a3, a4, a5) %>%
		summarise(
			home_possessions = sum(team == 'home' & (type %in% c('2FGA', '2FGM', '3FGA', '3FGM', 'TOV') | (type %in% c('FTA_PE', 'FTM_PE')))) - sum(team == 'home' & type == 'OREB'),
			away_possessions = sum(team == 'away' & (type %in% c('2FGA', '2FGM', '3FGA', '3FGM', 'TOV') | (type %in% c('FTA_PE', 'FTM_PE')))) - sum(team == 'away' & type == 'OREB'),
			home_points = sum(home_difference),
			away_points = sum(away_difference),
			home_offrtg = 100 * home_points / home_possessions,
			away_offrtg = 100 * away_points / away_possessions,
		) %>%
		ungroup() %>%
		pivot_longer(cols=c('home_possessions', 'away_possessions', 'away_points', 'home_points', 'home_offrtg', 'away_offrtg'), names_sep = "\\_", names_to = c("team", "stat")) %>%
		pivot_wider(names_from = 'stat') %>%
		filter(possessions >= 1) %>%
		mutate(stint = row_number()) %>%
		pivot_longer(cols = c(h1:a5), values_to = 'player_id', names_to = 'position') %>%
		mutate(
			side = ifelse(substr(team, 1, 1) == substr(position, 1, 1), 'offense', 'defense'),
			player_id = paste(player_id, side, sep=','),
			value = ifelse(side == 'offense', 1, -1)
		) %>%
		select(-position, -points, -side, -team) %>%
		filter(is.finite(offrtg)) %>%
		distinct() %>%
		pivot_wider(names_from = player_id, values_fill = 0) %>%
		select(-stint)
```
The difference we have now is that we have 2 new columns, one for the game_type (2x weight if the game is a playoffs game), and one for game phase, namely 0.5 if the action is during garbage time (20 or more points of difference in the 4th quarter), 1.5 if the action is in the clutch (namely, overtime or less than 5 points of difference withing the last 3 minutes of te 4th period), 1 otherwise.

The other difference is that the same stint is repeated twice in the dataset, one time for the home offense and one time for the away offense. We also have twice as many columns, one for the offense and one for the defense, in fact for every player there are the columns `player,offense` and `player,defense`. At the end of the computation we will split offense and defense for each player.

Since the `glmnet` function does not support multiple weight vectors, we create a single vector as the product of `possessions`, `game_type` and `game_phase`, then proceed with the same steps as before.

```R
net_rating <- df2$offrtg

players_matrix <-
	df2 %>%
		select(-offrtg, -possessions, -game_type, -game_phase) %>%
		data.matrix()

weights <-
	df2 %>%
		mutate(weight = possessions * game_type * game_phase) %>%
		select(weight) %>%
		pull()

lambda <- cv.glmnet(players_matrix, net_rating, alpha=0, weights=weights, nfolds=5, type.measure = 'deviance')

lambda.min <- lambda$lambda.min
lambda.min
# 243.7
```

Now we can find the values for both side, and compute the RAPM as the sum of offensive RAPM (ORAPM) and defensive RAPM (DRAPM). Differently from the traditional format of defensive ratings, where negative or low values are associated to good performances, in this case a positive value means a good impact, since it represent the points per 100 possessions that the player prevented from scoring.

```R
ridge2 <- glmnet(players_matrix, net_rating,
				family='gaussian',
				weights=weights, alpha=0,lambda=lambda.min) #run the ridge regression. x is the matrix of independent variables, Marg is the dependent variable, Poss are the weights. alpha=0 indicates the ridge penalty.


ramp_data2 <-
	ridge2 %>%
		coef(s = lambda.min) %>%
		as.matrix() %>%
		as.data.frame() %>%
		rownames_to_column('player_id') %>%
		as_tibble() %>%
		slice(-1) %>%
		rename(RAPM = s1) %>%
		mutate(
			side = sub(".*,", "", player_id),
			player_id = sub(",.*", "", player_id),
			stat = ifelse(side == 'offense', 'ORAPM', 'DRAPM')
		) %>%
		select(-side) %>%
		pivot_wider(names_from = 'stat', values_from = 'RAPM') %>%
		mutate(RAPM = ORAPM + DRAPM)

result2 <-
	inner_join(rapm_data2, stats, by='player_id') %>%
	mutate(MINxGP = MIN / GAMES) %>%
	select(player_id, GAMES, record, MIN, POSS, MINxGP, PM, ORAPM, DRAPM, RAPM) %>%
	filter(MIN >= 1000) %>%
	arrange(desc(RAPM))

head(result2, 10)
```

| **player_id**    | **GAMES** | **record** | **MIN** | **POSS** | **MINxGP** | **PM** | **ORAPM** | **DRAPM** | **RAPM** |
|------------------|-----------|------------|---------|----------|------------|--------|-----------|-----------|----------|
| Dogus Balbay     | 113       | 0.59       | 1048.42 | 1768     | 9.28       | 141    | 1.49      | 2.21      | 3.69     |
| Sertac Sanli     | 96        | 0.74       | 1248.58 | 2371     | 13.01      | 355    | 1.83      | 0.97      | 2.79     |
| Walter Tavares   | 162       | 0.68       | 3558.57 | 6284     | 21.97      | 794    | 0.35      | 2.1       | 2.45     |
| Facundo Campazzo | 102       | 0.72       | 2416.03 | 4355     | 23.69      | 586    | 0.97      | 1.47      | 2.44     |
| Rolands Smits    | 107       | 0.68       | 1501.85 | 2600     | 14.04      | 277    | 0.19      | 1.87      | 2.06     |
| Alex Abrines     | 77        | 0.71       | 1286.68 | 2253     | 16.71      | 246    | 0.5       | 1.53      | 2.03     |
| Moustapha Fall   | 68        | 0.51       | 1531.95 | 2728     | 22.53      | 185    | 0.76      | 1.13      | 1.9      |
| Marko Guduric    | 121       | 0.65       | 2518.07 | 4305     | 20.81      | 414    | 1.33      | 0.55      | 1.88     |
| Chris Singleton  | 174       | 0.65       | 4127.42 | 7707     | 23.72      | 657    | 0.81      | 1.07      | 1.88     |
| Daniel Hackett   | 135       | 0.64       | 2863.98 | 5046     | 21.21      | 451    | 1.07      | 0.79      | 1.85     |


We can see how the players are more or the less the same, despite some changes in the ranking. Interestingly, 4 out of the first 6 players played on average less than 20 minutes per game.

In this case, we can see in which side of the court players add most of their value: there are some defnesive specialists, such as the two-time Defensive Player of The Year Walter Tavares, or an offensive specialist as Marko Guduric, or two-way players such as Balbay, Sanly and Singleton.

![Players RAPM](/assets/images/articles/rapm/players.png)


Yet, the leader of the board Balbay is quite a surprise, since he plays only 9 minutes per game and, even if he is a solid player, is not considered such an impactful player. This may be

We can also look at the best players on the offensive side:

```R
result2 %>%
	arrange(desc(ORAPM)) %>%
	head()
```

| **player_id**   | **GAMES** | **record** | **MIN** | **POSS** | **MINxGP** | **PM** | **ORAPM** | **DRAPM** | **RAPM** |
|-----------------|-----------|------------|---------|----------|------------|--------|-----------|-----------|----------|
| Sertac Sanli    | 96        | 0.74       | 1248.58 | 2371     | 13.01      | 355    | 1.83      | 0.97      | 2.79     |
| Dogus Balbay    | 113       | 0.59       | 1048.42 | 1768     | 9.28       | 141    | 1.49      | 2.21      | 3.69     |
| Will Clyburn    | 126       | 0.76       | 3351.32 | 5946     | 26.6       | 480    | 1.42      | -0.06     | 1.36     |
| Marko Guduric   | 121       | 0.65       | 2518.07 | 4305     | 20.81      | 414    | 1.33      | 0.55      | 1.88     |
| Krunoslav Simon | 151       | 0.61       | 3759.5  | 6625     | 24.9       | 516    | 1.24      | -0.06     | 1.17     |
| Alec Peters     | 108       | 0.69       | 2014.32 | 3897     | 18.65      | 292    | 1.21      | 0.12      | 1.33     |
| Gustavo Ayon    | 75        | 0.59       | 1584.9  | 2859     | 21.13      | 139    | 1.11      | 0.08      | 1.19     |
| Nikita Kurbanov | 161       | 0.71       | 3151.02 | 5549     | 19.57      | 443    | 1.1       | 0.32      | 1.42     |
| Semen Antonov   | 125       | 0.73       | 1206.93 | 2125     | 9.66       | 161    | 1.09      | 0.54      | 1.63     |
| Daniel Hackett  | 135       | 0.64       | 2863.98 | 5046     | 21.21      | 451    | 1.07      | 0.79      | 1.85     |

Where we can see Will Clyburn, one of the best scorers of the league, and of the "clutchest" players.

On the other side, the players with the highest defensive RAPM are:

```R
result2 %>%
	arrange(desc(DRAPM)) %>%
	head()
```

| **player_id**    | **GAMES** | **record** | **MIN** | **POSS** | **MINxGP** | **PM** | **ORAPM** | **DRAPM** | **RAPM** |
|------------------|-----------|------------|---------|----------|------------|--------|-----------|-----------|----------|
| Dogus Balbay     | 113       | 0.59       | 1048.42 | 1768     | 9.28       | 141    | 1.49      | 2.21      | 3.69     |
| Walter Tavares   | 162       | 0.68       | 3558.57 | 6284     | 21.97      | 794    | 0.35      | 2.1       | 2.45     |
| Rolands Smits    | 107       | 0.68       | 1501.85 | 2600     | 14.04      | 277    | 0.19      | 1.87      | 2.06     |
| Joel Bolomboy    | 103       | 0.67       | 1504.37 | 2665     | 14.61      | 179    | -0.6      | 1.8       | 1.2      |
| Nick Weiler-Babb | 62        | 0.55       | 1539.87 | 2640     | 24.84      | 131    | -0.06     | 1.54      | 1.48     |
| Alex Abrines     | 77        | 0.71       | 1286.68 | 2253     | 16.71      | 246    | 0.5       | 1.53      | 2.03     |
| Facundo Campazzo | 102       | 0.72       | 2416.03 | 4355     | 23.69      | 586    | 0.97      | 1.47      | 2.44     |
| Adam Hanga       | 153       | 0.63       | 3024.32 | 5722     | 19.77      | 235    | -0.29     | 1.43      | 1.14     |
| Moustapha Fall   | 68        | 0.51       | 1531.95 | 2728     | 22.53      | 185    | 0.76      | 1.13      | 1.9      |
| Elijah Bryant    | 97        | 0.57       | 2012.8  | 3768     | 20.75      | 137    | -0.16     | 1.13      | 0.97     |

## Conclusions

Overall, I am satisfied with the results: there are some unexpected values, specially among low usage players, but overall I think that the model effectively managed to measure the impact of players, despite what the boxscore tell. In fact, the insight of this metric is not how much does a player contributes to his team given his points or his defense, but rather his contribution to his team regardless of the teammates he shares the court which.

I think that the results are valid enough to be used as a base for tuning other NBA-oriented metrics to NBA basketball, by finding how are certain metrics correlated with RAPM, and thus the relevance they have in the Euroleague rather than in the NBA.

### Future developments

A next step could be to include prior knowledge about players, such as their boxscore stats, age and role.

Also, I aim to develop an unsupervised clustering method to segment players according to their role on the court, and computing RAPM by cluster rather than by player. This could also lead to evaluating lineup fit according to the available players in a roster.

## Bibliography

- [https://basketballstat.home.blog/2019/08/14/regularized-adjusted-plus-minus-rapm/](https://basketballstat.home.blog/2019/08/14/regularized-adjusted-plus-minus-rapm/)
- [https://squared2020.com/2017/09/18/deep-dive-on-regularized-adjusted-plus-minus-i-introductory-example/](https://squared2020.com/2017/09/18/deep-dive-on-regularized-adjusted-plus-minus-i-introductory-example/)
