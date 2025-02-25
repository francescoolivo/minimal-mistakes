---
title: "Sdeng: the SQL database"
categories:
- "Sdeng"
gallery:
- url: /assets/images/sdeng_model.png
  image_path: /assets/images/sdeng_model.png
  alt: "database model"
  title: "SDENG database model"
---

Welcome back! Last time we analyzed and understood the main entities involved in our model. Today we will develop the SQL database that will store the data. The juicy part is definitely coming, so without further ado, let's get started.

The first thing to do is to set up the SQL database. The last time I used SQL, the Coronavirus was only a remote voice from China, so forgive me if my code is not the cleanest.

For the same reason, I will use the SQL dialect I am most comfortable with, which is MySQL. About the topic, I want to make a couple of considerations about the two SQL dialects that I considered, MySQL and SQLite.

As you have probably guessed from the name, SQLite is lightweight and portable. It requires almost no configuration and allows us to be operative within minutes. Yet, it offers little flexibility as far as types are concerned, and it has some performance issues when dealing with bigger databases.

On the other hand, MySQL requires setting up a server, an account, and permissions, thus being more complex. Nonetheless, this complexity has its pros, since it allows us to manage more data types and higher loads.
Luckily for us, switching from one dialect to the other is straightforward, and there are many online tools to help us achieve the goal.

---

Now, an important consideration is due: the goal of a relational database is to store millions of entries and to separate the view from the model.
Consider the table containing the players on the field in every action: an NBA game as more or less 500 actions, in every of each 10 players are on the court. If you consider that NBA regular season alone has 1230 games, this means that for a single season we have more than 5 million rows in this table.

Thus, the database must be efficient. For this purpose, every table will have as primary key an autogenerated numerical id. Of course, this leads to a tradeoff in terms of readability, since it is much harder to understand the entities involved when they are represented by a number.

Nonetheless, we can separate the low level, where we use the numerical id,  from the presentation level, where we display the entity's full details, e.g. the full name for players. This also leads to more complex queries, since we have to join more tables to fetch the correct information. Yet, the advantages that this strategy provides in terms of efficiency make it worthy to use it.

---

Let's dive into the database. I will not go into details on how to install and configure a MySQL server, many people already did that and in a much better way than I would ever be able to do. I suggest this.

Let's start from creating the MySQL database.

```sql
CREATE DATABASE sdeng;
```

I named it Sdeng, but you can name it as you prefer.

From now on, I will assume that you have all permissions on this database. If you haven't, you can add them by logging into MySQL as root and type

```sql
GRANT ALL ON sdeng.* TO 'user'@'localhost';
```

Be sure to replace the database name and username with the ones you chose.

## Tables

### League

```sql
CREATE TABLE leagues (
    id              INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    full_name       VARCHAR(50)          NOT NULL,
    name            VARCHAR(10)          NOT NULL,
    website         TINYTEXT             NULL,
    foundation_year YEAR                 NULL,
    international   TINYINT(1) DEFAULT 0 NULL,
    
    CONSTRAINT leagues_full_name_uq
        UNIQUE (full_name),
    CONSTRAINT leagues_name_uq
        UNIQUE (name)
)
```

The table is straightforward: the ```international``` flag is a boolean value, which we will set for the Olympics or the World Cup.

### Season

```sql
CREATE TABLE seasons (
    id            INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    starting_year YEAR NOT NULL,
    ending_year   YEAR NOT NULL,
    CONSTRAINT years_uq
        UNIQUE (starting_year, ending_year),
    CONSTRAINT seasons_years_chk
        CHECK (`ending_year` >= `starting_year`)
)
```

This is even more straightforward: notice how the uniqueness of the season is given by the couple ```(starting_year, ending_year)```; thus, we can store data from a League which starts in the spring and ends in autumn. This is frequent in Soccer competitions from Southern Hemisphere, so let's consider the case.

I decided to use a year and not a date because, even if usually a season starts on the first of July and ends on the thirty of June, Covid-19 outbreak or lockouts may change the dates, and it would not make sense to create a different season only for NBA, since it would make much harder to compare leagues.

