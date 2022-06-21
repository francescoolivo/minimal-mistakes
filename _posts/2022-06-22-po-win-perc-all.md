---
title: "Yet another playoffs winning percentage chart"
excerpt: "But this the scraper is written with R!"
header:
    teaser: "/assets/images/articles/mvps_win_perc_final.png"
categories:
- "Analyses"
---

Welcome back! Today I will make a new plot, answering the question that I raised in [my last article]({% post_url 2022-06-18-mvps-po-win-perc %}).

The question in subject is if there is a correlation between the number of played playoffs games and the playoffs winning percentage of a player.
The rationale is that a good player is likely to play many playoffs games, and that a good player should also help the team to win.

So, without further ado, let's jump into the tutorial to fetch the data and then plot it. This time I will scrape the data using R and the `rvest` package.
It's my first time scraping with R, so forgive if the code is not perfect.

# Getting the data

Just like last time, we will get the data from [basketball-reference](basketball-reference.com). Yet this time, instead of fetching the data for MVPs only, we will get it for every player who ever made an appearance in the NBA playoffs.

To do so, we will start from [the list of NBA players](https://www.basketball-reference.com/players/), and we shall iterate over all the letters. From any letter page, we will get some player information, such as the role and the status (active or retired), and using the link to his page we will eventually scrape the playoffs record.

Let's import the library and set-up the empty dataframe:

```R
library(rvest)
library(tidyverse)
library(ggplot2)

baseurl <- "https://www.basketball-reference.com"

# setting up the empty dataframe
df <- tibble(
  name = character(),
  status = character(),
  role = character(),
  games = numeric(),
  wins = numeric(),
  losses = numeric()
)
```

Now we can scrape the data. Since we have to scrape a lot of pages, it will take some time, approximately twenty minutes in my experience.
As I said before, it's the first time I ever use `rvest`, but when dealing with so many pages it should be integrated with `polite`, to avoid overloading the website we are scraping. I will update the article as soon as I will learn how to use it.

```R
# iterating over all the letters
for (letter in letters) {
  letter_url <- paste(baseurl, "players", letter, sep = "/")

  # getting all the players with the current letter
  players_letter <- read_html(letter_url) %>%
    html_node("table#players") %>%
    html_node("tbody") %>%
    html_nodes("tr")

  for (p in players_letter) {
    name <- p %>%
      html_node("th") %>%
      html_text()

    # if the player is active the name is embedded in a strong tag, we check if it exists
    strong_tag <- p %>%
      html_node("th") %>%
      html_node("strong")

    if (!is.na(strong_tag)) {
      status <- "Active"
    }
    else {
      status <- "Retired"
    }

    role <- p %>%
      html_node("td[data-stat=pos]") %>%
      html_text()

    player_url <- p %>%
      html_node("th") %>%
      html_node("a") %>%
      html_attr("href")

    print(c(name, player_url))

    player_html <- read_html(paste0(baseurl, player_url))

    # removing the tables from comments, see https://stackoverflow.com/questions/40665907/web-scraping-data-table-with-r-rvest
    corrected_html <- player_html %>%
      html_nodes(xpath = '//comment()') %>%    # select comments
      html_text() %>%    # extract comment text
      paste(collapse = '') %>%    # collapse to single string
      read_html()

    ## skipping players who did not play any playoffs game
    if (is.na(corrected_html %>%
                 html_node("table#playoffs-series"))) {
      next
    }

    footer_row <- corrected_html %>%
      html_node("table#playoffs-series") %>%
      html_node("tfoot") %>%
      html_node("tr")

    games <- footer_row %>%
      html_node("td[data-stat=g]") %>%
      html_text() %>%
      as.integer()

    wins <- footer_row %>%
      html_node("td[data-stat=wins]") %>%
      html_text() %>%
      as.integer()

    losses <- footer_row %>%
      html_node("td[data-stat=losses]") %>%
      html_text() %>%
      as.integer()

    df <- df %>%
      add_row(name = name, status = status, role = role, games = games, wins = wins, losses = losses)

  }
}


df <- df %>%
  mutate(perc = wins / games,
         status = factor(status, levels = c("Active", "Retired")))


write.csv(df, "data.csv")
```

And we're done! We simply need to plot the data and analyze it.

## Plotting the data

This time I will not make any intermediate version of the chart, if you are interested in the details you can read my [last article]({% post_url 2022-06-18-mvps-po-win-perc %}) where I discuss them.

```R
df %>%
  # removing non-relevant data
  filter(games >= 50) %>%
  ggplot(aes(x = games, y = perc)) +
  # adding a smoothed line to the chart to understand the trend
  geom_smooth(color = "gray30", se = FALSE) +
  # plotting the points with a small jitter
  geom_point(aes(fill = status), shape = 21, alpha = .75, size = 2.5, position = position_jitter(seed = 1729)) +
  # setting the scales
  scale_fill_manual(values = c("firebrick2", "dodgerblue2")) +
  scale_x_continuous(breaks = seq(50, 300, 50)) +
  scale_y_continuous(breaks = seq(0.2, 1, 0.1), labels = scales::percent, limits = c(.29, .76)) +
  #labeling interesting active players
  annotate(geom = 'label', x = 263, y = .66, hjust = 1, vjust = 0, label = "LeBron", fill = "ghostwhite", color = 'grey20', family = "Consolas", size = 3) +
  annotate(geom = 'label', x = 147, y = .71, hjust = 0, vjust = 0, label = "Draymond & Klay", fill = "ghostwhite", color = 'grey20', family = "Consolas", size = 3) +
  annotate(geom = 'label', x = 134, y = .705, hjust = 0.75, vjust = 0, label = "Steph", fill = "ghostwhite", color = 'grey20', family = "Consolas", size = 3) +
  annotate(geom = 'label', x = 66, y = .725, hjust = 0, vjust = 0, label = "Looney", fill = "ghostwhite", color = 'grey20', family = "Consolas", size = 3) +
  annotate(geom = 'label', x = 63, y = .305, hjust = 0.25, vjust = 1, label = "McCollum", fill = "ghostwhite", color = 'grey20', family = "Consolas", size = 3) +
  annotate(geom = 'label', x = 85, y = .33, hjust = 0, vjust = 1, label = "Melo", fill = "ghostwhite", color = 'grey20', family = "Consolas", size = 3) +
  # adding the labs
  labs(
      title = "Playoffs winning percentage",
      subtitle = "Among players with at least 50 playoffs games.",
      caption = "Author: Francesco Olivo\nData: basketball-reference.com",
      x = "Games",
      y = "PO win percentage"
    ) +
  # setting the theme
  theme_minimal(base_size=9, base_family="Consolas") +
  theme(
    plot.background = element_rect(fill = 'ghostwhite', color = "ghostwhite"),
    plot.title.position = 'plot',
    plot.title = element_text(face = 'bold', size = 15, hjust = 0),
    plot.subtitle = element_text(margin = margin(5, 0, 5, 0), lineheight = 1.2, hjust = 0),
    plot.caption = element_text(lineheight = 1.2, margin = margin(10, 0, 0, 0), hjust = 1),
    plot.margin = margin(10, 15, 10, 10),
    axis.title.y = element_text(margin = margin(t = 0, r = 5, b = 0, l = 0)),
    axis.title.x = element_text(margin = margin(t = 5, r = 0, b = 0, l = 0)),
    legend.position = "bottom",
    legend.title = element_blank(),
    legend.text = element_text(size = 8)
  )

# saving the plot
ggsave("winning_perc.png", w = 8, h = 6, dpi = 600)
```

<a href="/assets/images/articles/win_perc_all.png">
    <img 
        src="/assets/images/articles/win_perc_all.png" 
        alt="Winning percentage in the playoffs"
    >
</a>

## Analyzing the data

It looks like there is a certain correlation, particularly for *very* good players. Basically, all the players with at least 200 games in the playoffs, namely:
- LeBron James (266)
- Derek Fisher (259)
- Tim Duncan (251)
- Robert Horry (244)
- Kareem Abdul-Jabbar (237)
- Tony Parker (226)
- Kobe Bryant (220)
- Manu Ginóbili (218)
- Shaquille O'Neal (216)
- Scottie Pippen (208)
Have a winning percentage of at least 60%. Incidentally, the first player with less than 50% is John Stockton, who won 89 of his 182 playoffs games.

Thus said, as the smoothed line suggest, there is a certain correlation. If we look at the data including also players who played less than 50 playoff games, it is even more clear:

<a href="/assets/images/articles/win_perc_all_nolimits.png">
    <img 
        src="/assets/images/articles/win_perc_all_nolimits.png" 
        alt="Winning percentage in the playoffs, all players"
    >
</a>

If we compute the correlation score, using

```R
cor(df$games, df$perc)
```

We get 0.33, which indicates a correlation, but quite weak. So it's likely that the number of playoff games is a factor, but not the only one.

An important factor is the team that a players plays for, since a role guy may get an incredibly high winning percentage just by playing along a superstar. For instance, the active player with the highest percentage right now is Kevon Looney, that is a good player but not a superstar.

This chart shows how we can "cluster" some of the most iconic dynasties:

<a href="/assets/images/articles/win_perc_dynasties.png">
    <img 
        src="/assets/images/articles/win_perc_dynasties.png" 
        alt="Winning percentage in the playoffs by dynasty"
    >
</a>

Also, there are many great players that played a lot of PO games but did not win anything, take the dynamic duo Stockton and Malone.

So, all in all, we can answer the question:

The number of PO games is *one* of the factors which helps to predict a player PO winning percentage, but it is not the only one. For a more complete analysis one should also consider the role in the team, the teammates and the strength of the opponents.

This is for today, thank you for making it till the end!
