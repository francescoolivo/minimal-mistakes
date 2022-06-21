---
title: "Plotting the MVPs playoffs winning percentage"
header:
    teaser: "/assets/images/articles/mvps_win_perc_final.png"
categories:
- "Analyses"
---

If you don't live under a rock, you probably heard that the Golden State Warriors won the 2022 NBA title, their fourth since 2015,
and maybe you also heard that Stephen Curry finally won the Finals MVP award.

Yesterday afternoon I was scrolling my Twitter feed when I stumbled upon this curious chart.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">UPDATED: <br>22-4 (.846) in career playoff series<br>93-41 (.694) in career playoff games <a href="https://t.co/VfgtfAZvY8">https://t.co/VfgtfAZvY8</a></p>&mdash; Tom Haberstroh (@tomhaberstroh) <a href="https://twitter.com/tomhaberstroh/status/1537792559774420992?ref_src=twsrc%5Etfw">June 17, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

Now, to quote one of my favorite Facebook tag pages, *this is terrible data presentation, and we're rejecting your paper*.  Most of the lines are overlaid, thus making the chart unreadable.

The information is not wrong *per se*, but the plot type does not allow to fully express the range of values within the MVP winners.
I then decided to remake the chart in a better way, and I am writing this tutorial hoping that you might find it useful.

## Getting the data

The author of the original chart got the data from [stathead](stathead.com), but since I don't have a subscription, we'll have to scrape the data from basketball-reference.

First thing, we set up the jupyter notebook, importing the libraries and creating a utilty function that we will need frequently.

```python
import requests
from bs4 import BeautifulSoup, Comment
import csv


def get_soup(url: str, headers=None, params=None):
    response = requests.get(url, headers=headers, params=params).text
    soup = BeautifulSoup(response, 'lxml')

    return soup
```
BeautifulSoup is the state of the art for scraping using python, and the ```get_soup``` function will avoid us to repeat code.

Let's start from the MVPs.