### Franchise

```sql
CREATE TABLE franchises (
    id              INT UNSIGNED AUTO_INCREMENT
    PRIMARY KEY,
    name            VARCHAR(50)          NOT NULL,
    city            TINYTEXT             NOT NULL,
    country         TINYTEXT             NOT NULL,
    foundation_year YEAR                 NULL,
    national_team   TINYINT(1) DEFAULT 0 NULL,
    CONSTRAINT franchises_name_uq
        UNIQUE (name)
)
```

The considerations for this table are similar to the ones for the leagues' table.

### Players and Managers

Here come the first difficulties: to model the differences between players and managers I decided to use inheritance: the parent table is the "persons" table, where we store personal information.

```sql
CREATE TABLE persons (
    id            INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    full_name     VARCHAR(100)     NOT NULL,
    name          TINYTEXT         NOT NULL,
    middle_name   TINYTEXT         NULL,
    surname       TINYTEXT         NOT NULL,
    birthday      DATE             NOT NULL,
    birth_city    TINYTEXT         NOT NULL,
    birth_country TINYTEXT         NOT NULL,
    sex           CHAR DEFAULT 'M' NOT NULL,
    nationality_1 TINYTEXT         NULL,
    nationality_2 TINYTEXT         NULL,
    image_url     TINYTEXT         NULL,
    
    CONSTRAINT persons_uq
        UNIQUE (full_name, birthday, sex)
);
```

We then have a `players` table and a `managers` table: both of them inherit from `persons` using a foreign key.

```sql
CREATE TABLE players (
    id             INT UNSIGNED  NOT NULL PRIMARY KEY,
    main_role      VARCHAR(2)    NOT NULL,
    secondary_role VARCHAR(2)    NULL,
    height         DECIMAL(3, 2) NULL COMMENT 'unit of measure is meters',
    weight         DECIMAL(4, 1) NULL COMMENT 'unit of measure is kgs',
    wingspan       DECIMAL(3, 2) NULL COMMENT 'unit of measure is meters',
    retired        TINYINT(1)    NULL,
    
    CONSTRAINT players_persons_fk
        FOREIGN KEY (id) REFERENCES persons(id)
);
```
```sql
CREATE TABLE managers (
    id INT UNSIGNED NOT NULL PRIMARY KEY,
    
    CONSTRAINT managers_persons_fk
        FOREIGN KEY (id) REFERENCES persons(id)
)
```

As a general rule I will only use the metrical system since honestly, I find the imperial system gibberish.

### Team

```sql
CREATE TABLE teams (
    id           INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    franchise_id INT UNSIGNED            NULL,
    season_id    INT UNSIGNED            NOT NULL,
    type         VARCHAR(10) DEFAULT 'M' NOT NULL,
    name         VARCHAR(100)            NOT NULL,
    abbreviation VARCHAR(3)              NOT NULL,
    
    CONSTRAINT teams_name_uq
        UNIQUE (name, season_id, type),
    CONSTRAINT teams_uq
        UNIQUE (franchise_id, season_id, type),
    CONSTRAINT teams_franchises_fk
        FOREIGN KEY (franchise_id) REFERENCES franchises(id),
    CONSTRAINT teams_seasons_fk
        FOREIGN KEY (season_id) REFERENCES seasons(id)
)
```
A team is uniquely defined by the triple ("franchise", "season", "type"). This allows franchises not only to have a male (`M`) or female (`F`) team, but also youth teams, which is very common in Europe. For instance this attribute would be `U18M` for the male under-18 team.

Notice how franchise can be null: this should help us to fetch teams from websites where the franchise is not easily obtainable, allowing us to collect the franchise later using different techniques or websites.

### Roster and Staff

To model rosters and staffs I used two tables, one for players and one for managers. They include information about contracts, following the logic that I described in the previous post.

