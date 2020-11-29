04-visualise-scot
================
2020-11-29

## Length of speech

``` r
lm_words <- lm(n_words ~ date, data = covid_speeches_scot)
lm_words_rsq <- glance(lm_words)$r.squared

covid_speeches_scot %>%
  ggplot(aes(x = date, y = n_words)) +
  geom_point(color = scotblue, alpha = 0.7) +
  geom_smooth(aes(x = date, y = n_words), method = lm, formula = y ~ x, color = "darkgray") +
  labs(
    title = "Length of First Minister's COVID-19 speeches",
    subtitle = glue("Measured in number of words, R-squared = {percent(lm_words_rsq)}"),
    x = NULL, y = "Number of words", color = NULL, shape = NULL
  )
```

<img src="04-visualise-scot_files/figure-gfm/unnamed-chunk-1-1.png" width="90%" />

## Word frequency

``` r
threshold <- 400

covid_speeches_scot_words %>%
  count(word, sort = TRUE) %>%
  filter(n > threshold) %>%
  ggplot(aes(y = fct_reorder(word, n), x = n, fill = n)) +
  geom_col() +
  guides(fill = FALSE) +
  labs(
    title = "Frequency of words in First Minister's COVID-19 briefings",
    subtitle = glue("Words occurring more than {threshold} times"),
    y = NULL, x = NULL
  )
```

<img src="04-visualise-scot_files/figure-gfm/unnamed-chunk-2-1.png" width="90%" />

## Sentiment analysis

Sentiments: Positive and negative.

``` r
covid_speeches_scot_words %>%
  inner_join(get_sentiments("bing")) %>%
  count(sentiment, word, sort = TRUE) %>%
  group_by(sentiment) %>%
  slice_head(n = 20) %>%
  ggplot(aes(y = fct_reorder(word, n), x = n, fill = sentiment)) +
  geom_col() +
  guides(fill = FALSE) +
  facet_wrap(~ sentiment, scales = "free") +
  labs(
    title = "Sentiment and frequency of words in First Minister's COVID-19 briefings",
    subtitle = "Sentiment evaluated using the Bing lexicon",
    y = NULL, x = NULL
  )
```

    ## Joining, by = "word"

<img src="04-visualise-scot_files/figure-gfm/unnamed-chunk-3-1.png" width="90%" />

“positive” isn’t really a positive word in this context. Remove and plot
again.

``` r
covid_speeches_scot_words <- covid_speeches_scot_words %>%
  filter(word != "positive")
```

``` r
covid_speeches_scot_words %>%
  inner_join(get_sentiments("bing")) %>%
  count(sentiment, word, sort = TRUE) %>%
  group_by(sentiment) %>%
  slice_head(n = 20) %>%
  ggplot(aes(y = fct_reorder(word, n), x = n, fill = sentiment)) +
  geom_col() +
  guides(fill = FALSE) +
  facet_wrap(~ sentiment, scales = "free") +
  labs(
    title = "Sentiment and frequency of words in First Minister's COVID-19 briefings",
    subtitle = "Sentiment evaluated using the Bing lexicon",
    y = NULL, x = NULL
  )
```

    ## Joining, by = "word"

<img src="04-visualise-scot_files/figure-gfm/unnamed-chunk-5-1.png" width="90%" />

## Daily sentiments

### Lexicon: Bing

Sentiments: Positive and negative.

``` r
covid_speeches_scot_words %>%
  inner_join(get_sentiments("bing")) %>%
  count(date, sentiment) %>%
  pivot_wider(names_from = sentiment, values_from = n) %>%
  mutate(sentiment = positive - negative) %>%
  ggplot(aes(x = date, y = sentiment)) +
  geom_line(color = "gray") +
  geom_point(aes(color = sentiment > 0), size = 2) +
  #scale_color_manual(values = c("gray", "red")) +
  geom_hline(yintercept = 0, linetype = "dashed", color = "lightgray") +
  guides(color = FALSE) +
  labs(
    title = "Daily sentiment score of words in First Minister's COVID-19 briefings",
    subtitle = "Sentiment score calculated as the difference between the number of words with 
positive and negative sentiments according to the Bing lexicon",
    x = "Date", y = "Sentiment score"
  ) +
  theme(legend.position = "bottom")
```

    ## Joining, by = "word"

<img src="04-visualise-scot_files/figure-gfm/unnamed-chunk-6-1.png" width="90%" />

and now with a smooth curve…

``` r
covid_speeches_scot_words %>%
  inner_join(get_sentiments("bing")) %>%
  count(date, sentiment) %>%
  pivot_wider(names_from = sentiment, values_from = n) %>%
  mutate(sentiment = positive - negative) %>%
  ggplot(aes(x = date, y = sentiment)) +
  geom_smooth(color = "gray") +
  geom_point(aes(color = sentiment > 0), size = 1) +
  #scale_color_manual(values = c("gray", "red")) +
  geom_hline(yintercept = 0, linetype = "dashed", color = "lightgray") +
  guides(color = FALSE) +
  labs(
    title = "Daily sentiment score of words in First Minister's COVID-19 briefings",
    subtitle = "Sentiment score calculated as the difference between the number of words with 
positive and negative sentiments according to the Bing lexicon",
    x = "Date", y = "Sentiment score"
  ) +
  theme(legend.position = "bottom")
```

    ## Joining, by = "word"

    ## `geom_smooth()` using method = 'loess' and formula 'y ~ x'

