---
title: Last Words from Death Row
author: Kathryn Daugherty
date: '2017-11-25'
slug: last-words-from-death-row
categories: []
tags: []
css: []
description: ''
highlight: yes
scripts: []
---

From Kaggle, we get a <a href="https://www.kaggle.com/ianmobbs/texas-death-row-executions-info-and-last-words">data set</a> including last words from Texas death row inmates.  It has names, ages, race, county, date and statements for each person spanning from 1982 to 2017.  We're going to use R to identify common words/themes.

```{r}
# Set up tidytext
require(tidytext)

# Set working directory
setwd("~/Desktop/R/TX_Executions/text/")

# Load dataset
# offenders <- read.table("Last_Statement.txt")
offenders <- readLines(file.choose())
head(offenders)
```

Let's load required R packages and make a text corpus.

```{r}

# Load other packages
require(dplyr)
require(ggplot2)
require(wordcloud)
require(stringr)
require(tm)
require(SnowballC)
require(RColorBrewer)

# Convert text to corpus
lastWords <- Corpus(VectorSource(offenders))
inspect(lastWords)
```

Let's clean up the verbiage and formatting so they are easier to analyze.

```{r}
# Clean up text
toSpace <- content_transformer(function (x , pattern ) gsub(pattern, " ", x))
lastWords <- tm_map(lastWords, toSpace, "/")
lastWords <- tm_map(lastWords, toSpace, "@")
lastWords <- tm_map(lastWords, toSpace, "\\|")
lastWords <- tm_map(lastWords, toSpace, "\x89")
lastWords <- tm_map(lastWords, toSpace, "xdb")
lastWords <- tm_map(lastWords, toSpace, "xcf")
lastWords <- tm_map(lastWords, toSpace, "xe5")
lastWords <- tm_map(lastWords, toSpace, "xca")

# Convert the text to lower case
lastWords <- tm_map(lastWords, content_transformer(tolower))

# Remove numbers
lastWords <- tm_map(lastWords, removeNumbers)

# Remove english common stopwords
lastWords <- tm_map(lastWords, removeWords, stopwords("english"))

# Remove punctuations
lastWords <- tm_map(lastWords, removePunctuation)

# Eliminate extra white spaces
lastWords <- tm_map(lastWords, stripWhitespace)
```

Now we'll create the term document matrix (TDM).

```{r}
# Construct term document matrix
dtm <- TermDocumentMatrix(lastWords)
m <- as.matrix(dtm)
v <- sort(rowSums(m),decreasing=TRUE)
d <- data.frame(word = names(v),freq=v)
head(d, 10)
```

To get a feel for the most common words, lets look at the top 75 that occur at least 20 times.  We'll viualize this in a word cloud.

```{r}
# Create word cloud
set.seed(1234)
wordcloud(words = d$word, freq = d$freq, min.freq = 20,
          max.words=75, random.order=FALSE, rot.per=0.00, 
          colors=brewer.pal(4, "RdGy"))
```
Love is the most spoken word.  We see thoughts of remorse (sorry and foregiveness), focus on religion/spirituality (God, lord, pray, etc.) and family.

To see the top 10 words in a more concise format, we'll construct a bar chart.

```{r}
# Plot word frequencies
# Bar chart
barplot(d[1:15,]$freq, las = 2, names.arg = d[1:15,]$word,
        col ="grey", main ="Most Spoken Last Words",
        ylab = "Frequency")
```
Again, we see that "love" is the most used word.  

[THE END]