```sql
CREATE TABLE rosters (
  id            INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  player_id     INT UNSIGNED     NOT NULL,
  team_id       INT UNSIGNED     NOT NULL,
  season_id     INT UNSIGNED     NOT NULL,
  jersey_number TINYINT UNSIGNED NULL,
  amount        DECIMAL(11, 2)   NULL,
  type          TINYTEXT         NULL,
  start_date    DATE             NULL,
  end_date      DATE             NULL,
  
  CONSTRAINT rosters_players_fk
    FOREIGN KEY (player_id) REFERENCES players(id),
  CONSTRAINT rosters_seasons_fk
    FOREIGN KEY (season_id) REFERENCES seasons(id),
  CONSTRAINT rosters_teams_fk
    FOREIGN KEY (team_id) REFERENCES teams(id)
)
```

```sql
CREATE TABLE staffs (
  id         INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  manager_id INT UNSIGNED   NOT NULL,
  team_id    INT UNSIGNED   NOT NULL,
  season_id  INT UNSIGNED   NOT NULL,
  role       TINYTEXT       NOT NULL,
  amount     DECIMAL(11, 2) NULL,
  type       TINYTEXT       NULL,
  start_date DATE           NULL,
  end_date   DATE           NULL,
  
  CONSTRAINT staffs_players_fk
    FOREIGN KEY (manager_id) REFERENCES managers(id),
  CONSTRAINT staffs_seasons_fk
    FOREIGN KEY (season_id) REFERENCES seasons(id),
  CONSTRAINT staffs_teams_fk
    FOREIGN KEY (team_id) REFERENCES teams(id)
)
```

The two tables are almost identical, the only differences are that "rosters" references "players" while "staffs" references "managers" and the "role" attribute for staff members: it represents the role inside the team, for instance "General manager", "Coach", "Assistant coach" etc.

### Edition
```sql
CREATE TABLE editions (
    id                  INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    league_id           INT UNSIGNED                    NOT NULL,
    season_id           INT UNSIGNED                    NOT NULL,
    number_of_teams     SMALLINT UNSIGNED               NULL,
    concluded           TINYINT(1)                      NULL,
    winner_id           INT UNSIGNED                    NULL,
    number_of_periods   TINYINT UNSIGNED  DEFAULT '4'   NULL,
    period_duration     SMALLINT UNSIGNED DEFAULT '600' NULL COMMENT 'unit of measure is seconds',
    shot_clock_duration SMALLINT UNSIGNED DEFAULT '24'  NULL COMMENT 'unit of measure is seconds',
    overtime_duration   SMALLINT UNSIGNED DEFAULT '300' NULL COMMENT 'unit of measure is seconds',
    
    CONSTRAINT editions_uq
        UNIQUE (league_id, season_id),
    CONSTRAINT editions_leagues_id_fk
        FOREIGN KEY (league_id) REFERENCES leagues(id),
    CONSTRAINT editions_seasons_id_fk
        FOREIGN KEY (season_id) REFERENCES seasons(id),
    CONSTRAINT editions_teams_winner_id_fk
        FOREIGN KEY (winner_id) REFERENCES teams(id)
)
```
The `editions` table models what we discussed in the previous chapter. Notice that apart from the league and the season, all other attributes are optional. This allows us to create an edition even without knowing some parameters, that can be easily added in a second moment. As default values I used FIBA rules, since they are used in most leagues.

```sql
CREATE TABLE conferences (
    id         INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    edition_id INT UNSIGNED NOT NULL,
    name       VARCHAR(50)  NOT NULL,
    
    CONSTRAINT conferences_uq
        UNIQUE (edition_id, name),
    CONSTRAINT conferences_editions_id_fk
        FOREIGN KEY (edition_id) REFERENCES editions(id)
)
```
The `conferences` table allows us to track all conferences (or groups) of a league.

