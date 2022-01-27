---
title: "Koukolon"
excerpt: "A Java-based Tablut artificial player."
header:
    teaser: "/assets/images/portfolio/transistor-waiting.png"
last_modified_at: 2021-11-26
order: 3
---

## 2021 Tablut Challenge

Tablut is a northern Europe board game, whose rules were first
transcribed in Latin by Linnaeus in 1732. The Latin transcription
led to uncertainties, and for this reason we are not sure about
the real rules of the game. This uncertainty also led to many
versions of the game: we are going to reference the Ashton version
described in this
[paper](http://ww.aagenielsen.dk/LinnaeusPaper-Longer.pdf).

## Koukolon
Koukolon is an artificial player for Ashton's Tablut, developed by Enrico Pallotta, Francesco Olivo, Yuri Noviello e Flavio Pinzarrone to
compete in 2021 Tablut Challenge, organized by professor Milano and
tutor Galassi for Fundamentals of Artificial Intelligence Course,
which is part of the AI Master Degree of University of Bologna.
Koukolon was inspired by 2020 Tablut Challenge winner,
[BrAInMates](https://github.com/gmurro/Tablut),
whose heuristics have been expanded in order to be more flexible and smart.

## Download
You can download the project through git:
```
git clone https://github.com/flaviopinzarrone/koukolon.git
cd koukolon
```

## Run
In order to run the project, you need to move in the 'Executables'
directory and run the server using:
```
java -jar Server.jar
```
Then you simply need to run the artificial players, using:
```
./koukolon black 60 localhost
```
or
```
./koukolon white 60 localhost
```
If you want to play koukolon's white vs koukolon's black you will have
to run both the previous commands on different terminals.

## Replay last game
You can rewatch the last game played using in the 'koukolon' directory:

```
ant replay_games
```
