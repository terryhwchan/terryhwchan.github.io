---
layout: post
title: GA Data Science Project 2 Post!
---

### MISSION STATEMENT FOR THE PROJECT: To get insight into the behavior of Billboard 100 rankings and how it relates to other variables of the songs

## Initial Observations...

* The data sets shows the Billboard Hot 100 data in 2000

* Corresponding song information such as artist, trackname, length of track, and genre are shown

* Date of entry and date of peaking are shown for each song

* Up to a maximum of 76 weeks of ranking is shown since the song entered the billboard (we can assume "xlst.week" is the relative to the song entry to the billboard, since none of the "x1st.week" is blank.

* All the tracks are in "year 2000", despite the fact that some of the "date.entered" is in 1999

* Initially, I am tempted to assume that these are ALL the songs that's have appeared on the billboard at some point in year 2000 (which means they must have "date.entered" either in or prior to year 2000

* However, with further look at all the unique dates, I realize that all the "date.peaked" are in year 2000, which makes me suspicious that NOT ALL the songs appeared on the billboard in year 2000 is shown in the data (since you would expect some songs that remained on the billboard in 2000 to have peaked in 1999. 

* Therefore, we can assume that ONLY songs whose ranking has peaked in 2000 are included. 

## Additional Observations after manipulating the data and plotting...
* There are two songs called "Where I Wanna be" by two different artists, we have renamed them "Where I Wanna Be (Jones)" and "Where I Wanna Be (Sheist)"

* The rankings are released on Saturdays as confirmed above by the "Date entered" "Date peaked" fields using the weekday() function

* Date Peaked is the FIRST week that a track hits the peak. From observing Destiny's Child's 'Independent Woman', we can confirm that"Date_Peaked" is the first week that the songs get to the highest ranking (ie. if the song is number 1 from week 9 to 19, week 9 is the "Date_Peaked". 

* It is not easy to aggregate statistics using "rankings" because:
    * Ranking is inverse: 1 is the best and 100 is the worst
    * Ranking is missing when a song is not on the billboard
    
* We introduced a new variable called **"Popularity Score"**:
    * For each week, popularity = [101 - weekly ranking], which means:
        * if a song is rank 1, it will have a popularity score of 100
        * if a song is rank 50, it will have a popluarity score of 51
        * if a song is rank 100, it will have a popularity score of 1
        * if a song is not on the billboard, it will have a popularity score of 0
    * Popularity Score allows us to:
        * Aggregate the weekly popularity into an overall track popularity:
            * for example, if a song is rank 1 for 5 weeks, that's an aggregate popularity of 500
        * Aggregate the track popularity to get the popularity of a particular artist or genre
            * We can take the sum or average of the popularity of each track

## Code Examples:

* melt the week numbers into a variable "Week" with corresponding value "Ranking", 
* saved into another df called bb_melt_df

bb_melt_df = pd.melt(bb_df, id_vars=["Year",
                        "Artist",
                        "Track",
                        "Time",
                        "Genre",
                        "Date Entered",
                        "Date Peaked",
                        "weeks_on_bb",
                        "avg_bb_ranking",
                        "Time_in_sec",
                        "Time_in_datetime", 
                        "Entered_datetime", 
                        "Peaked_datetime",
                        "Days_to_peak"], var_name="Week", value_name="Ranking")

* apply extract_num to the "Week" column to change weeks into numbers

bb_melt_df["Week"] = bb_melt_df["Week"].apply(extract_num) 

* adding the time_delta to the entered date to form "ranking date" and include in the bb_melt_df

for i in range(len(bb_melt_df["Week"])): # iterate thru every row
    time_delta = pd.Timedelta(days = (bb_melt_df.ix[i, "Week"]-1)*7) 
    bb_melt_df.ix[i, "Ranking Date"] = bb_melt_df.ix[i, "Entered_datetime"] + time_delta 

* Introducing another variable called "Popularity Score" that equals to [101 - weekly ranking], which means
    * if a song is rank 1, it will have a popularity score of 100
    * if a song is rank 50, it will have a popluarity score of 51
    * if a song is rank 100, it will have a popularity score of 1
    * if a song is not on the billboard, it will have a popularity score of 0
* This is a very useful tool to aggregate the popularity of a song over the entire duration

* def a function that changes a ranking into a popularity score
def popularity(ranking): 
    if ranking >= 0 and ranking <= 100:
        return 101 - ranking
    else:
        return 0
* apply it on the rankings and save into a new column

bb_melt_df["weekly_popularity"] = bb_melt_df["Ranking"].apply(popularity) 

# ![](https://raw.githubusercontent.com/terryhwchan/terryhwchan.github.io/master/1.png)
*Comment: Rock genre dominates the billboard 2000 dataset (in terms of number of tracks), followed by country and rap*

# ![](https://raw.githubusercontent.com/terryhwchan/terryhwchan.github.io/master/2.png)
*Comment: Days-to-peak seems to be highest at around 40 days and concentrate within the first 70 days, this makes sense given we would expect a song to be popular initially then drops off pretty quickly after the new favorite hits come out or when people are bored of the song. Note that the chart seems to suggest the days-to-peak data might follow a standard deviation. 

# ![](https://raw.githubusercontent.com/terryhwchan/terryhwchan.github.io/master/3.png)
*Comment: The above chart shows the ranking and popularity of each songs (see above for definition) over time. Note that popularity is simply [101 - ranking] for each week and that's why the charts look pretty much the same when inversed. Note the big gaps in some of the songs, which could be the result. The songs are ranked by the aggregate popularity of each song over time. Breathe by Faith Hill receives the highest aggregate popularity score.*

# ![](https://raw.githubusercontent.com/terryhwchan/terryhwchan.github.io/master/4.png)
*Comment: The above chart shows the weekly popularity and cumulative popularity as the weeks go by. As mentioned above, Breathe by Faith Hill receives the highest aggregate popularity score... of over 4000.*

# ![](https://raw.githubusercontent.com/terryhwchan/terryhwchan.github.io/master/5.png)
*Comment: Rock genre dominates the billboard 100 in terms of total track popularity (sum of popularity across all tracks over time) and track counts. Rock also has the highest avg track popularity. Other metrics are included for reference but note they do not suggest other genres are more popular given the lack of data for other genres on the billboard.*

# ![](https://raw.githubusercontent.com/terryhwchan/terryhwchan.github.io/master/6.png)
*Comment: Here is a lead table of all the songs using various metrics. Breathe by Faith Hill, Amazed by Lonestar, Kryptonite by 3 doors down are have the highest aggregate popularity. I believe track popularity is the best measure of song popularity given it accounts for both a) the height of the rank and b) the length of the rank. Also note that popular songs takes longer to peak, which makes sense given they last longer in general and may push the peak date to later.*

# ![](https://raw.githubusercontent.com/terryhwchan/terryhwchan.github.io/master/7.png)
*Comment: This is the distribution of aggregate popularity of each track, most tracks are in the 0-200 range given they stay on the billboard for short period of time. Again, this histogram seems to follow a probability distribution, looks a bit like poisson.*

# ![](https://raw.githubusercontent.com/terryhwchan/terryhwchan.github.io/master/8.png)
*Comment: This shows the lead table by artist: Destiny's Child (3 tracks), Creed (2 tracks), and N'Sync (3 tracks) have the highest overall popularity across their tracks. Other metrics are also included.*

# ![](https://raw.githubusercontent.com/terryhwchan/terryhwchan.github.io/master/9.png)
*Comment: Most data are within 220sec (3min40sec) - 255sec (4min-15sec) with avg around 235sec (3min55sec). The grey area shows the inter-quartile data, there's likely to be correlation between time and track popularity* 

# ![](https://raw.githubusercontent.com/terryhwchan/terryhwchan.github.io/master/10.png)
*Comment: The above chart shows the month that the track enters the billboard and its overall popularity, no clear conclusion can be drawn here*

# ![](https://raw.githubusercontent.com/terryhwchan/terryhwchan.github.io/master/11.png)
*Comment: Statistic summary table of all the top songs in the data set*

# ![](https://raw.githubusercontent.com/terryhwchan/terryhwchan.github.io/master/12.png)
*Comment: This shows the top 5 ranking songs for the entire year 2000, dates are properly matched using the weekly ranking*

#### PROBLEM STATEMENT: How to create a top ranking billboard song in general?

Independent Variables:

* Artist the song
* Length of the song
* Genre of the song
* Month of release the song

Dependent Variable:

* Aggregate popularity of the song = SUM of all weekly popularity score as defined by:
        weekly_popularity = 101 - weekly_ranking

From the billboard dataset, we have the following answers for the 4 questions for year 2000:

* Artist the song: Destiny's Child, Creed, N'Sync, Lonestar, and Christina Aguilara
* Length of the song: 220sec (3min40sec) - 255sec (4min-15sec)
* Genre of the song: ROCK!
* Month of release the song: Does not seem to matter too much

For the future data...similar to what we did above with the billboard data set:
* we can collect data in each of the four independent variables(Artist, Genre, Length, Month of release) and the dependent variable (billboard rankings => aggregate popularity)
* track the data over a longer period of time (instead only one year in our example)
* account for disppearance then reappearance on the billboard if appropriate (which would gives us a more realistic popularity of the song
* can collect addition data for the tracks such as: 
    * number of concurrent singles release by the same artist
    * marketing budget of the song