```sql
CREATE TABLE edition_participants (
    id            INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    team_id       INT UNSIGNED NOT NULL,
    edition_id    INT UNSIGNED NOT NULL,
    conference_id INT UNSIGNED NULL,
    
    CONSTRAINT edition_participants_uq
        UNIQUE (team_id, edition_id),
    CONSTRAINT edition_participants_conferences_id_fk
        FOREIGN KEY (conference_id) REFERENCES conferences(id),
    CONSTRAINT edition_participants_editions_id_fk
        FOREIGN KEY (edition_id) REFERENCES editions(id),
    CONSTRAINT edition_participants_teams_id_fk
        FOREIGN KEY (team_id) REFERENCES teams(id)
)
```
This junction-table tracks which teams participate to a league edition. Notice that the unique constraint on the couple("team", "edition") has some drawbacks, consider the 2020–21 Eurocup format: regular season consisted of four groups of six teams each; at the end of the RS the best four teams of every group moved to the Top16, where they were sorted into four new groups of four teams each. The two best teams then moved to play-offs.
Apart from the incredible complexity of this format, notice that our database would not be able to grasp the fact that a team can be part of more than one conference/group during the same league edition. Yet, I believe that although it is our model that must follow reality, and not vice-versa, we don't get any advantage by strictly representing this particular format, even more so considering that this format is not used anymore. We therefore use a more strict condition, which applies in the vast majority of cases.

### Awards and Trophies

```sql
CREATE TABLE trophies (
    id         INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    edition_id INT UNSIGNED         NOT NULL,
    name       VARCHAR(50)          NOT NULL,
    individual TINYINT(1) DEFAULT 0 NULL,
    
    CONSTRAINT trophies_uq
        UNIQUE (edition_id, name),
    CONSTRAINT trophies_editions_id_fk
        FOREIGN KEY (edition_id) REFERENCES editions(id)
)
```

This simple table allows us to store the trophies. We then use a junction table to join trophies, teams and persons:

```sql
CREATE TABLE trophy_winners (
    id        INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    trophy_id INT UNSIGNED NOT NULL,
    team_id   INT UNSIGNED NOT NULL,
    person_id INT UNSIGNED NULL,
    
    CONSTRAINT trophy_winners_persons_id_fk
        FOREIGN KEY (person_id) REFERENCES persons(id),
    CONSTRAINT trophy_winners_teams_id_fk
        FOREIGN KEY (team_id) REFERENCES teams(id),
    CONSTRAINT trophy_winners_trophies_id_fk
        FOREIGN KEY (trophy_id) REFERENCES trophies(id)
)
```

### Game

```sql
CREATE TABLE games (
    id                  INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    edition_id          INT UNSIGNED                      NOT NULL,
    start_time          DATETIME                          NOT NULL,
    type                VARCHAR(20)      DEFAULT 'RS'     NOT NULL,
    phase               VARCHAR(50)                       NULL,
    round               SMALLINT UNSIGNED                 NULL,
    state               VARCHAR(20)      DEFAULT 'played' NOT NULL,
    home_team_id        INT UNSIGNED                      NOT NULL,
    away_team_id        INT UNSIGNED                      NOT NULL,
    home_score          SMALLINT UNSIGNED                 NULL,
    away_score          SMALLINT UNSIGNED                 NULL,
    number_of_overtimes TINYINT UNSIGNED DEFAULT '0'      NULL,
    arena               TINYTEXT                          NULL,
    arena_capacity      SMALLINT UNSIGNED                 NULL,
    attendance          SMALLINT UNSIGNED                 NULL,
    
    CONSTRAINT games_uq
        UNIQUE (home_team_id, away_team_id, start_time),
    CONSTRAINT away_team_id_fk
        FOREIGN KEY (away_team_id) REFERENCES teams(id),
    CONSTRAINT games_editions_id_fk
        FOREIGN KEY (edition_id) REFERENCES editions(id),
    CONSTRAINT home_team_id_fk
        FOREIGN KEY (home_team_id) REFERENCES teams(id)
)
```

Plenty of fields here, yet all of them are quite self-explaining. I decided not to use a dictionary of allowed for "type" and "state" because I don't have yet considered all possible values, and I don't want to limit myself right now. I will probably add a constraint in the next weeks. For instance, some type values could be "RS" (Regular Season), "PO" (Play-off), "PI" (Play-in), "TOP16", "F4" (Final Four) etc. Some state values could be "Finished", "Suspended", "Postponed", "Canceled", "To be played" etc.

