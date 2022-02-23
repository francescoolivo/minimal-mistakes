---
title: "Sdeng: the functions design"
categories:
- "Sdeng"
---

Welcome back! Today's goal is to design the io functions that we will use to download and load data, from the web or from the database.

Let's get to work!

## Download to database

This function is the juiciest one. It will have a lot of parameters, but we will try to strictly follow the single responsibility principle.

### Parameters

We will now divide the parameters into macro-categories, which will help us to better understand how to better structure our code and what is actually needed.

#### Database related parameters

- Type of database: if the database is a MySQL database (default) or a SQLite database. We want to make this part extendable, so we must design code carefully.
- Database name: the name of the database to use
- Host: the host where the database is located (only with MySQL)
- User: the user to login into the database (only with MySQL)
- Password: the password of the user in the database. It should not be passed as an argument for security reasons, so a flag is necessary.

These parameters, nonetheless, do not actually modify the behavior of the function, and could be read from a configuration file. Of course, writing a password on a text file is not a valid security policy, but since we are not dealing with sensitive data we can deal with it.

#### Request type

- play-by-play logs: the classical play by play logs, formatted as described in the [previous chapter]({% post_url 2022-02-02-sdeng-database %}). They require to dynamically fetch and add all dependencies, such as games, editions, teams, players, etc
- contracts: information about contracts and salaries. Of course, these data are easily obtainable only from a few leagues, NBA in particular. For most international leagues, these data must be inserted manually
- awards: the winners of season awards. Similarly to contracts, only NBA awards can be obtained easily
- staffs: the staff members
- games: games information without play-by-play logs.

#### Filters

- leagues: the leagues we are interested in. A default value could be "NBA".
- seasons: the set of seasons to collect data from. Default could be current season.
- teams: a set of teams to consider withing
- date range: a range of dates to consider
- round: the round of the match, as usual in Europe
- game type: such as playoffs, play-ins, regular season etc.

### Behavior

1. Input check: verify that the database connection works properly, and check that request type and filters are allowed. Not only should be allowed individually. but also the combination. As I mentioned a few lines above, some data are available only for certain leagues, and we want to verify that before starting.
    1. Verify that the database connection works correctly
    2. Check that the request type is allowed
    3. Check that the filters are allowed
    4. Check that the combination request type/filters is valid. For instance, fetching contracts information could be possible only for NBA, or play-by-play data in a certain league may not be available for some seasons.
2. For every league, fetch the games url corresponding to the input filters. This can be done by looking at the calendar page of a league, from which game ids are collected 
3. Check that the league is in the database, and add it otherwise
4. Check that the edition of the league is in the database, and add it otherwise
5. Check that the club associated with a team is in the database, and add it otherwise
6. Check that the teams involved in a match are in the database, and add them otherwise
7. If absent, insert the games-related information in the database, and store the mapping between website game id and database game id
8. Download the play by play data
9. Collect the starting fives
10. For every player involved in an action, check if he already exists in the database. If he does, save the mapping between player's name and database player id, if he doesn't, insert it and store the mapping
11. Transform the play-by-play following the database format. Give a temporary index to the action ids
12. Update on court players on substitutions
13. Iteratively add actions to the database obtaining the database id. Once collected, update all tables referencing that action




