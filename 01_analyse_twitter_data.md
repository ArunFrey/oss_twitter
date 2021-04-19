---
title: "Oxford Spring School in Advanced Research Methods"
author:
  name: Arun Frey & Christopher Barrie
  affiliation: University of Oxford & University of Edinburgh
output: 
  html_document:
    theme: flatly
    highlight: haddock
    # code_folding: show
    toc: yes
    toc_depth: 4
    toc_float: yes
    keep_md: true
---



# Using Twitter for Social Science Research

This online tutorial is designed to give you some first hands-on experience with accessing and analysing Twitter data for social science research. 

The content covered in this tutorial is by no means exhaustive---it is designed to give you a first taste of what you can do with data from the Twitter API.

## What's covered
In this tutorial you will learn how to: 

* Get developers access credentials to Twitter data

* Use the `rtweet` package to query the Twitter API

* Locate tweeters on map

* Collect different types of data on protest

* Predict user type from account (bot yes/no)


## Setup

### Installing `R`

We will be using `R` in the exercises below and will assume some familiarity with the programming language. 

If you haven't used `R` before or would like to refresh your memory, a great point of reference is the book "[_R For Data Science_](https://r4ds.had.co.nz)" by Hadley Wickham. 

You can install `R` by clicking [here](https://www.r-project.org). We also recommend using [RStudio](https://www.rstudio.com) for an easy-to-use development environment. 

### Packages

We will be using the following packages in the exercise, so make sure to have them installed and loaded. 


```r
# install packages (you only need to do this once)
install.packages("tidyverse")
install.packages("rtweet")
install.packages("kableExtra")
install.packages("magick")
install.packages("rgdal")
install.packages("sp")
install.packages("tidytext")
install.packages("devtools")
library(devtools)
install_github("pablobarbera/twitter_ideology/pkg/tweetscores")
```


```r
# load packages
library(tidyverse)
library(rtweet)
library(kableExtra)
library(magick)
library(rgdal)
library(sp)
library(tidytext)
library(tweetscores)

# setting the plot theme
theme_set(theme_minimal())
```

## Connect to Twitter's API

Below we will briefly describe how to obtain access to Twitter's API. For a more detailed description of the authenticaction process, read the following vignette by Michael W. Kearney, the creator of the package. 


```r
vignette("auth", package = "rtweet")
```


To connect to Twitter's API, you need a **consumer key** and a **consumer secret**, both of which you get by creating a Twitter App. 

To create an app, you will first need to apply for a developer account. To do so, create a Twitter acount or log into your existing one, and go to the Twitter [developer portal](https://developer.twitter.com/en). 

Click on _Apply_ in the navigation bar on the top right of the page, and fill in all the relevant information before submitting your application. Your application will then be reviewed by Twitter before access is granted. This might take hours or days. 

Once you have acquired a developers account, navigate to [developer.twitter.com/apps](developer.twitter.com/apps) and "Create a New App". This includes the name, description and reasons of use for the app. This is Chris' account, and you can see that he has registered several apps for different purposes.

![](images/twitterdev3.png){width=100%}

After creating an app, you will be given a **consumer key** (or "API key") and **consumer secret** (or "API secret key"), which you will use to interact with the Twitter API. You **MUST** make a record of these. 

Select your Twitter app and click on the tab labelled "keys and tokens". Click on "Create" to obtain your four keys, and copy and paste them into an R script file in order to create and permanently store your access token. 


```r
## store api keys (these are fake example values; replace with your own keys)
api_key <- "afYS1vbIlPAj016E30c4W1fiK"
api_secret_key <- "bI93kqnqFoNCrZFbsjAWHD4gJ91LQAhdCJXCj3yscfuULtNkuu"
access_token <- "9531451262-wK2EmA942kxZYIwa5LMKZoQA4Xc2uyIiEwu2YXL"
access_token_secret <- "3vpiSGKg1fIPQtxc5d5ESiFlZQpfbknEN1f1m2xe5byw7"

## authenticate via web browser
token <- create_token(
  app = "OSS Research",
  consumer_key = api_key,
  consumer_secret = api_key_secret,
  access_token = access_token,
  access_secret = access_token_secret
)
```

The `create_token()` function will save your access token as an environment variable for you. This way, the `rtweet` package will automatically find the token next time you try to access the Twitter API. 

Once you have all of these keys and tokens recorded somewhere safe, you are ready to collect data!

## Querying the Twitter API with `rtweet`

The `rtweet` package makes it very easy to collect and analyse Twitter data, including individual tweets, or follower and friendship networks.

Let's begin by collecting the last tweets mentioning the hashtag #BLM or #BlackLivesMatter. We are collecting 1500 tweets here, but you can choose a higher or lower number of tweets. Note that, to return more than 18,000 tweets in a single call, users must set `retryonratelimit = TRUE`. Here, we have set `include_rts = FALSE` meaning that all of our tweets are original tweets rather than retweets.


```r
blm_tweets <- search_tweets("#BLM OR #BlackLivesMatter", 
                           n = 1500, include_rts = FALSE)
```

Since many of you may not have obtained a developers account, we have uploaded a sample dataset to allow you to familiarise yourself with data from the Twitter API. To download the data, clone [our Github repository](https://github.com/ArunFrey/oss_twitter), navigate to the downloaded folders, and run all code in the corresponding R markdown file (`01_analyse_twitter_data.Rmd`). 

Alternatively, you can also download all datasets directly from the Github website. Note that you will have to add "https://raw.githack.com/ArunFrey/oss_twitter/main/" to all subsequent file paths to download the data, so we encourage working from within the cloned Github repo instead.


```r
blm_tweets <- readRDS("data/BLMtweets.rds")
```


```r
# load the data from within the cloned Github folder
blm_tweets <- readRDS("data/BLMtweets.rds")

# alternatively, download the data directly from Github
blm_tweets <- readRDS(url("https://raw.githack.com/ArunFrey/oss_twitter/main/data/BLMtweets.rds"))
```

The Twitter API allows us to retrieve a lot of information about tweets and users, but let's stick with a few for now. 

<table class="table table-striped table-hover table-condensed table-responsive" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> created_at </th>
   <th style="text-align:left;"> screen_name </th>
   <th style="text-align:left;"> text </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 2021-03-12 09:35:14 </td>
   <td style="text-align:left;"> Purge321 </td>
   <td style="text-align:left;"> Sorry, we don't stand with your rhetoric. The moment your ALM hashtag came out was the turning point. 

#BlackLivesMatter #Asians4BlackLives https://t.co/CltI8nW2AH </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-03-12 09:36:34 </td>
   <td style="text-align:left;"> vernon78784378 </td>
   <td style="text-align:left;"> @ASK_des MY @Labour of 1950's destroyed. Replaced by #woke, #antisemitic, #antizionist, #antiwhite, #misandrist, #antipolice #BLM, #rejoiner economically illiterate factions with @Keir_Starmer's front bench squabbling like fizzing lumps risen up in a cess pit @labouryouth @youngfabians https://t.co/1qoWiUG1kD </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-03-12 09:38:37 </td>
   <td style="text-align:left;"> KatlynSkett </td>
   <td style="text-align:left;"> Keir starmer 
AKA  NETANYAHU PUPPET 
with his blairites AKA LFI 
Are not only purging the left 
But have been openly conspiring  against them since #corbyn
Was elected leader 
#LabourLeaks shows that evidence 
#FreePalestine 
#corbynWasRight 
#StarmerOut 
#StarmerIsARacist #BLM https://t.co/NabTJnWXbf https://t.co/sGTAlBOXFE </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-03-12 09:41:48 </td>
   <td style="text-align:left;"> SHAWNTAKAS </td>
   <td style="text-align:left;"> 🤣🤣Ugandan Police score a distinction in corruption.

What would your country's police score in your opinion?🤔
#WitsProtests #corruption #PoliceBrutality #BlackLivesMatter #Police https://t.co/QPFTVaeXcQ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-03-12 09:45:47 </td>
   <td style="text-align:left;"> culturereviewed </td>
   <td style="text-align:left;"> When Wits black students were fighting for the doors of learning to be open as the Freedom Charter promises, police responded with violence. Those are the "black crowd control" techniques they know. Nothing else
#WitsProtest #BlackLivesMatter #CReview </td>
  </tr>
</tbody>
</table>

These tweets all occur within quick succession of each other (Note: here, the tweets were collected in advance of the workshop, explaining why the dates are not more recent). In fact, we can visualise the frequency of the tweets. We could use the `ggplot` package for this, but `rtweet` already has a built in function to easily visualise time serie of Twitter data.


```r
# plot a time series of all tweets in our data.
ts_plot(blm_tweets, "1 hour") + 
  ylab("Number of tweets") 
```

![](01_analyse_twitter_data_files/figure-html/plot timeline-1.png)<!-- -->

We can see that all 1500 tweets capture only a small fraction of the online discussion surrounding Black Lives Matter on Twitter. What's more, the plot reveals that users are more active during certain periods of the day.

# Studying protests using Twitter 

Next, we turn our attention to using Twitter data to study protest events, and focus on the 2017 Women's March protests. Protests are notoriously hard to survey, and Twitter can potentially provide us with valuable insights into who is participating in a demonstration. 

Below is a map of all geolocated tweets that were sent on January 21, 2017, the day of the Women's March protest, showing that users across the world tweeted about the event.

![](01_analyse_twitter_data_files/figure-html/load WM data-1.png)<!-- -->

We can manipulate these data into a `SpatialPointsDataFrame`, making sure the CRS is correctly defined, allowing us to plot the points easily using base R plotting functions. The CRS stands for "Coordinate Reference System," which controls the "projection" of the map we wish to visualize--i.e., how it looks. For more info. on map projections, see [this guide](https://docs.qgis.org/2.8/en/docs/gentle_gis_introduction/coordinate_reference_systems.html).


Manipulating the data in this way will be helpful when clipping to the boundaries of shapefiles as we go on to describe below.

Let's begin by loading the data: 

```r
# load the data 
wm_geo <- readRDS("data/geo_raw.rds") %>% 
  mutate_all(as.numeric) 
```


```r
# alternatively, load direclty from the github website
wm_geo <- readRDS(url("https://raw.githack.com/ArunFrey/oss_twitter/main/data/geo_raw.rds")) %>% 
  mutate_all(as.numeric) 
```


```r
# generate a spacialpoints dataframe (order must be long/lat)
xy <- wm_geo[,c(1,2)]
points <- SpatialPointsDataFrame(coords = xy, data = xy,
                               proj4string = CRS("+proj=longlat +ellps=WGS84 +datum=WGS84 +no_defs"))

plot(points)
```

![](01_analyse_twitter_data_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

These data have been stripped of user identifying information, including user name, bio etc. Instead we just have two columns: latitude and longitude. The points are from all tweets that contained in the #WomensMarch. When we plot the simple latitude and longitude of the points, we can make out the vague outline of countries. 

In order to identify points within the routes of our targeted protest marches, we first read in three shapefiles. Each of these have been created by drawing a buffer of increasing sizes around the route of the 2017 Washington D.C. Women's March. 


```r
route50 <- readOGR("shapes/wm_route_example50.shp", verbose = FALSE)
route100 <- readOGR("shapes/wm_route_example100.shp", verbose = FALSE)
route1000 <- readOGR("shapes/wm_route_example1000.shp", verbose = FALSE)

# we also load an image of the women's march protest
routeimg <- image_read("images/wm_march_map.jpg")
```

We can compare these to the original march route below:


```r
par(mfrow=c(2,2))
plot(routeimg)
plot(route50, main="50m buffer")
plot(route100, main="100m buffer")
plot(route1000, main="1000m buffer")
```

![](01_analyse_twitter_data_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

We can create these shapefiles with relative ease in open-source GIS softwares like QGIS.

It is not completely necessary to use these more accurate geographic projections of protest routes, however. In fact, the use of a rectangular bounding box is able to capture these same protestors, with limited cost in terms of inaccuracy. To find the coordinates of a bounding box, we recommend using the open-source [*OpenStreetMap*](https://www.openstreetmap.org) platform.

As shown below, by searching a location in OpenStreetMap, and selecting the "Export" option at the top of the window, we are able to view the coordinates of the left-upper and right-lower diagonals of the map displayed in the viewer window. The user can zoom in and out on this map in order to select an appropriate geographical area.

![](images/osm_figure.png){width=100%}

To generate a rectangular bounding box object from these four coordinates, we simply need to combine them into a matrix for the purposes of plotting. We can then convert this into a spatial object, and assign the relevant CRS--the same as we assigned to our spatial points above.

We show the coordinates in the image above. From these coordinates, we can easily now generate a SpatialPolygons bounding box by combing the x1, y1, x2, and y2 coordinates into a matrix


```r
#bounding box
x1<- -77.0722
y1<- 38.9145
x2 <- -76.9786
y2 <- 38.8660
coords = matrix(c(x1, y1,
                  x1,y2,
                  x2,y2,
                  x2,y1,
                  x1,y1),
                ncol = 2, byrow = TRUE)
P1 <- Polygon(coords)
bb <- SpatialPolygons(list(Polygons(list(P1), ID = "a")), 
                      proj4string=CRS("+proj=longlat +ellps=WGS84 +datum=WGS84 +no_defs"))

plot(bb)
```

![](01_analyse_twitter_data_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

We compare our bounding box shape to our route buffer shapes below. 


```r
#set consistent CRS
CRS.new<-CRS("+proj=longlat +ellps=WGS84 +datum=WGS84 +no_defs")
proj4string(route50) <- CRS.new 
proj4string(route100) <- CRS.new 
proj4string(route1000) <- CRS.new 

par(mfrow=c(2,2))
plot(bb)
plot(route50,add=T)
plot(bb)
plot(route100,add=T)
plot(bb)
plot(route1000,add=T)
plot(bb)
```

![](01_analyse_twitter_data_files/figure-html/plot boxes with protestors-1.png)<!-- -->

We can see that our rectangular bounding box shape covers a larger area than march route shapes. If we think this bounding box is too large, we can always reduce it in size by lifting new coordinates from OpenStreetMap, converting to a matrix, and generating a smaller spatial bounding box. For now, we will continue with the bounding box we have generated.



```r
par(mfrow=c(2,2))

plot(route50)
pts_subset <- points[route50,]
plot(pts_subset, add=T, col="blue")


plot(route100)
pts_subset <- points[route100,]
plot(pts_subset, add=T, col="green")

plot(route1000)
pts_subset <- points[route1000,]
plot(pts_subset, add=T, col="red")

plot(bb)
pts_subset <- points[bb,]
plot(pts_subset, add=T, col="black")
```

![](01_analyse_twitter_data_files/figure-html/unnamed-chunk-8-1.png)<!-- -->


```r
# plot all points that fall within buffer box
plot(bb, axes = T)
pts_subset <- points[bb,]
plot(pts_subset,add=T)

# add 1000m buffer (in red)
plot(route1000, add=T)
pts_subset <- points[route1000,]
plot(pts_subset, add=T, col="red")

#add 100m buffer (in green)
plot(route100, add=T)
pts_subset <- points[route100,]
plot(pts_subset, add=T, col="green")

# add 50m buffer (in blue)
plot(route50, add=T)
pts_subset <- points[route50,]
plot(pts_subset, add=T, col="blue")
```

![](01_analyse_twitter_data_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

One of the challenges with Twitter data is that it is unclear whether someone who tweets about a protest actually participates in it. Information on the geo-location of users allows us to assess whether or not a user tweeted from within the protest march. 

The above, using open source GIS softwares, means we can **easily locate individuals to within the route of a protest march**, providing a confident measure of participation.

# Protest hashtags
We have also provided you with a sample dataset containing a subset of 500 users who tweeted from D.C about the Women's March on the day of the protest. We have changed the names and status ids of all tweets in the data, and have only uploaded information on a few key variables.

To load the dataset, run the following code: 


```r
# load data
wm_dc <- read_csv("data/wm_dc.csv")

# examine the data 
wm_dc %>% 
  select(status_id, screen_name, created_at, hashtags) %>%
  head(5) %>%
  kbl() %>%
  kable_styling(c("striped", "hover", "condensed", "responsive"))
```

<table class="table table-striped table-hover table-condensed table-responsive" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> status_id </th>
   <th style="text-align:left;"> screen_name </th>
   <th style="text-align:left;"> created_at </th>
   <th style="text-align:left;"> hashtags </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 04f8ecca36a6c773 </td>
   <td style="text-align:left;"> amusing_mule </td>
   <td style="text-align:left;"> Sat Jan 21 23:59:44 +0000 2017 </td>
   <td style="text-align:left;"> womensMarch </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 6b47bbc6904c6383 </td>
   <td style="text-align:left;"> evilminded_springpeeper </td>
   <td style="text-align:left;"> Sat Jan 21 23:59:22 +0000 2017 </td>
   <td style="text-align:left;"> Trump WomensMarch </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 4fd844a06013dbce </td>
   <td style="text-align:left;"> nonirrational_iceblueredtopzebra </td>
   <td style="text-align:left;"> Sat Jan 21 23:57:34 +0000 2017 </td>
   <td style="text-align:left;"> womensmarch </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 9b4d721dcdd8fbb4 </td>
   <td style="text-align:left;"> frigid_junebug </td>
   <td style="text-align:left;"> Sat Jan 21 23:56:13 +0000 2017 </td>
   <td style="text-align:left;"> womensmarch womensmarchonwashington </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 910e8cfc0a5597d2 </td>
   <td style="text-align:left;"> benignant_nabarlek </td>
   <td style="text-align:left;"> Sat Jan 21 23:55:28 +0000 2017 </td>
   <td style="text-align:left;"> womensmarchonwashington womensmarch </td>
  </tr>
</tbody>
</table>

We can look at which hashtags were used most frequently during the march in DC. To do that, we use the `hashtags` variable, which lists the hashtags used in each tweet. To separate multiple hashtags into individual rows, we use the `unnest_tokens` command from the `tidytext` package. The plot below visualises all hashtags that were used at least 10 times during the 2017 Women's March in DC. 


```r
wm_dc %>% 
  unnest_tokens(output = hashtag, input = hashtags) %>%  # this divides words separated by space into individual rows
  group_by(hashtag) %>%
  count() %>% 
  na.omit() %>%
  filter(n >= 10) %>%
  ggplot(aes(x = n, y = reorder(hashtag, n))) +
  geom_bar(stat = "identity") + 
  ylab("Hashtag") + 
  xlab("")
```

![](01_analyse_twitter_data_files/figure-html/plot hashtags-1.png)<!-- -->


# Estimating ideology

Once we have located our protestor-users, the estimation of their ideological position (based on their follow network) is straightforward using the `tweetscores` package by Pablo Barberá. We will not estimate the ideologies of our users above as they have been anonymized. But you can certainly look at your own: simply change the user name to your own Twitter username. 
Note: you will also need to use the authentication token (here: `my_oauth_CB`) you created above to download the following network of Twitter users. For more information follow the steps outlined by Barberá [here](https://github.com/pablobarbera/twitter_ideology)


```r
library(tweetscores)

user <- "cbarrie"
friends <- getFriends(screen_name=user, oauth = "my_oauth_CB")

ideo <- estimateIdeology(user, friends)

plot(ideo)
```

![](01_analyse_twitter_data_files/figure-html/plot chris ideology-1.png)<!-- -->

If you want to estimate the ideology for multiple users, we suggest opting for Maximum Likelihood estimation, which is considerably faster. You can do this by using the `estimateIdeology2` function (see [here](https://github.com/pablobarbera/twitter_ideology/blob/master/pkg/tweetscores/R/utils.R) for the estimation functions for each of these two approaches.)


# Some other ways to use Twitter data

This is only the beginning of what we can do with Twitter data. The code below uses the [`tweetbotornot2`](https://github.com/mkearney/tweetbotornot2) package by Michael Kearney, the author of the `rtweet` package, and predicts how many accounts in our Black Lives Matter tweets dataset are likely bots.

We plot a histogram of predicted likely bot accounts below, along with a short selection of some of the tweets.


```r
library(remotes) # install remotes package if necessary
library(tweetbotornot2) # install from github if necessary 

bots_p <- predict_bot(blm_tweets)
```

```
## [10:37:35] WARNING: amalgamation/../src/learner.cc:790: Loading model from XGBoost < 1.0.0, consider saving it again for improved compatibility
```

```r
ggplot(bots_p) +
  geom_histogram(aes(prob_bot)) +
  labs(x= "Probability of being a bot", y= "Count")
```

![](01_analyse_twitter_data_files/figure-html/predict bots-1.png)<!-- -->


<table class="table table-striped table-hover table-condensed table-responsive" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> screen_name </th>
   <th style="text-align:left;"> text </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> heyman_bot </td>
   <td style="text-align:left;"> #BLM </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PortsideOrg </td>
   <td style="text-align:left;"> Amy Sherald Directs Her Breonna Taylor Painting Toward Justice  #BreonnaTaylor #Louisville #policekillings #DefundthePolice #police #AfricanAmericanart #BLM #BlackLivesMatter https://t.co/HXUtAerPeS </td>
  </tr>
  <tr>
   <td style="text-align:left;"> culturereviewed </td>
   <td style="text-align:left;"> When Wits black students were fighting for the doors of learning to be open as the Freedom Charter promises, police responded with violence. Those are the "black crowd control" techniques they know. Nothing else
#WitsProtest #BlackLivesMatter #CReview </td>
  </tr>
  <tr>
   <td style="text-align:left;"> say_the_names </td>
   <td style="text-align:left;"> Willie McCoy #BlackLivesMatter </td>
  </tr>
  <tr>
   <td style="text-align:left;"> say_the_names </td>
   <td style="text-align:left;"> Michael Dean #BlackLivesMatter </td>
  </tr>
  <tr>
   <td style="text-align:left;"> BLMProtestBot </td>
   <td style="text-align:left;"> If you're reading this, remember that #BlackLivesMatter </td>
  </tr>
  <tr>
   <td style="text-align:left;"> say_the_names </td>
   <td style="text-align:left;"> John Crawford III #BlackLivesMatter </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tbasharks </td>
   <td style="text-align:left;"> Wed Jul 29 2020 Portland, Oregon - Independent journalist arrested WATCH: https://t.co/3PUJqzKHfX #PortlandOregon #PPD #blacklivesmatter #blm #defundthepolice #abolishthepolice </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tbasharks </td>
   <td style="text-align:left;"> Sat May 30 2020 Austin, Texas - Police shoot non-violent protester in the head WATCH: https://t.co/tMCKqCxjkp #AustinTexas #APD #blacklivesmatter #blm #defundthepolice #abolishthepolice </td>
  </tr>
  <tr>
   <td style="text-align:left;"> TheLivesThatMtr </td>
   <td style="text-align:left;"> Say their name. Derrick Lee Hunt, 2015-08-07 #BlackLivesMatter. </td>
  </tr>
</tbody>
</table>



