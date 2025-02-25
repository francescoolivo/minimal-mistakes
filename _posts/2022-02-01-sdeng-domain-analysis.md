---
title: "Sdeng: the domain analysis"
categories:
- "Sdeng"
---

Hello everyone, and welcome back to <i>Sdeng: a library for basketball data and analysis</i>. Today's chapter focuses on the domain analysis: we will make some considerations about the involved entities and the relationships among them.

## Domain analysis

Let's start with the entities involved. It's likely that I forget or don't consider some useful attribute: if you think I forgot a relevant attribute or class please feel free to comment, and I will add it.

As far as attributes are involved, it is quite easy that I forgot some, so if you think of any important attribute that I have ignored please write it in the comments and I will proceed to add it.

### League
Since we want to store data from more than a league, a reference to the league is mandatory. A league is identified by its name and has a website. Some leagues' examples are NBA, NCAA, Euroleague, LBA (Italian National League), and Eurocup. We also store the foundation year.

### Season
There is not much to say about a season, it has a starting year and an ending year. By convention, where referring to a certain year season, I am referring to its ending year: for instance, the Milwaukee Bucks won 2021s NBA Edition, meaning that they won the 2020–2021 edition.

### Franchise/Club
A franchise is the NBA equivalent of Club for Europeans. Nonetheless, there are some differences between NBA Franchises ed European Clubs. NBA Franchises may change not only their name but also their city. European Clubs on the other hand do not change their city, at least as far I know, but do often change their name, or at least part of it, due to sponsorship reasons. We are interested in storing the name, the city, and the country.

### Players and Managers
There are two kind of people, players and managers. For every person, we want to track basic personal data, such as name, surname, birthday, country and city of birth, and sex. These attributes are all we want to store for managers, while for players we also want to store the main role, the secondary role, physical information, draft information and, in future, injuries. We also store if the player is still active.

### Team
A Franchise may have up to two teams for season, a men's and a women's one. Every team has a particular name, a bunch of players and staff members (though we want to consider that a player may play for a team only for a short period of time, think about 10-days contracts in NBA, or that a coach may be waved during a season). A team takes part in one or more Championships, in particular to the Edition of the same season which identifies the team. For instance Barcelona every year takes part in the Spanish League, the Copa del Rey, the Spanish Supercup, and of course the Euroleague.

### National Teams
A mention to national teams and competitions is due. Nonetheless, since they are very similar to any other competition, a simple boolean flag will be enough to keep track of them, without adding another layer of complexity.

### Contract
As a premise, I don't want to model all NBA contract world, because it would require a whole series itself. To keep it simple, we are going to store the player, the team, the season, the contract amount, the role, the contract type, and the starting and ending dates.

For instance, during the 2021–22 season LeBron James will earn from the Los Angeles Lakers 41 million dollars and change; the role is "player", the contract type is "2 years contract" and the contract duration is from 2021/07/01 to 2022/06/30, if he isn't traded. Please not that the duration of the contract is of one year even if the contract type is "2 years". This is because I am considering that contract only within the 2021–22 season, which only lasts a year.

For simplicity's sake, I will not consider guaranteed/non-guaranteed contracts, trade bonuses etc.

<h2>Edition</h2>
Every league has many editions, and each one is different from the other ones. It isn't simply a matter of different seasons: there may be a different number of participating teams or different rules. Moreover, an edition may be concluded, and in case it is it may have a winner. Notice that not necessarily a concluded edition has a winner.

### Conferences/Groups and Divisions
NBA standings are divided into two conferences of 15 teams each, and each of them is divided into three divisions of 5 teams each, while Eurocup is divided into two groups of 10 teams each. Please notice that the situations are quite different since in NBA's Regular Season a team plays also against other conference teams, while in Eurocup RS a team only plays against teams of the same group. Nonetheless, the difference is way too subtle for our analysis, and we won't consider it.

For consistency with the Franchise/Club case, I will use the NBA naming convention also for European competitions.

### Awards and Trophies
A trophy is part of a league edition, and may be won by a team or by a player, moreover all trophies won by a team are also won by all the players in the team. Therefore we can store in a table the trophies, and information such like name, foundation year, and type, and in another table the winners and the respective season.

