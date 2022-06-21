---
title: "Sdeng"
excerpt: "A python web-crawler for European hoops."
header:
    teaser: "/assets/images/portfolio/transistor-upload.png"
last_modified_at: 2022-05-15
--- 

_Sdeng_ is a web crawler for downloading and querying international basketball data from the web.

I noticed that it's very hard to find accurate play-by-play logs for overseas basketball, so I tried to develop an easy-to-use and expandable system to download them.

Differently from other basketball libraries, such as [nba_api](https://github.com/swar/nba_api), right now Sdeng does not allow to download directly to a pandas dataframe, but it requires a persistent storage system, such as a relational database.

I am aware that this may complicate the usage of the app, but it is actually necessary since European leagues' websites do not offer a direct API endpoint, and since complex queries require time and memories, I think that is better to save data first, and query it subsequently, in order to guarantee more robustness and reusability.

As of today (23rd of March) the only supported storage system is MySQL, but from the next weeks SQLite and CSV will be supported too.

When downloading games from FIBA competitions, expect much longer download times than with other leagues. This is due to the fact that FIBA website widely uses Ajax, and therefore requires a more complex handling of http requests, and slower download times.

You can find the description of the process and the explanation about why certain decisions were made on [my website](https://francescoolivo.com/categories/#sdeng).

If you want to collaborate I am more than happy to receive some help, and if you have any doubts you can send me a [mail](mailto:francesco.olivo@sdeng.io)

## Supported leagues
At the time being, supported leagues are:
- Euroleague (EL)
- Eurocup (EC)
- Basketball Champions League (BCL)
- Europe Cup (FIBAEC)
- Italian National League (LBA)
- Italian Cup (CITA)
- Italian Supercup (SCITA)
- Italian Next Gen CUP (LBAU19)
- Spanish National League (ACB)
- Adriatic League (ABA)
- U19 Adriatic League (ABAU19)
- Tokyo 2020 Men Olympics (OLY2020)

During the next months I will add new competitons, starting from NBA, NCAA, Australian National League, and others.

## Usage

### Installation

First, clone this repo on your machine and move to the project directory:
```shell
git clone https://github.com/francescoolivo/sdeng.git
cd sdeng
```

Now create a conda environment and activate it:
```shell
conda env create -f environment.yml
conda activate sdeng
```

If you want to call the environment with a different name, you just have to change the environment name in the first line of the `environment.yml` file.

### Downloading

If you plan to download data from FIBA leagues, your script will need access to a web browser in order to make Ajax requests.

### Storage

#### MySQL

I will not describe how to download MySQL here, since many people already did it and in a much better way than I would ever be able to do. I suggest you [this guide](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04), but go for whatever best suits you and your OS.

From now on, I will assume that you have a database on which you have full access and grant.

The first thing to do is to create the tables in the database. Move to the root directory of the repo and prepare the database using the SQL file in the MySQL directory:
```shell
cd sdeng
mysql -u username -h host -p database_name < MySQL/schema.sql
```
And prompt you password.

Now you have to create a config file in the MySQL directory, that the python program will use to access the database. Use
```shell
nano MySQL/config.yaml
```

and paste
```yaml
host: 'localhost'
user: 'username'
password: 'password'
database: 'database_name'
```
with your values.

Et voilà, you are done and ready to fill the database.

### Run

As a fundamental premise, scraping data from the internet is the biggest source of unexpected exceptions, errors and wrong data that you could ever imagine.
Therefore, expect some wrong data, missing values and outliers. I am doing my best to catch all possible errors and solve them, so if you notice a weird behavior just tell me or open an Issue, and I'll do my best to fix it as soon as possible.

The download interface is pretty straightforward, if you are not already in the repo root directory go there by using

```shell
cd sdeng
```

Where you will find the run.py file. It takes two mandatory arguments, a list of leagues (-l or --leagues) and the writer to use (-w or --writer), and a bunch of possible filters. You can see all possible parameters by typing

```shell
python3 run.py -h
```

I strongly suggest you to use less filters as possible, and to apply them when you query the data, not when you download it. I would recommend to use only the `start_date`, `end_date` filters when you want to add new games, and the `--status played` filter since often games are rescheduled, and it can get messy.

These are some example of how to use the downloader:

- To download all available data from Euroleague on your MySQL database:
```shell
python3 run.py -l EL -w mysql
```
or
```shell
python3 run.py --leagues EL --writer mysql
```
- To download all data from the last two seasons of the Italian league:
```shell
python3 run.py -l LBA -w myql --seasons 2020-2021 2021-2022
```
- To download all the Eurocup and Euroleague games of Regular Season from the last week, using a config file located somewhere different from the MySQL directory:
```shell
python3 run.py -l EC EL -w mysql --config path/config.yaml --start_date 2022-3-16 --end_date 2022-3-23
```

In my experience, downloading a league season takes approximately 30/45 minutes. Performance improvements will come where possible.

Downloading a league season from FIBA takes approximately a couple of hours, since the scraper needs to use Selenium.

## Future developments

In the next week I plan to add new writers (SQLite and CSV) and new leagues, in particular NBA, NCAA, G-League and others.
I also want to develop a `Reader` for executing common queries, such as standings, individual and teams stats and other useful statistics.
Also, a full logger is on development.
