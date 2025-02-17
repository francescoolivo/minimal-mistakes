---
title: "Sdeng: the introduction"
categories:
    - "Sdeng"
---

Hello everyone, and welcome to <i>Sdeng: a library for basketball data and analysis</i>. During this series I will build a python library to download and analyze basketball statistics, not only from the NBA but also from other leagues such as Euroleague, NCAA, Italian Leagues and others.

To date, Machine Learning and AI are probably the most inflated words on the planet. Just kidding, they are crypto and Bitcoin. Nonetheless, anywhere you turn, Machine Learning is there, from medical diagnosis to finance to any possible field you can imagine.

Sports are no different. If you watched Moneyball, you know that this has been going on for over two decades now. Across the years, the use of automated systems for tracking and analyzing sports events and statistics is grown exponentially: today is the de facto standard in all the main championships.

<a href="https://www.v7labs.com/blog/ai-in-sports">This article</a> points out some interesting sports-related fields where ML is applied and constantly developed:

Truth to be told, until this a few months ago, my knowledge about ML in sports was depressingly shallow: of course, studying Computer Science, I knew that it was used, but I never went down the rabbit hole to find out to which level.

It all changed one summer night shortly after my Bachelor's Degree graduation. I was talking with my basketball buddy when he showed me this video.

{% include video id="MpLHMKTolVw" provider="youtube" %}

Knowing that Data Science in NBA teams was performed by a dozen math PhDs gave me a whole new perspective on the subject. I had always believed that ML was more a fashionable word that teams used as a decoration than an actual relevant decision-making tool.

I decided to go down the rabbit hole.

During the last few months, I read many books and many articles on basketball analysis, and I started to think about doing my own research. I nonetheless realized that I still did not have enough experience in Machine Learning and Data Science, but the Master's Degree in Artificial Intelligence that I am attending has started to fill the gap.

Therefore, I now feel confident enough to start this journey.

Since I like to go big, doing some analysis on some online data is not enough for me, so I decided to develop a python library to help me during the process.

This series is none other than a journal of all the steps and the decisions I will take during the journey, hoping that it can be helpful to other people, both as documentation and as inspiration in sports analysis.

There are a lot of great tools online, such as the <a href="https://github.com/swar/nba_api">NBA API</a> on Github or <a href="http://www.pbpstats.com/">pbpstats</a>, and excellent basketball data newsletters.

My goal is not to develop software as good as the ones I mentioned, it's simply to have fun, learn something new and answer some basketball-related questions. Nonetheless, I will do my best to respect all software engineering principles, keep good documentation and maintain the code frequently.

A nice feature of this project is that it can easily extend into all principal CS and AI fields: we will start with a SQL database development. Subsequently, we will design a web crawler to obtain data from leagues websites. Once we download and store the data nice and tidy, we can proceed with the juicy part, the analyses. From this point on, there are many possible extensions I would like to explore, from a website to a CV system for obtaining advanced data to social network analysis, and so on.

Although the primary goal of this project is to have fun and learn, this does not mean that we cannot get something practical out of it. The basketball world is extremely NBA-oriented. This is more than legit if you consider that NBA volumes are larger by order of magnitudes than any other league ones.

Nonetheless, European (and Australian) basketball have a massive fan base, and they deserve good open-source software as well! Jokes aside, overseas basketball recently produced some of the most dominating players in the league, such as Giannis, Jokic, and Doncic.

My hope is to make overseas basketball a bit more popular and accessible.
From now on, I will refer to the project as SDEng, which ideally stands for Sports Data Engineering, but is actually the Italian known onomatopoeia for the sound of a missed shot bouncing off the rim.

Before saying goodbye, some thankings are due. Firstly, to professor Patella from the University of Bologna, who taught me the software engineering principles that I will follow. He also kindly helped me to solve some doubts I had in the domain analysis, and I am grateful for the insightful suggestions.

Secondly, I would like to thank professor Zuccolotto from the University of Brescia for all the tips she provided me, both in terms of suggested readings and field-related advice. Her help has proven fundamental in developing the bases without whom this series could simply not exist.

This is the end of the introduction: in the next episode, I will analyze the basketball domain, to design a consistent and extendable SQL database.
Episode 1 is coming soon, so stay tuned. Bye!