### Game
For a game we track the timestamp, the championship edition, the home and away team, the game type (regular season, playoff, finals), and, if already available, the scores and how many overtimes were played if any. We then track other useful information such as the referees, the arena, the arena capacity, and the attendance.

Another useful piece of information is the round: differently from the NBA, in European leagues each team plays against each other team a fixed number of times, usually two. Since this system is the most widespread in many leagues, it is convenient to allow it inside our database.

Due to Covid-19, we are now used to postponed or suspended games, therefore we record the state too.
 
---

Now we get to the main difficulty of this domain analysis: the Action entity. As I mentioned in the introduction, I am not going to use Computer Vision in this series, but since I want this code to be extendable, everything we designed up to this point would not change in case we started using CV algorithms.

Let's jump in

### Action
An Action describes anything relevant which happens on a basketball court.

To store the best set of information possible, we will now compare how the book "Basketball Data Science" from Zuccolotto and Manisera (BSD from now on), NBA, Euroleague, and LBA track this data. We take into consideration four sources because I believe that a single source may not consider some insightful attribute. By comparing them we can reach a more complete result.

Before diving into the code, please consider that this is just a sample code, if you don't understand the details about how information is retrieved it's completely normal, but we are going to get back to this in a few articles.

As a premise, please note that these code snippets' goal is exclusively to analyze the results, how results are obtained is not important at the moment. Nonetheless, I want to mention how I obtained the BDS file.

BDS develops an R library, <a href="https://github.com/sndmrc/BasketballAnalyzeR">BasketballAnalyzerR</a>. The library provides a sample dataset in .rda format, which I downloaded and converted to .csv using R. The process is quite straightforward and easily replicable.

Thus said, let's take a look at this sample dataset features:

<script src="https://gist.github.com/francescoolivo/8fe0597b2cc171f6d76ae207854672ce.js"></script>

Let's start with some considerations on the Basketball Data Science attributes. Storing all 10 players on the court (a1 … a5, h1 … h5) may be useful since it allows us to investigate the performances when two or three particular players are together on the court, or when a team is playing small-ball. `enter` and `left` may not be necessary, since we can infer substitutions by changes within the on-court players. Moreover, BSD stores an attribute for any possible action a player can do, for instance, steal, assist, or foul: I find this to be a bit redundant, and I think this can be improved.

`remaining time` and `elapsed` are complementary, therefore we can only keep one. It's important to consider that a period has different lengths according to the League, for instance, 12 minutes in NBA and 10 minutes in FIBA competitions. Moreover, `play length` can be easily computed afterward, so we don't need to store it.

Tracking both original coordinates and converted coordinates has some pros and cons: the main pro is that it allows us to change the coordinate system (origin in the half-court or in the bottom-left corner), on the other hand, if a league changes coordinate system recomputing the converted one may lead to errors.

Now let's take a look at the NBA data frame.

<script src="https://gist.github.com/francescoolivo/4e3f8f2574776a773fe024ab0aeb6094.js"></script>