### Action

Also in this case the fields are self-explaining, and were all analyzed in the previous chapter.

```sql
CREATE TABLE actions (
    id                    INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    game_id               INT UNSIGNED           NOT NULL,
    period                TINYINT UNSIGNED NOT NULL,
    home_score            SMALLINT               NULL,
    away_score            SMALLINT               NULL,
    remaining_period_time DECIMAL(5, 1) UNSIGNED NOT NULL COMMENT 'unit of measure is seconds',
    description           VARCHAR(200)           NULL,
    main_type             VARCHAR(30)            NOT NULL,
    secondary_type        VARCHAR(50)            NULL,
    player_id             INT UNSIGNED           NULL,
    team_id               INT UNSIGNED           NULL,
    x                     DECIMAL(5, 1)          NULL,
    y                     DECIMAL(5, 1)          NULL,
    linked_action_id      INT UNSIGNED           NULL,
    
    CONSTRAINT actions_games_id_fk
        FOREIGN KEY (game_id) REFERENCES games(id),
    CONSTRAINT linked_action_id
        FOREIGN KEY (linked_action_id) REFERENCES actions(id),
    CONSTRAINT player_fk
        FOREIGN KEY (player_id) REFERENCES players(id),
    CONSTRAINT team_fk
        FOREIGN KEY (team_id) REFERENCES teams(id)
)
```

To track the players on court we use a junction table between "actions", "players" and "teams". This does not allow to model that there must be exactly ten players on court, but is way more flexible and query-prone than storing then fields in the "actions" table.

```sql
CREATE TABLE actions_players (
    action_id INT UNSIGNED NOT NULL,
    team_id   INT UNSIGNED NOT NULL,
    player_id INT UNSIGNED NOT NULL,
    
    CONSTRAINT actions_players_uq
        UNIQUE (action_id, player_id, team_id),
    CONSTRAINT actions_players_actions_id_fk
        FOREIGN KEY (action_id) REFERENCES actions(id),
    CONSTRAINT actions_players_players_id_fk
        FOREIGN KEY (player_id) REFERENCES players(id),
    CONSTRAINT actions_players_teams_id_fk
        FOREIGN KEY (team_id) REFERENCES teams(id)
)
```

### Draft

To collect draft information, we will use a separate table to collect all possible data:

```sql
CREATE TABLE drafts (
    season_id         INT UNSIGNED     NOT NULL,
    player_id         INT UNSIGNED     NOT NULL,
    franchise_id      INT UNSIGNED     NOT NULL,
    round             TINYINT UNSIGNED NULL,
    pick              TINYINT UNSIGNED NULL,
    team_of_origin_id INT UNSIGNED     NULL,
    
    CONSTRAINT drafts_player_id_uindex
        UNIQUE (player_id),
    CONSTRAINT drafts_franchises_id_fk
        FOREIGN KEY (franchise_id) REFERENCES franchises(id),
    CONSTRAINT drafts_franchises_id_fk_2
        FOREIGN KEY (team_of_origin_id) REFERENCES franchises(id),
    CONSTRAINT drafts_players_id_fk
        FOREIGN KEY (player_id) REFERENCES players(id),
    CONSTRAINT drafts_seasons_id_fk
        FOREIGN KEY (season_id) REFERENCES seasons(id),
    CONSTRAINT pick_chk
        CHECK ((`pick` >= 1) AND (`pick` <= 30)),
    CONSTRAINT round_chk
        CHECK ((`round` = 1) OR (`round` = 2))
);
```

### Referees

A few weeks ago I read a brilliant analysis by Owen Phillips about how referees stopped punishing defensive 3 seconds violation, and how different referees followed this rules. I found it brilliant, and I would like to replicate it, so I will track referees data and which referee whistles a specific foul, where available.

Now, tracking referees may lead to some obvious analysis, such as if a certain referee is correlated with the results of a specific team. I was a soccer referee for a long time, and I firmly believe in the good faith of referees, so you won't find such analyses here.

