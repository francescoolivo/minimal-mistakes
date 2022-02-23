---
title: "Sdeng: the LBA scraper"
categories:
- "Sdeng"
---

Hello everyone! Welcome back to the development of _Sdeng_, the library for international basketball data download.

Today we start developing the scrapers that will do the hard work and download all our interesting data. This is my first web crawler ever, so forgive me if something is messy or unclear, trust me, you have no idea how messy this job looks to me.

Nonetheless, we shall not give up during difficult tasks, so here we are, trying to do something good with what we have.

Let's start by analyzing the website we will scrape and by trying to understand how data are sent and organized.

## The website

You actually believed it would be so simple that we only had to scrape a single website?

LOL

Nope.

The Italian league recently moved to a new website, which you can find [here](https://www.legabasket.it/). Now, you might think, where is the problem, they surely moved all contents from the [old website](http://web.legabasket.it/) to the new one.

![gif](https://media.giphy.com/media/SVgKToBLI6S6DUye1Y/giphy.gif)

They _partly_ moved stuff. Let's see what I mean.

Suppose we want to find play-by-play data, an odd and rare event, how you may have imagined from the previous posts. If you check on the new website you will only find pbp logs from the 2017-2018 season, and, just to top it off, they are [incorrect](https://www.legabasket.it/game/22408/sidigas-avellino-dolomiti-energia-trentino/playbyplay).

Nothing that can't be fixed of course, since we can create the scores from scratch, but I fairly believe that this match did not end at 0-0. 

Nonetheless, you give up to the fact that for the Italian league you will only be able to collect pbp logs from the last 5 seasons. It is completely legit, since Italian basketball really loves to stay behind the rest of the world. BUT, if you check on the old website, you will find pbp logs from the [2012-2013 season](http://web.legabasket.it/game/65628/juvecaserta-ea7_emporio_armani_milano_69:78/pbp).

We can easily ignore the visualization problems, since the site is no longer maintained. The structure behind holds, and will allow us to get the precious data. 

Now, the only reason I could find to not moving such data to the new website is that in 2017 they changed the pbp data format, and they didn't want to refactor all previous play by play logs. I can understand it, but this will obviously harden our job here. Hurray. 

Another difference between the old website and the new one is the absence of club staffs and rosters from previous seasons. For 2021-2022 season you can find the complete staff only on the new website, while all previous staffs are only on the old one.

Let's consider the same season (2020-2021) for Olimpia Milano. You can find past seasons by moving to the "Team" or "Squadre" section of the website, and, on the page of the team, by clicking on "Storia", that stands for history. If you then click on the team name corresponding to the season, you can find all information about the team for the specified season.

In the Milan case, the two pages should contain the same exact data. This is the [old one](http://web.legabasket.it/team/1351/a_x_armani_exchange_milano/2020) while this is the [new one](https://www.legabasket.it/lba/squadre/2020/1433/a-x-armani-exchange-milano/).

Now, this is not a huge problem, but there is something annoying: the internal team id changes among the two versions. In the old version, as you can see in the previous link, the id is 1351, while in the new one it is 1433. This is annoying since it doesn't allow us to make two different queries using the same parameters, but that we have to fetch the same data for the same team twice. Ugh.

At least the new website apparently contains all other game information correctly, such as box scores and details. This means that the only problematic parts are the play by play logs from 2012 to 2017 and rosters and staffs up to 2021. It's not optimal, but we can deal with it.