<img src="04-visualise-scot_files/figure-gfm/unnamed-chunk-7-1.png" width="90%" />

### Lexicon: NRC

Sentiments: rust, fear, negative, sadness, anger, surprise, positive,
disgust, joy, and anticipation.

``` r
covid_speeches_scot_words %>%
  inner_join(get_sentiments("nrc")) %>%
  mutate(
    sentiment = fct_relevel(sentiment, "positive", "anticipation", "joy", "surprise", "trust",
                            "negative", "anger", "disgust", "fear", "sadness"),
    sentiment_binary = if_else(sentiment %in% c("positive", "anticipation", "joy", "surprise", "trust"), "positive", "negative")
  ) %>%
  count(sentiment_binary, sentiment, word, sort = TRUE) %>%
  group_by(sentiment) %>%
  slice_head(n = 10) %>%
  ggplot(aes(y = fct_reorder(word, n), x = n, fill = sentiment_binary)) +
  geom_col() +
  guides(fill = FALSE) +
  facet_wrap(~ sentiment, scales = "free_y", ncol = 5) +
  labs(
    title = "Sentiment and frequency of words in First Minister's COVID-19 briefings",
    subtitle = "Sentiment evaluated using the NRC lexicon",
    y = NULL, x = NULL
  )
```

    ## Joining, by = "word"

<img src="04-visualise-scot_files/figure-gfm/unnamed-chunk-8-1.png" width="90%" />

``` r
covid_speeches_scot_words %>%
  inner_join(get_sentiments("nrc")) %>%
  mutate(
    sentiment = fct_relevel(sentiment, "positive", "anticipation", "joy", "surprise", "trust",
                            "negative", "anger", "sadness", "disgust", "fear"),
    sentiment_binary = if_else(sentiment %in% c("positive", "anticipation", "joy", "surprise", "trust"), "positive", "negative")
    ) %>%
  count(date, sentiment_binary, sentiment) %>%
  ggplot(aes(x = date, y = n, color = sentiment_binary)) +
  geom_line(size = 0.3) +
  guides(color = FALSE) +
  facet_wrap(~ sentiment, ncol = 5) +
  labs(
    title = "Sentiment score of words in First Minister's COVID-19 briefings over time",
    subtitle = "Sentiment score calculated as number of words associated with a given sentiment according to the NRC lexicon",
    x = "Date", y = "Sentiment score", shape = NULL, color = NULL
  )
```

    ## Joining, by = "word"

<img src="04-visualise-scot_files/figure-gfm/unnamed-chunk-9-1.png" width="90%" />

## Bigram frequency

``` r
covid_speeches_scot_bigrams %>%
  mutate(
    bigram = if_else(bigram == "care home", "care home(s)", bigram),
    bigram = if_else(bigram == "care homes", "care home(s)", bigram)
  ) %>%
  count(bigram, sort = TRUE) %>%
  filter(n > 60) %>%
  ggplot(aes(y = fct_reorder(bigram, n), x = n, fill = n)) +
  geom_col() +
  guides(fill = FALSE) +
  labs(
    title = "Frequency of bigrams in First Minister's COVID-19 briefings",
    subtitle = "Bigrams occurring more than 60 times",
    y = NULL, x = NULL
  )
```

<img src="04-visualise-scot_files/figure-gfm/unnamed-chunk-10-1.png" width="90%" />

## Social vs. physical distancing

``` r
covid_speeches_scot %>%
  # make sure COVID-19 (and all its various spellings) don't get split
  # tidytext doesn't remove underscores
  # https://stackoverflow.com/questions/58281091/preserve-hyphenated-words-in-ngrams-analysis-with-tidytext
  mutate(
    text = str_replace_all(text, "COVID-19", "COVID_19"),
    text = str_replace_all(text, "COVID 19", "COVID_19"),
    text = str_replace_all(text, "Covid-19", "COVID_19"),
    text = str_replace_all(text, "Covid 19", "COVID_19")
  ) %>%
  unnest_tokens(bigram, text, token = "ngrams", n = 2) %>%
  filter(str_detect(bigram, "social dist|physical dist")) %>%
  mutate(soc_phys = if_else(str_detect(bigram, "social"), "S", "P")) %>%
  count(date, soc_phys) %>%
  ggplot(aes(x = date, y = n, color = soc_phys)) +
  geom_text(aes(label = soc_phys)) +
  guides(color = FALSE) +
  labs(x = "Date", y = "Frequency",
       title = "Social (S) vs. physical (P) distancing",
       subtitle = "Number of mentions over time") +
  scale_color_manual(values = c(scotblue, "darkgray")) +
  scale_y_continuous(limits = c(0, 10), breaks = seq(0, 10, 2))
```

<img src="04-visualise-scot_files/figure-gfm/unnamed-chunk-11-1.png" width="90%" />
