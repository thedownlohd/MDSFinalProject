---
title: "Hw06"
author: "Kendall Loh"
date: "10/30/2018"
output: 
  html_document:
    keep_md: true
---




```r
library('gutenbergr')
library('readr')
library('skimr')
library('tidyverse')
```

```
## -- Attaching packages ------------------------------------------------------------ tidyverse 1.2.1 --
```

```
## v ggplot2 3.1.0     v purrr   0.2.5
## v tibble  1.4.2     v dplyr   0.7.6
## v tidyr   0.8.1     v stringr 1.3.1
## v ggplot2 3.1.0     v forcats 0.3.0
```

```
## -- Conflicts --------------------------------------------------------------- tidyverse_conflicts() --
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library('stringr')
library('rebus')
```

```
## 
## Attaching package: 'rebus'
```

```
## The following object is masked from 'package:stringr':
## 
##     regex
```

```
## The following object is masked from 'package:ggplot2':
## 
##     alpha
```

```r
# Let's get the book "Ulysses" by James Joyce
gutenberg_works(author == "Joyce, James")
book <- gutenberg_download(4300)
```

```
## Determining mirror for Project Gutenberg from http://www.gutenberg.org/robot/harvest
```

```
## Using mirror http://aleph.gutenberg.org
```

```r
book

# Install and load the package `tidytext`. 
# install.packages(tidytext)
library(tidytext)
library(tidyverse)
library(stringr)

# Now let's get the words in the text
words <- book %>%
  unnest_tokens(word, text) %>%
  select(word)
words
```

a) Words with z
Select all unique words that contain at least one z. Among the z-words, tabulate how many z’s the words contain (i.e. how many words contain one z, two z’s etc.). Find the z-word(s) with z’s that are as far apart as possible (i.e. we are interested in the distance between two z’s in the word. That means the word could contain more than two z’s.).

Hint: You can focus on the distance between consecutive z’s. In this example, it does not make a difference to generalize to the distance between more than two z’s in the word and it is easier to do.




```r
words
zords <- str_subset(words$word, "z")

table_of_zords <- str_count(zords,
                         "z")

table(table_of_zords)
```





```r
zords_2 <- zords[table_of_zords == 2]
zords_2
distance_2 <- str_locate_all(zords_2, "z")
distance_2
distance_2a <- as.data.frame(distance_2)
zords_3 <- zords[table_of_zords == 3]
zords_3
distance_3 <- str_locate_all(zords_3, "z") 
distance_3a <- as.data.frame(distance_3)
distance_2a[1, 163] <- 2
distance_2a[2, 163] <- 4
colnames(distance_2a)[163] <- "start.81"
distance_2a[1, 164] <- 4
distance_2a[2, 164] <- 12
colnames(distance_2a)[164] <- "end.81"
distance_2a[3, ] <- distance_2a[2, ] - distance_2a[1, ]
lengths_total <- sort(unique(distance_2a[3, ]), decreasing = TRUE)
lengths_total
col_num <- which(lengths_total > 1)
lengths_total[ , col_num]



zords_2[3] 
zords_2[14] 
zords_2[62] 
zords_2[67] 
zords_2[68] 
zords_2[56] 
```

So basically each of these words are one word ahead of the table entries.

b) Vowels
How many unique words start and end with a vowel? Are there words that start with two or more vowels? Find and display the word(s) with the most consecutive vowels (anywhere in the word).


```r
vowels_start_end <- str_subset(words$word,
                   pattern = "^[aeiouAEIOU]+.[aeiouAEIOU]$")

length(vowels_start_end)
```

```
## [1] 1679
```
There are a total of 1679 unique words that start and ent with a vowel.

```r
vowels_2_st <- str_subset(words$word,
                          pattern = "^[aeiouAEIOU]{2,}")

length(vowels_2_st)
```

```
## [1] 2571
```
2571 unique words start with two or more vowels.

```r
long_vowels <- str_subset(words$word, pattern = "[aeiouAEIOU]{10,}")

table(long_vowels)
```

```
## long_vowels
##                dooooooooooog frseeeeeeeeeeeeeeeeeeeefrong 
##                            1                            1 
##                goooooooooood           iiiiiiiiiaaaaaaach 
##                            1                            1 
##               sooooooooooong           steeeeeeeeeeeephen 
##                            1                            1
```



c) English spelling
Empirically verify the rule “i before e except after c”. No need to become a linguist here; simply tabulate the proportion of words when the rule holds and when it does not.




```r
English <- sapply(words$word, tolower)
```

I hid the results to save space, but you can evaluate below to see the results for works and not works.


```r
#Works:
str_view(English, "(cei|[^c]ie)", match = TRUE)
#Doesn't work:
str_view(English, "(cie|[^c]ei)", match = TRUE)
```



### 2. MTA Delays

The file `mta.RDS` contains a dataframe of tweets on the [New York City subway services' official twitter account](https://twitter.com/NYCTSubway/) (`@NYCTSubway`). Let's try to extract some structured data from these tweets.



```r
tweets <- readRDS('C:/Users/kenda/Desktop/course_materials/Exercises/hw06/mta.RDS')
#View(tweets)
```

a) Reduce the data to direct tweets only by removing any retweets (variable `is_retweet` = TRUE) and replies to other users (starting with `@username`). Also only keep the columns `text` and `created_at`.


```r
cleantweets <- tweets %>% 
  filter(is_retweet == "FALSE") %>% 
  select(text, created_at)
class(cleantweets)
```

