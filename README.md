DS 202 – Lab \#4 Progress Report
================
Ash Bhuiyan
2025-04-26

- [Lab report \#4 - instructions](#lab-report-4---instructions)
- [Lab 4: Scraping (into) the Hall of
  Fame](#lab-4-scraping-into-the-hall-of-fame)
  - [Ash Bhuiyan’s Contribution: Analysis and
    Summary](#ash-bhuiyans-contribution-analysis-and-summary)

# Lab report \#4 - instructions

Follow the instructions posted at
<https://ds202-at-isu.github.io/labs.html> for the lab assignment. The
work is meant to be finished during the lab time, but you have time
until Monday (after Thanksgiving) to polish things.

All submissions to the github repo will be automatically uploaded for
grading once the due date is passed. Submit a link to your repository on
Canvas (only one submission per team) to signal to the instructors that
you are done with your submission.

# Lab 4: Scraping (into) the Hall of Fame

------------------------------------------------------------------------

### 1. Visualize existing HallOfFame data

``` r
hof <- Lahman::HallOfFame
hof %>% 
  ggplot(aes(x = yearID, y = votes/needed*100, group=playerID)) +
  geom_hline(yintercept = 100, colour="grey70") + 
  geom_line() +
  geom_point(aes(colour = inducted), data = hof %>% filter(inducted=="Y")) +
  xlim(2000, 2022) +
  ylab("Percent of votes")
```

    ## Warning: Removed 5465 rows containing missing values or values outside the scale range
    ## (`geom_line()`).

    ## Warning: Removed 284 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

![](README_files/figure-gfm/visualize-lahman-1.png)<!-- -->

### 2. Scrape and clean 2025 Hall of Fame data

``` r
url <- "https://www.baseball-reference.com/awards/hof_2025.shtml"
raw_tbl <- read_html(url) %>%
  html_element("table") %>%
  html_table(fill = TRUE)

# Promote the first row to column names
names(raw_tbl) <- raw_tbl[1, ]
```

    ## Warning: The `value` argument of `names<-()` must be a character vector as of tibble
    ## 3.0.0.
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
    ## generated.

``` r
hof_2025_raw <- raw_tbl[-1, ]

# FIX: Make column names unique
names(hof_2025_raw) <- make.names(names(hof_2025_raw), unique = TRUE)

# Now you can safely filter and mutate
hof_2025_clean <- hof_2025_raw %>%
  filter(!is.na(Name), Name != "", !is.na(Votes), Votes != "") %>%
  distinct(Name, .keep_all = TRUE)

hof_2025 <- hof_2025_clean %>%
  mutate(
    playerID = str_replace_all(Name, " ", ""),
    yearID = 2025L,
    votedBy = "BBWAA",
    ballots = NA_integer_,
    needed = NA_integer_,
    votes = as.integer(Votes),
    # Note: The column for percentage may be called "X.vote" if it was "%vote" originally
    pct_vote = as.numeric(str_replace(X.vote, "%", "")),
    inducted = if_else(pct_vote >= 75, "Y", "N"),
    category = "Player",
    needed_note = NA_character_
  ) %>%
  select(playerID, yearID, votedBy, ballots, needed, votes, pct_vote, inducted, category, needed_note)
updated_hof <- bind_rows(HallOfFame, hof_2025)
```

### 3. Visualize updated Hall of Fame dataset

``` r
updated_hof %>% 
  ggplot(aes(x = yearID, fill = inducted)) +
  geom_bar() +
  xlim(1936, 2025) +
  labs(title = "Hall of Fame Elections, 1936-2025", x = "Year", y = "Number of Players")
```

    ## Warning: Removed 4 rows containing missing values or values outside the scale range
    ## (`geom_bar()`).

![](README_files/figure-gfm/plot-updated-1.png)<!-- -->

------------------------------------------------------------------------

## Ash Bhuiyan’s Contribution: Analysis and Summary

### 1. Histogram of 2025 Vote Percentages

``` r
hof_2025 %>%
  ggplot(aes(x = pct_vote)) +
  geom_histogram(binwidth = 5, boundary = 0, closed = "left") +
  scale_x_continuous(breaks = seq(0,100,10)) +
  labs(
    title = "Histogram of 2025 Hall-of-Fame Vote Percentages",
    x = "Vote Percentage",
    y = "Number of Candidates"
  ) +
  geom_vline(xintercept = 75, linetype="dashed", color = "red") +
  annotate("text", x = 80, y = 3, label = "75% inducted", vjust = -0.5)
```

![](README_files/figure-gfm/hist-2025-1.png)<!-- -->

### 2. Summary Statistics for 2025 Ballot

``` r
hof_2025 %>%
  summarize(
    mean_pct   = mean(pct_vote, na.rm = TRUE),
    median_pct = median(pct_vote, na.rm = TRUE),
    sd_pct     = sd(pct_vote, na.rm = TRUE),
    min_pct    = min(pct_vote, na.rm = TRUE),
    max_pct    = max(pct_vote, na.rm = TRUE)
  )
```

    ## # A tibble: 1 × 5
    ##   mean_pct median_pct sd_pct min_pct max_pct
    ##      <dbl>      <dbl>  <dbl>   <dbl>   <dbl>
    ## 1     24.2       11.6   29.9       0    99.7

| Statistic    | Value |
|:-------------|:-----:|
| **Mean**     | 24.2% |
| **Median**   | 11.7% |
| **Std. Dev** | 29.9% |
| **Min**      |  0%   |
| **Max**      | 99.7% |

### 3. Reflection

In 2025, the Hall of Fame ballot showed a wide range of support for
candidates. The histogram above shows most players received under 50%,
with a few standouts above the 75% threshold for induction.

------------------------------------------------------------------------

*I have completed (analysis/summary/visualization). - Ash Bhuiyan*