A list of the mvps is available [here](https://www.basketball-reference.com/awards/mvp.html). For simplicity's sake, I will use the summary table, since it allows us to avoid repetitions. This is not completely true, since Julius Erving, aka Dr J., won both an NBA MVP and 3 ABA MVPs, but we can simply skip the ABA MVPs.

This is what we get when we analyze the page using the browser inspector:

<a href="/assets/images/articles/mvp_summary.png">
    <img 
        src="/assets/images/articles/mvp_summary.png" 
        alt="MVPs website inspection"
    >
</a>

We can find the table we need using its id, namely ```mvp_summary```, and every row in the body can be collected by iterating over all ```tr``` tags.

We need three data from every row:
- The name of the player, which we can obtain as the text of the ```th``` tag.
- The relative url of the player's page, which is the ```href``` attribute of the ```a``` tag. We can find it embedded inside the previously mentioned ```th``` tag.
- The league where the award was won, either NBA or ABA. We can find it in the first ```td``` tag.

```python
baseurl = "https://www.basketball-reference.com"
mvps_url = baseurl + "/awards/mvp.html"

# the list which will store the result
players = []

mvps_soup = get_soup(mvps_url)

# getting the body of the goal table
mvps_table = mvps_soup.find('table', id = 'mvp_summary').find('tbody')

# iterating across all rows in the table
for mvp in mvps_table.find_all('tr'):

    name = mvp.find('th').text.strip()
    league = mvp.find("td").text.strip()

    # skipping ABA MVPs
    if league != "NBA":
        continue

    url = mvp.find('th').find('a').attrs['href']
    
    # get the data from the player page and insert it into the players list
```

The code is quite straightforward once you get the grasp of how bs4 works, if you don't understand something feel free to contact me, or simply check the bs4 docs. It may get a while to get accustomed to bs4 syntax, but it is not impossible.

Now let's get into the playoffs win percentage. Let's take an arbitrary MVP, say Steph, and have a look at his [basketball-reference page](https://www.basketball-reference.com/players/c/curryst01.html)

This is the table we are interested in.

<script src="https://widgets.sports-reference.com/wg.fcgi?css=1&site=bbr&url=%2Fplayers%2Fc%2Fcurryst01.html&div=div_playoffs-series"></script>

The data that we are interested in is the first row of the footer of the table, that collects the overall playoffs win record

By inspecting the page we find out that the table has the id ```playoffs-series```, and that the first ```tr``` of the ```tfoot``` tag contains all the information we are looking for.

<a href="/assets/images/articles/po_record.png">
    <img 
        src="/assets/images/articles/po_record.png" 
        alt="PO record inspection"
    >
</a>

In particular, we can categorize the columns using the `data-stat` attribute:
- `g` for the total number of games
- `wins` for the number of wins
- `losses` for the number of losses. This information is not necessary for computing the win percentage but let's keep it anyway.

Unfortunately, when we download the page using Beautiful Soup, the tables are embedded inside a comment, where bs4 is not able to search for tags. Thus, we shall "unpack" the comment:

```python
div = po_soup.find("div", id="all_playoffs-series")
for comment in div(text=lambda text: isinstance(text, Comment)):
    if 'table"' in comment.string:
        tag = BeautifulSoup(comment, 'html.parser')
        comment.replace_with(tag)
        break
```

Once we collected all the information, let's append it as a dictionary to the `players` list. Eventually our code will look like this:

```python
mvp_url = "https://www.basketball-reference.com/awards/mvp.html"
baseurl = "https://www.basketball-reference.com"

players = []

mvps_soup = get_soup(mvp_url)

mvps_table = mvps_soup.find('div', id = 'all_mvp_summary').find('tbody')

for mvp in mvps_table.find_all('tr'):
    name = mvp.find('th').text.strip()
    league = mvp.find("td").text.strip()

    if league != "NBA":
        continue

    url = mvp.find('th').find('a').attrs['href']

    po_soup = get_soup(baseurl + url)

    div = po_soup.find("div", id="all_playoffs-series")
    for comment in div(text=lambda text: isinstance(text, Comment)):
        if 'table"' in comment.string:
            tag = BeautifulSoup(comment, 'html.parser')
            comment.replace_with(tag)
            break

    summary_row = div.find("tfoot").find("tr")

    games = summary_row.find('td', {"data-stat": "g"}).text
    wins = summary_row.find('td', {"data-stat": "wins"}).text
    losses = summary_row.find('td', {"data-stat": "losses"}).text

    players.append(
        {"player": name,
         "games": games,
         "wins": wins,
         "losses": losses}
    )
```

Now we simply need to save the data as a csv file, which we can do in such way:

```python
keys = players[0].keys()

with open('mvps.csv', 'w', newline='') as output_file:
    dict_writer = csv.DictWriter(output_file, keys)
    dict_writer.writeheader()
    dict_writer.writerows(players)
```

## Creating the plot

We will use `R` and `ggplot2` to plot the data. Let's start from the required libraries and from reading the csv file we created:

```R
library(tidyverse)
library(ggplot2)
library(ggrepel)

df <- read_csv("mvps.csv")
```

Plotting this is the pretty easy, we can do it in just four lines of code:

```R
df %>%
  mutate(perc = wins / games) %>%
  ggplot(aes(x = games, y = perc, label = player)) +
  geom_label()
```

<a href="/assets/images/articles/mvps_win_perc_1.png">
    <img 
        src="/assets/images/articles/mvps_win_perc_1.png" 
        alt="Plot version 1"
    >
</a>

This plot does provide much more information than the original one:
1. We can see in a clearer way the differences among players with a similar win percentage.
2. We notice that Bill Walton was deliberately cut out from the chart to make it look like Curry is the first one among the MVPs. In fact, the 1978 MVP Walton only played 49 playoffs game, which quite conveniently allowed the author to set up a 50 games threshold. The provided data is not wrong, since Curry actually has the highest PO winning percentage among MVPs with at least 50 PO games, but I find it quite misleading.
3. We can see a kind of correlation among played PO games and win percentage. This makes me wonder if there is a correlation between playoff games and playoff win percentage, which I will explore in another article. Sounds possible though, from the moment that only the best players are likely to play many PO games, and the best players usually make a team win.  


Having said that, let's add some details to our plot, including if a player is active or retired. Since the dataset is small we can do it manually, otherwise we could have done it while scraping the data.

In particular, let's add if a player is active, let's switch from `geom_label` to `geom_label_repel` to avoid overlaid labels, then let's manually set the colors and adjust the scales.

```R
df %>%
  mutate(perc = wins / games,
         active = case_when(
           player %in% c("LeBron James",
                         "Stephen Curry",
                         "Giannis Antetokounmpo",
                         "Nikola Jokić",
                         "Kevin Durant",
                         "James Harden",
                         "Russell Westbrook",
                         "Derrick Rose") ~ "Active",
           TRUE ~ "Retired")) %>%
  ggplot(aes(x = games, y = perc, label = player, fill = active)) +
  geom_label_repel(color = "white", family = "Consolas", force_pull = 5, seed = 1729) +
  scale_fill_manual(values = c("firebrick4", "dodgerblue4"))  +
  scale_x_continuous(breaks = seq(50, 300, 50)) +
  scale_y_continuous(breaks = seq(0.2, 1, 0.1), labels = scales::percent)
```
<a href="/assets/images/articles/mvps_win_perc_2.png">
    <img 
        src="/assets/images/articles/mvps_win_perc_2.png" 
        alt="Plot version 2"
    >
</a>

Finally, let's adjust the label and the theme of the plot:

```R
df %>%
  mutate(perc = wins / games,
         active = case_when(
           player %in% c("LeBron James",
                          "Stephen Curry",
                          "Giannis Antetokounmpo",
                          "Nikola Jokić",
                          "Kevin Durant",
                          "James Harden",
                          "Russell Westbrook",
                          "Derrick Rose") ~ "Active",
           TRUE ~ "Retired")) %>%
  ggplot(aes(x = games, y = perc, label = player, fill = active)) +
  geom_label_repel(color = "white", family = "Consolas", force_pull = 5, seed = 1729) +
  scale_x_continuous(breaks = seq(50, 300, 50)) +
  scale_y_continuous(breaks = seq(0.2, 1, 0.1), labels = scales::percent) +
  scale_fill_manual(values = c("firebrick4", "dodgerblue4")) +
  guides(
    fill = guide_legend(
      title = "",
      override.aes = aes(label = "")
    )
  ) +
  labs(
    title = "MVPS playoffs winning percentage",
    subtitle = "In NBA history.",
    caption = "Author: Francesco Olivo\nData: basketball-reference.com",
    x = "Games",
    y = "PO win percentage"
  ) +
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
ggsave("mvps_win_perc_final.png", w = 12, h = 9, dpi = 300)
```

Thus creating the final plot.

<a href="/assets/images/articles/mvps_win_perc_final.png">
    <img 
        src="/assets/images/articles/mvps_win_perc_final.png" 
        alt="Plot final version"
    >
</a>

Hope you found the tutorial useful, stay tuned for other articles!