I won't use the "persons" table, although I can guarantee you that referees are persons too, since it's much harder to collect personal data for refs. Therefore, we will only use the strictly essentials name and surname.

```sql
CREATE TABLE referees
(
    id      INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    surname VARCHAR(50) NOT NULL,
    name    VARCHAR(50) NULL,
    
    CONSTRAINT referees_uq
        UNIQUE (surname, name)
);
```

### Action details

To add a layer of depth to the actions, the best way is to use some junction tables. Junction tables allow us to add details to an action without the need of storing an attribute in advance.

I will use a junction table to add details and flags to an action (off turnover, fast break, etc.), one for shots and one for fouls.

As far as action details, I could have done it by using an attribute in the action itself and creating a list of comma separated flags, but I reckon that this strategy is less efficient than a join.

As far as shots are concerned, notice that the coordinates remain in the action, and not in the shot. This is due to the fact that coordinates may also be available for turnovers or steals, not only for shots.

To store all the possible values we create dictionary table, to guarantee that only allowed values are added to the tables.

```sql
CREATE TABLE action_types_dictionary (
    id          INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name        VARCHAR(10) NOT NULL,
    description TINYTEXT    NULL,
    
    CONSTRAINT action_types_dictionary_name_uindex
        UNIQUE (name)
)

CREATE TABLE action_types (
    action_id INT UNSIGNED NOT NULL,
    type_id   INT UNSIGNED NULL,
    
    CONSTRAINT action_types_action_id_action_type_id_uindex
        UNIQUE (action_id, type_id),
    CONSTRAINT action_types_action_types_dictionary_id_fk
        FOREIGN KEY (type_id) REFERENCES action_types_dictionary(id),
    CONSTRAINT action_types_actions_id_fk
        FOREIGN KEY (action_id) REFERENCES actions(id)
)
```

```sql
CREATE TABLE shot_types_dictionary (
    id          INT UNSIGNED AUTO_INCREMENT
        PRIMARY KEY,
    name        VARCHAR(10) NOT NULL,
    description TINYTEXT    NULL,
    
    CONSTRAINT shot_types_dictionary_name_uq
        UNIQUE (name)
)

CREATE TABLE shot_types (
    shot_id INT UNSIGNED NOT NULL,
    type_id INT UNSIGNED NULL,
    
    CONSTRAINT shot_id_type_id_uq
        UNIQUE (shot_id, type_id),
    CONSTRAINT shot_types_shot_id_fk
        FOREIGN KEY (shot_id) REFERENCES actions(id),
    CONSTRAINT shot_types_shot_types_dictionary_id_fk
        FOREIGN KEY (type_id) REFERENCES action_types_dictionary(id)
)
```

```sql
CREATE TABLE foul_types_dictionary (
    id          INT UNSIGNED AUTO_INCREMENT
        PRIMARY KEY,
    name        VARCHAR(10) NOT NULL,
    description TINYTEXT    NULL,
    
    CONSTRAINT foul_types_dictionary_name_uq
        UNIQUE (name)
)

CREATE TABLE foul_types (
    foul_id    INT UNSIGNED NOT NULL,
    type_id    INT UNSIGNED NULL,
    referee_id INT UNSIGNED NULL,
    
    CONSTRAINT foul_id_type_id_uq
        UNIQUE (foul_id, type_id),
    CONSTRAINT foul_types_foul_id_fk
        FOREIGN KEY (foul_id) REFERENCES actions(id),
    CONSTRAINT foul_types_foul_types_dictionary_id_fk
        FOREIGN KEY (type_id) REFERENCES foul_types_dictionary(id),
    CONSTRAINT foul_types_referees_id_fk
        FOREIGN KEY (referee_id) REFERENCES referees(id)
)
```

This is the overall schema, you can click on the image to display it to full screen.

{% include gallery caption="Database schema" %}

That's all for today, folks. In the next episode we will analyze the required function to put everything together, stay tuned!
