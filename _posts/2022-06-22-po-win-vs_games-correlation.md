---
title: "Yet another playoffs wins chart"
excerpt: "But this time the scraper is written with R!"
header:
    teaser: "/assets/images/articles/win_vs_games_dynasties.png"
categories:
- "Analyses"
gallery:
- url: /assets/images/articles/win_vs_games_dynasties.png
  image_path: /assets/images/articles/win_vs_games_dynasties.png
  alt: "Some of the most iconic dynasties form clusters"
- url: /assets/images/articles/win_perc_all.png
  image_path: /assets/images/articles/win_perc_all.png
  alt: "Winning percentage against PO games"
---

Welcome back! Today I will make a new plot, answering the question that I raised in [my last article]({% post_url 2022-06-18-mvps-po-win-perc %}).

The question in subject is if there is a correlation between the number of played playoffs games and the playoffs wins of a player. Actually, the question was if the correlation is about games and winning percentage, but since the winning percentage is a ratio and is computed using the wins, I will directly use the wins.
The rationale is that a good player is likely to play many playoffs games, and that a good player should also help the team to win.

So, without further ado, let's jump into the tutorial to fetch the data and then plot it. This time I will scrape the data using R and the `rvest` package.
It's my first time scraping with R, so forgive if the code is not perfect.

# Getting the data

Just like last time, we will get the data from [basketball-reference](https://www.basketball-reference.com). Yet this time, instead of fetching the data for MVPs only, we will get it for every player who ever made an appearance in the NBA playoffs.

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

I will plot two different charts, one about the winning percentage, and one about the total wins.

This time I will not make any intermediate version of the chart, if you are interested in the details you can read my [last article]({% post_url 2022-06-18-mvps-po-win-perc %}) where I discuss them.

```R
df %>%
  ggplot(aes(x = games, y = wins)) +
  geom_point(shape = 21, alpha = .75, size = 2.5, position = position_jitter(seed = 1729), fill = "gray") +
  # adding a smoothed line to the chart to understand the trend
  geom_smooth(color = "firebrick2", se = TRUE, fill = "firebrick3") +
  # adding the labs
  labs(
    title = "Playoffs wins against games",
    caption = "Author: Francesco Olivo\nData: basketball-reference.com",
    x = "Games",
    y = "Wins"
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
ggsave("win_vs_games.png", w = 8, h = 6, dpi = 600)
```

<a href="/assets/images/articles/win_vs_games.png">
    <img 
        src="/assets/images/articles/win_vs_games.png" 
        alt="PO wins against PO games"
    >
</a>

## Analyzing the data

The linear relationship is quite evident. Nonetheless, let's use a scientific approach and let's formulate a hypothesis.

### Choosing a test statistic

I will use Pearson correlation as test statistic.

In R, we can compute it in this simple way:
```R
cor(df$games, df$wins, method = "pearson")
```

Which returns a quite strong 0.98 result. Remember that Pearson correlation coefficient ranges between 1 and -1, so it's a pretty good result.

### Defining the null hypothesis

Having a good result is not enough, in fact we should test the null hypothesis, to guarantee that we did not obtain this result by chance.

In our case, the null hypothesis is that playoffs games and playoffs wins are not correlated.

### Computing the p-value

We choose a standard threshold for the p-value, that is 0.05. If our p-value is less than this threshold, we can reject the null hypothesis.

```R
cor.test(df$games, df$wins, method = "pearson")
```

### Interpreting the result

The result is `p-value < 2.2e-16`, which is way lower than 0.05, thus allowing us to reject the null hypothesis.

We can now answer the question positively: there is a correlation between played games and wins. 

In particular, by looking at the previous plot, we can observe two patterns: for players with less than (approximately) 75 games the ration is 0.5, that means a win every two games.

The trend seems to be higher for players with more than 75 games, in particular the ratio seems to increase up to 0.75, which is pretty high. This data was computed approximately by looking at the chart, maybe I will compute it precisely in the next weeks so stay tuned.

## Another couple of charts

While I was writing the article I made another couple of charts which may be interesting, so I will leave them here. If you are interested in the tutorial you can [contact me](mailto:francesco.olivo@ßdeng.io), and I will send you the code. 

This way of sharing code sucks, I know, and I am preparing a git where you can find all of my charts, but it will take some internal reorganizing. It will probably be ready in the autumn since now I'm packed with my exams.

{% include gallery caption="Some nice charts" %}
