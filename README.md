# Analyzing Webtoon
Webtoon is a website originated in South Korea where users can find and read millions of digital comics. Such comics are categorized into either originals (whose creators are paid by Webtoon) or canvas (whose creators are unpaid).

Originals in Webtoon are either in *daily pass* or not. Daily pass series allow readers to read the first few episodes for free, while the latter episodes are only unlocked once per day for free or more via in-app payment. Daily pass series can be completed or ongoing series.  

This repository uses the datasets of WEBTOON ORIGINALS which can be found [here](https://www.kaggle.com/datasets/iridazzle/webtoon-originals-datasets).

***
### Remarks regarding calculating averages
For average rating and average subscribers: The sum of rating/subscribers of every originals within a specific genre divided by the total number of originals in that genre. For average views and likes: The views and likes of each original are first divided by the original length before calculating genre average.

The reason for this is because unlike rating and subscribers, views and likes are accumulated through the whole length of the original, i.e. originals with less views and likes for each chapter can surpass those with more views and likes per chapter if they are long enough. To keep things fair, we perform the average differently as mentioned earlier.

***
### Data cleaning
I performed a small data cleaning step by converting some columns that are in text format into int, bigint or float format, and save copy the converted **webtoon_originals** table into a new table **webtoon_fixed**
````sql
SELECT CONVERT(int, title_id) AS title_id
	,[title] ,[genre] ,[authors] ,[weekdays]
	,CONVERT(int, [length]) AS length
	,CONVERT(bigint, [subscribers]) AS subscribers
	,CONVERT(float, [rating]) AS rating
	,CONVERT(bigint, [views]) AS views
	,CONVERT(bigint, [likes]) AS likes
	,[status] ,[daily_pass] ,[synopsis]
INTO webtoon_fixed
FROM [dbo].[webtoon_originals]
````

*** 
### Questions
[1. What is the number of originals for each genre in Webtoon and how are their performances?](https://github.com/huongtxdo/SQL-analyzing-Webtoon/blob/main/solution.md#1-what-is-the-number-of-originals-for-each-genre-in-webtoon-and-how-are-their-performances)

[2. What are the top 10 originals in Webtoon excluding those in daily pass?](https://github.com/huongtxdo/SQL-analyzing-Webtoon/blob/main/solution.md#2-what-are-the-top-10-originals-in-webtoon-excluding-those-in-daily-pass)

[3. What are the top 10 originals in Webtoon including those in daily pass?](https://github.com/huongtxdo/SQL-analyzing-Webtoon/blob/main/solution.md#3-what-are-the-top-10-originals-in-webtoon-including-those-in-daily-pass)

[4. For each genre, what is the top originals excluding those in daily pass?](https://github.com/huongtxdo/SQL-analyzing-Webtoon/blob/main/solution.md#4-for-each-genre-what-is-the-top-originals-excluding-those-in-daily-pass)

[5. What are the weekday performances of Webtoon so far (including every original regardless of their status)?](https://github.com/huongtxdo/SQL-analyzing-Webtoon/blob/main/solution.md#5-what-are-the-weekday-performances-of-webtoon-so-far-including-every-original-regardless-of-their-status)

[6. What are the weekday performances of ONGOING Webtoon?](https://github.com/huongtxdo/SQL-analyzing-Webtoon/blob/main/solution.md#6-what-are-the-weekday-performances-of-ongoing-webtoon)

[7. Which authors attract the most views?](https://github.com/huongtxdo/SQL-analyzing-Webtoon/blob/main/solution.md#7-which-authors-attract-the-most-views)