Firstly, we notice how NBA uses numerical identifiers for message type and action type: by looking carefully at the code above, we can see how `EVENTMSGTYPE` represents the main type of an action, for instance, 2 represents a missed shot, 4 a defensive rebound and 5 a steal. On the other hand, ```EVENTMSGACTIONTYPE``` represents more complex details. Thanks to [this](https://github.com/swar/nba_api/blob/master/docs/examples/PlayByPlay.ipynb) amazing notebook we can obtain a more comprehensive idea of what these identifiers represent. Even though this simple example only deals with a small subset of the possible combinations, we can grasp the main concept, which is to add more details to the action using a dictionary of allowed values.Firstly, we notice how NBA uses numerical identifiers for message type and action type: by looking carefully at the code above, we can see how `EVENTMSGTYPE` represents the main type of an action, for instance, 2 represents a missed shot, 4 a defensive rebound and 5 a steal. On the other hand, `EVENTMSGACTIONTYPE` represents more complex details. Thanks to this amazing notebook we can obtain a more comprehensive idea of what these identifiers represent. Even though this simple example only deals with a small subset of the possible combinations, we can grasp the main concept, which is to add more details to the action using a dictionary of allowed values.

By the way, if you don't know about nba_api, the library from which this notebook is taken, do check it out because it is amazing.

This approach guarantees a more efficient usage of memory, since integers need way fewer bytes than strings, but the readability of a row drastically decreases.

<script src="https://gist.github.com/francescoolivo/c04d82dc9226cffcfca22cf65a13a11f.js"></script>

Another interesting point is that NBA uses three attributes to describe an action, one for the home team, one for the away team, and one for neutral events, and actions where three players are involved. Let's try to understand which cases are we dealing with:

As far as neutral events are concerned, apart from the start and the end of a period, the only interesting line is the double technical foul assigned to Durant and Embiid.

In my opinion, storing three attributes, one for the home team, one for the away team and one for neutral events has some drawbacks, both in terms of readability and in terms of query complexity. I believe that a better strategy would be to split actions as much as we can. An example of this would be to split an action involving two players into two single actions and to keep the relationship between actions by a reference to the main one. Moreover, instead of using three columns, I would use one for the player and one for the team.

For instance, consider the third action from the `NBA_pbp` notebook, where Harden steals the ball from Embiid. Instead of storing this on a single row, I would use a row for Embiid's turnover, and one for Harden's steal, with a reference to the turnover action.

This will ease queries where we don't care if a team is the home team or the away team, but will slightly complicate queries where the difference between home team and away team, such as plus-minus, are relevant. Referencing the main action in a sequence will also help to model relationships between multiple actions.

Interestingly, this is exactly what happens in the Italian League case:

<script src="https://gist.github.com/francescoolivo/08d56537c22f11e89039d38218d266b7.js"></script>

The `linked_id` attribute is exactly what I was mentioning before. The `home_club` flag is redundant, but It can be quite helpful in computing the plus-minus. The `side` flag is a useful attribute to create shot charts, but it's not fundamental.

Let's notice also that LBA too stores two action descriptions, the main one and a secondary one, in a similar way to NBA.

In our last case for today, let's look at Euroleague. Euroleague's API sends action-related data and shot-related data separately, so we must join them to analyze it in the best possible way. Please notice that although I removed features with the same name, many columns from the two frames carry the same information even if they have different names: I didn't remove them for laziness' sake.

<script src="https://gist.github.com/francescoolivo/dd699831ea130e54cc4f5d8b92ceea4c.js"></script>

We already discussed most of the features as they already appeared in the previous examples. What I'd like to point out is the `playtype` field. It basically overlaps with NBA's `EVENTMSGTYPE`, but being a string it is much more readable.

There are some useful flags such as `fast break`, `points off turnover` or `second chance` which are useful, and we should store them.

---

Now that we looked at all these storing methods, I want to make a more general observation. In all these cases we have been lucky enough to obtain pretty detailed data, with insightful attributes such as the shot coordinates. Of course, when we are dealing with NBA the quality of the data is not an issue, but we want to be able to store potentially any championship we are interested in, from NBA to Armenian 2nd Division, provided that they offer a play-by-play in their website. This requirements claims for robustness.

Consider the case where a league doesn't store shot coordinates, but only if the shot was from the paint, from the mid-range, or from behind the three-point line. This is frequent for many second-division leagues in Europe, for instance. If we only relied on coordinates, we would not be able to store this information, and storing a default coordinate is definitely not a good idea.

We therefore must consider a hierarchical way to store some attributes. This is the case with shot coordinates, but also with shot type. As we saw in the nba_api example, NBA stores dozens of shot types, most of which are a sub-case of another shot type. Our model should be able to capture this complexity, in order to represent reality in the best possible way.

All in all, the fields to track for every action are:


- A play id.
- The ten players on the court [ a1 … a5, h1 … h5 ].
- The period.
- The away score.
- The home score.
- The remaining time in the quarter.
- The player involved in the action.
- The team of the player involved in the action.
- A flag to store if the team is the home one or the away one.
- The action main type.
- The action secondary type: since there can be more than one, we might use a table to track all of them or we can join the strings using a separator.
- The main action, if present. For instance an assist always references a scored shot.
- The coordinates and/or any other convenient geographic system.


---

Summing up, we described the main entities and their attributes. We looked at some borderline cases and at different ways to store the same data.

The next step will be the creation of a MySQL database, starting from the considerations we made today. We will then develop all the necessary views to express traditional and complex statistics, and we will eventually develop the scrapers, following the good old software engineering principles.

Next chapter is coming soon, so stay tuned and subscribe to my newsletter if you don't want to miss it!
