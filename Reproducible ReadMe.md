# Reproducibility ReadMe for INSC 590.001 Data Software Preservation Project
This file describes how to reproduce all analyses from the ANTH 511 Twitter Analysis project conducted by Yasmin Stoss, Spring 2019

# Version and System Information
platform       x86_64-w64-mingw32          
arch           x86_64                      
os             mingw32                     
system         x86_64, mingw32             
status                                     
major          3                           
minor          5.2                         
year           2018                        
month          12                          
day            20                          
svn rev        75870                       
language       R                           
version.string R version 3.5.2 (2018-12-20)
nickname       Eggshell Igloo

#Software (R Studio)
RStudio Team (2016). RStudio: Integrated Development for R.
RStudio, Inc., Boston, MA URL http://www.rstudio.com/.

Version: 1.1.463

# R Packages/Libraries
Plotly, lubridate, tidytext, tidyverse, lmtest, RColorBrewer, wordcloud

# R Code to install and initialize packages
NOTE: all code will be indicated with a "*"
* install.packages(c("lubridate", "tidytext", "tidyverse", "lmtest", "RColorBrewer", wordcloud"))
* install.packages(“plotly”, repos=“http://cran.rstudio.com/ 229”, dependencies=TRUE)
* packages <- c("lubridate", "tidytext", "tidyverse", "lmtest", "RColorBrewer", wordcloud", "plotly")
* lapply(packages, library, character.only = TRUE)

# Steps and R Code for Project Analyses
1.  Download "Clemson.csv" onto your local hard drive
2.  Open R Studio
3.	Read in csv file
    * tweets <-read.csv("Clemson.csv")
4.	Format dates in the " " column of the spreadsheet. This enables analyses using dates and timeframes. 
    * tweets_dated <-tweets %>% mutate(Date = mdy_hm(publish_date))
5. Rename the "Content" column to "Tweet". This makes things less confusing down the line
    * colnames(tweets_dated)[colnames(tweets_dated)=="content"] <-"Tweet"
6. Clean data by removing URLs and other unwanted characters
    * replace_reg <-"https://t.co/[A-Za-z\\d]+|http://[A-Za-z\\d]+|&amp;|&lt;|&gt;|https"
    * unnest_reg <-"([^A-Za-z_\\d#@']|'(?![A-Za-z_\\d#@]))"
    * clean_tweets<-tweets_dated%>%mutate(Tweet=str_replace_all(Tweet,replace_reg,""))
7.	Sort out all tweets containing the keyword(s): 
    * keywords<-c("vacc", “Vacc", “vax”, “Vax”, “clean coal", “Clean Coal”, "paris climate accord", “Paris Climate Accord”, "ebola", “Ebola, "zika", “Zika, "dakota access pipeline", “Dakota Access Pipeline”,"pasteuriz", “Pasteuriz”, “frack”, “Frack” )
    * tweets_sci_words<-clean_tweets%>%filter(str_detect(Tweet,paste(keywords, collapse="|")))
8. Count instance of each keyword
    Vaccines (result = 821): 
    *sum(str_count(tweets_sci_words$Tweet, "vacc")) AND sum(str_count(tweets_sci_words$Tweet, "Vacc")) AND sum(str_count(tweets_sci_words$Tweet, "Vax")) AND sum(str_count(tweets_sci_words$Tweet, "vax"))
c.	Ebola: Ebola = 556
d.	Zika: Zika = 2125
e.	Paris Climate Accord: Paris climate accord = 31
f.	Clean Coal: Clean coal = 27
g.	Dakota Access Pipeline: Dakota access pipeline = 50
h.	Pasteurization: Pasteuriz = 0
i.	Fracking: Frack = 174
6.	Plot instance of each keyword in a donut plot
a.	Keywords_count <- read.csv(“Keywords_count.csv”)
b.	Keywords_count %>%  group_by(keyword) %>% plot_ly(labels=~keyword, values=~count) %>% add_pie(hole=0.3)
7.	Create a data frame with the tweets separated out into single words
a.	sci_words<-tweets_sci_words%>%unnest_tokens(word,Tweet,token="regex",pattern=unnest_reg)%>%filter(!word %in% stop_words$word,str_detect(word, "[a-z]"))
8.	Filter out the keywords. 
a.	keywords2<-c("vacc", "Vacc", "vax", "Vax", "clean", "coal", "Clean", "Coal", "paris", "climate", "accord", "Paris", "Climate", "Accord", "ebola", "Ebola", "zika", "Zika", "dakota", "access", "pipeline", "Dakota", "Access", "Pipeline","pasteuriz", "Pasteuriz", "frack", "Frack" )
b.	sci_words2<-sci_words%>%filter(!str_detect(word,paste(keywords2, collapse="|")))
9.	Create a new data frame including the sentiments for each word
a.	nrc<-get_sentiments("nrc")
b.	sci_tweet_sentiment <-sci_words2 %>% inner_join(nrc) %>% count(index = Date, sentiment) %>% spread(sentiment, n, fill = 0)
10.	Create a word cloud for the top 20 words for the entire data set, the positive words, the negative words
a.	pal <- brewer.pal(9,"RdBu")
b.	wordcloud(words = count_sci_word$word, freq = count_sci_word$n, min.freq = 1, max.words=20, color=pal)
c.	wordcloud(words = positive_all$word, freq = positive_all$n, min.freq = 1, max.words=20, color=pal)
d.	wordcloud(words = Negative_all$word, freq = Negative_all$n, min.freq = 1, max.words=20, color=pal)
11.	Plot the negative-positive cumulative sum
a.	sci_tweet_sentiment %>% ggplot(aes(x=as.Date(index), y=cumsum(negative-positive))) + geom_line() 
12.	Plot the fear-trust cumulative sum
a.	sci_tweet_sentiment %>% ggplot(aes(x=as.Date(index), y=cumsum(fear-trust))) + geom_line() 
13.	Normalize the data by daily tweets
a.	daily_sci <-sci_words2 %>% mutate(tweetDay= floor_date(Date, "day"))
b.	daily_sci_sentiment <-daily_sci %>% inner_join(nrc) %>% count(index = tweetDay, sentiment) %>% spread(sentiment, n, fill = 0) 
c.	colnames(daily_sci_sentiment)[colnames(daily_sci_sentiment)=="index"] <-"tweetDay"
d.	wordsPerDay <-count(daily_sci, tweetDay)
e.	SentimentsPerDay <-inner_join(daily_sci_sentiment, wordsPerDay)
14.	 Plot the negative-positive cumulative sum with normalized data (include election day and date of Trump’s running announcement)
a.	ggplot(data = SentimentsPerDay) + geom_area(aes(x=tweetDay, y=cumsum(((negative-positive)/n))),color="#00AFBB", size=2, fill="#00AFBB")+geom_vline(xintercept = as.POSIXct(as.Date(c("2016-11-08"))), linetype="dotted",color = "#daa520", size=2)+geom_vline(xintercept = as.POSIXct(as.Date(c("2016-06-16"))), linetype="dotted",color = "#709963", size=2)
15.	Plot the fear-trust cumulative sum with normalized data (include election day and date of Trump’s running announcement)
a.	ggplot(data = SentimentsPerDay) + geom_area(aes(x=tweetDay, y=cumsum(((fear-trust)/n))),color="#00AFBB", size=2, fill="#00AFBB")+geom_vline(xintercept = as.POSIXct(as.Date(c("2016-11-08"))), linetype="dotted",color = "#daa520", size=2)+geom_vline(xintercept = as.POSIXct(as.Date(c("2016-06-16"))), linetype="dotted",color = "#709963", size=2)
16.	Determine the top 10 tweeters by count of tweets
a.	count(sci_words2, Twitter_Name, sort=TRUE)
17.	Do a ribbon plot of the top 10 tweeters 
a.	sci_words2 %>%   mutate(Twitter_Name = fct_lump(Twitter_Name, 10)) %>%    count(month = round_date(Date, "month"), Twitter_Name) %>%  complete(month, Twitter_Name, fill = list(n = 0)) %>%     mutate(Twitter_Name = reorder(Twitter_Name, -n, sum)) %>%    group_by(month) %>%mutate(percent = n /sum(n),      maximum = cumsum(percent),  minimum = lag(maximum, 1, 0)) %>%  ggplot(aes(month, ymin = minimum, ymax = maximum, fill = Twitter_Name)) + geom_ribbon()+theme(legend.position="bottom", legend.title = element_blank())+geom_vline(xintercept = as.POSIXct(as.Date(c("2016-11-08"))), linetype="dotted",color = "black", size=1.5)
18.	 Filter out the top 10 tweeters and do a ribbon plot for just those 10 (add a line for the election day)
a.	top10_tweeters<-c("NEWSPEAKDAILY","TODAYPITTSBURGH","SCREAMYMONKEY","SEATTLE_POST","ROOMOFRUMOR","MILWAUKEEVOICE","KANSASDAILYNEWS","WASHINGTONLINE","DAILYSANJOSE","DAILYSANFRAN","TODAYBOSTONMA")
b.	Top10_sci_words<-tweets_sci_words%>%filter(str_detect(Twitter_Name,paste(top10_tweeters, collapse="|")))
c.	Top10_sci_words %>%   mutate(Twitter_Name = fct_lump(Twitter_Name, 10)) %>%    count(month = round_date(Date, "month"), Twitter_Name) %>%  complete(month, Twitter_Name, fill = list(n = 0)) %>%     mutate(Twitter_Name = reorder(Twitter_Name, -n, sum)) %>%    group_by(month) %>%mutate(percent = n /sum(n),      maximum = cumsum(percent),  minimum = lag(maximum, 1, 0)) %>%  ggplot(aes(month, ymin = minimum, ymax = maximum, fill = Twitter_Name)) + geom_ribbon()+theme(legend.position="bottom", legend.title = element_blank())+geom_vline(xintercept = as.POSIXct(as.Date(c("2016-11-08"))), linetype="dotted",color = "black", size=1.5)
19.	Determine the top 3 tweeters
a.	count(sci_words2, Twitter_Name, sort=TRUE)
b.	Top3_tweeters<-c("NEWSPEAKDAILY", "SCREAMYMONKEY","TODAYPITTSBURGH")
20.	Do a Granger Causality test between the top 3 tweeters and the rest of the data set
a.	all_timeSeries<-sci_words2%>%group_by(week=round_date(Date,"week"))%>%summarize(tweets=n(), account=sum(str_detect(Twitter_Name, paste(top10_tweeters, collapse="|"))), percent=account/tweets)
b.	newspeakdaily_timeSeries<-sci_words2%>%group_by(week=round_date(Date,"week"))%>%summarize(tweets=n(), account=sum(str_detect(Twitter_Name, "NEWSPEAKDAILY")), percent=account/tweets)
c.	screamymonkey_timeSeries<-sci_words2%>%group_by(week=round_date(Date,"week"))%>%summarize(tweets=n(), account=sum(str_detect(Twitter_Name, "SCREAMYMONKEY")), percent=account/tweets)
d.	todaypittsburgh_timeSeries<-sci_words2%>%group_by(week=round_date(Date,"week"))%>%summarize(tweets=n(), account=sum(str_detect(Twitter_Name, "TODAYPITTSBURGH")), percent=account/tweets)
e.	grangertest(newspeakdaily_timeSeries$percent,all_timeSeries$percent, order=1)
f.	grangertest(todaypittsburgh_timeSeries$percent,all_timeSeries$percent, order=1)
g.	grangertest(screamymonkey_timeSeries$percent,all_timeSeries$percent, order=1)
21.	Visualize the time series for the granger causality tests with significant p values
a.	ggplot()+geom_area(data=all_timeSeries, aes(x=as.Date(week), y=percent), color="#00AFBB", size=2, fill = "#00AFBB")+geom_area(data=screamymonkey_timeSeries, aes(x=as.Date(week), y=percent), color="#FC4E07", size=2, fill="#FC4E07")+xlab("Date")+ylab("% of tweets")+geom_vline(aes(xintercept=as.numeric(as.Date("2016-11-13"))), linetype=4, color="black")