```
## [1] "tbl_df"     "tbl"        "data.frame"
```

```r
cleantweets1 <- cleantweets[!grepl("@", cleantweets$text),]


#View(cleantweets1)
```


b) Time of Delay

Let's focus on tweets that identify a **delay in the service** of one or more train lines. Here is an example:

> Northbound 2 and 3 trains are running with delays because of signal problems at 14 St.

Identify a pattern (or a combined set of patterns) to subset only tweets that identify the start of a delay. Explain briefly the choices you made here. 

I chose the word `delay` and the word `late` as potential markers of a delayed train. Based on my view of the tweets that announce either a delay or the resuming of service, most delay tweets seem to contain those two words.


```r
delay_view <- str_subset(cleantweets1$text,
         pattern = 
           "(delay|delays|delayed|Delay|Delays|Delayed|DELAYED|DELAYS|DELAY|late|Late|LATE)")
#View(delay_view)
```

Provide a table or visualization that shows delays by day of the week and time of day. Use the following time periods: mornings (6-10am), mid-day (10am-3pm), afternoon (3pm - 6pm), evening (6pm - 10pm), and night (10pm - 6am). What do you find?


```r
#View(cleantweets1)

split_times <- str_split_fixed(cleantweets1$created_at, pattern = " ", n = 2)
split_times_plus <- cbind(cleantweets1, split_times)
colnames(split_times_plus)[3] <- "Date"
colnames(split_times_plus)[4] <- "Time"
split_times_plus <- split_times_plus %>% 
  dplyr::select('text', 'Date', 'Time')

split_times_plus$Time <- format(strptime(split_times_plus$Time, tz = "" , format = "%H:%M:%S"), format = "%H:%M:%S")

#split_times_plus$Time <- as.numeric(gsub("\\:.*$", "", split_times_plus$Time))

split_times_plus$cat <- with(split_times_plus, ifelse(Time>=06 & Time<=10, "morning", ifelse(Time >= 10 & Time<=15, "midday", ifelse(Time>15 & Time<=18, "afternoon", ifelse(Time>=18 & Time <= 22, "evening", ifelse(Time>=22 & Time<=6,"night", "morning"))))))

delay_view <- str_detect(split_times_plus$text,
         pattern = 
           "(delay|delays|delayed|Delay|Delays|Delayed|DELAYED|DELAYS|DELAY|late|Late|LATE)")
sum(delay_view)
```

```
## [1] 4589
```

```r
delayed_text <- split_times_plus$text[delay_view]
View(delayed_text)


delayed_text <- as.data.frame(delayed_text)
colnames(delayed_text)[1] <- "text"
#delayed_text$text <- as.character(gsub("\\.*$", "", delayed_text$text))
View(delayed_text)

View(split_times_plus)
merged <- left_join(delayed_text, split_times_plus, by='text')
```

```
## Warning: Column `text` joining factor and character vector, coercing into
## character vector
```

```r
View(merged)
#View(split_times_plus)

merged$day <- weekdays(as.Date(merged$Date))
View(merged)
plot1 <- ggplot(merged, aes(day, ..count..)) + geom_bar(aes(fill = cat), position = "dodge")

plot1
```

![](Loh_Kendall_hw06_files/figure-html/unnamed-chunk-12-1.png)<!-- -->
According to my plot, a vast majority of the tweets about delays or late trains come in the midday hours. My hypothesis is that trains running at rush hour stack up towards 10AM, at which point trains are begin to be late enough that they are considered delayed, and the account tweets out the status update.


c) Type of Delay

Among the set of tweets in part b), try to categorize the types of delays. No need to be exhaustive but try to pick up the top 3-5 reasons for delays. Combine them into reasonable categories if necessary (e.g. signal problems, medical, technical problems, etc.). Provide an overview (table or graph) of which types of delays are most common.




```r
signal_delay <- merged[str_detect(merged$text,
         pattern = 
           "((S|s)ignal|signals)"),]

signal_delay$category="signal"

#View(signal_delay)

medical_delay <- merged[str_detect(merged$text,
         pattern = 
           "((M|m)edical|medicals)"),]
medical_delay$category="medical"

#View(medical_delay)

technical_delay <- merged[str_detect(merged$text,
         pattern = 
           "((T|t)echnical|technicals|Technicals|(t|T)ech)"),]
technical_delay$category="technical"

#View(technical_delay)
total <- rbind(signal_delay, medical_delay, technical_delay)
library(plyr)
```

```
## -------------------------------------------------------------------------
```

```
## You have loaded plyr after dplyr - this is likely to cause problems.
## If you need functions from both plyr and dplyr, please load plyr first, then dplyr:
## library(plyr); library(dplyr)
```

```
## -------------------------------------------------------------------------
```

```
## 
## Attaching package: 'plyr'
```

```
## The following objects are masked from 'package:dplyr':
## 
##     arrange, count, desc, failwith, id, mutate, rename, summarise,
##     summarize
```

```
## The following object is masked from 'package:purrr':
## 
##     compact
```

```r
count(total, "category")
```

```
##    category freq
## 1   medical  362
## 2    signal 1814
## 3 technical   65
```
According to my table, there were over 1800 signal delays, which dwarfs the other delay medical and technical reasons.

d) Which train lines affected?

Write a regex pattern that captures which train lines are affected by delays. We are not interested in other additional information (e.g. northbound vs. southbound, running express/local etc.). Provide a summary of which train lines are affected comparing weekday vs. weekend.






