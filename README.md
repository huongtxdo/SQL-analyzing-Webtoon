# Analyzing Webtoon
Webtoon is a website originated in South Korea where users can find and read millions of digital comics. Such comics are categorized into either originals (whose creators are paid by Webtoon) or canvas (whose creators are unpaid).

Originals in Webtoon are either in *daily pass* or not. Daily pass series allow readers to read the first few episodes for free, while the latter episodes are only unlocked once per day for free or more via in-app payment. Daily pass series can be completed or ongoing series.  

This repository uses the datasets of WEBTOON ORIGINALS which can be found [here](https://www.kaggle.com/datasets/iridazzle/webtoon-originals-datasets).

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
### Remarks regarding calculating averages
For average rating and average subscribers: The sum of rating/subscribers of every originals within a specific genre divided by the total number of originals in that genre. For average views and likes: The views and likes of each original are first divided by the original length before calculating genre average.

The reason for this is because unlike rating and subscribers, views and likes are accumulated through the whole length of the original, i.e. originals with less views and likes for each chapter can surpass those with more views and likes per chapter if they are long enough. To keep things fair, we perform the average differently as mentioned earlier.

***
### 1. What is the number of originals for each genre in Webtoon and how are their performances?
````sql
SELECT genre,
	COUNT(title_id) AS genre_originals_count,
	ROUND(AVG(rating),2) AS average_rating,
	AVG(subscribers) AS average_subscribers,
	AVG(views/length) AS average_views,
	AVG(likes/length) AS average_likes
FROM [webtoon].[dbo].[webtoon_fixed]
GROUP BY genre
ORDER BY genre_originals_count desc
````
#### Output:
| genre         | genre_originals_count | average_rating | average_subscribers | average_views | average_likes |
|---------------|-----------------------|----------------|---------------------|---------------|---------------|
| FANTASY       | 146                   | 9,5            | 442998              | 624338        | 54188         |
| ROMANCE       | 137                   | 9,38           | 767390              | 2172555       | 164990        |
| ACTION        | 93                    | 9,4            | 406440              | 781175        | 57244         |
| DRAMA         | 93                    | 9,46           | 378042              | 1266835       | 105140        |
| COMEDY        | 63                    | 9,2            | 354175              | 340343        | 31448         |
| SLICE_OF_LIFE | 54                    | 9,23           | 299763              | 313784        | 26291         |
| THRILLER      | 48                    | 9,41           | 481708              | 1755882       | 132400        |
| SUPERNATURAL  | 42                    | 9,44           | 469832              | 1427428       | 130247        |
| SF            | 40                    | 9,36           | 258711              | 313949        | 26805         |
| SUPER_HERO    | 30                    | 8,86           | 336570              | 484935        | 36613         |
| HORROR        | 24                    | 9,4            | 370832              | 916699        | 73604         |
| MYSTERY       | 14                    | 9,48           | 268925              | 187680        | 26297         |
| SPORTS        | 13                    | 9,25           | 475479              | 2800796       | 159622        |
| TIPTOON       | 7                     | 9,11           | 181389              | 185596        | 16739         |
| HISTORICAL    | 5                     | 9,54           | 269193              | 412703        | 60081         |
| HEARTWARMING  | 2                     | 9,7            | 236893              | 134425        | 21874         |

#### Remarks:
FANTASY and ROMANCE have significantly more originals than other genres, followed by ACTION and DRAMA.

Even though FANTASY has the most number of originals published, the top genres in term of popularity (number of subscribers, views and likes) are ROMANCE and SPORTS. With 10 times more originals than SPORTS, ROMANCE seems to be the attractive genre for readers.

The average rating is quite similar among genres, yet SUPER_HERO is not doing well being the only genre with a lower-than-9 rating. I guess just like superhero movies, it is hard to satisfy superhero comic readers.

***
### 2. What are the top 10 originals in Webtoon excluding those in daily pass?
We rank originals according to their rating, subscribers, views and likes.
We exclude daily pass originals because the data is scraped from the website, which show the views and likes as a sum of every episode, while the length (number of episodes) shown is only what available to read on the website (the rest has to be unlocked).
```sql
WITH rank_table AS
(SELECT title, title_id, genre,
		ROW_NUMBER() OVER (ORDER BY rating DESC) AS rating_rank,
		ROW_NUMBER() OVER (ORDER BY subscribers DESC) AS subs_rank,
		ROW_NUMBER() OVER (ORDER BY views/length DESC) AS views_rank,
		ROW_NUMBER() OVER (ORDER BY likes/length DESC) AS likes_rank
	FROM [webtoon].[dbo].[webtoon_fixed]
	WHERE daily_pass = 0) 
SELECT TOP 10 title_id, title, genre, rating_rank, subs_rank, views_rank, likes_rank,
	(rating_rank + subs_rank + views_rank + likes_rank) AS sum_rank
FROM rank_table
ORDER BY sum_rank
```

***
### 2. For each genre, what is the top originals excluding those in daily pass?
Originals are ranked according to their rating, subscribers, views and likes. Originals 
```sql
/* Create a temp_table that contains the ranking for every originals (excluding daily pass originals) */
WITH rank_table AS
(SELECT title, title_id, genre,
		ROW_NUMBER() OVER (ORDER BY rating DESC) AS rating_rank,
		ROW_NUMBER() OVER (ORDER BY subscribers DESC) AS subs_rank,
		ROW_NUMBER() OVER (ORDER BY views/length DESC) AS views_rank,
		ROW_NUMBER() OVER (ORDER BY likes/length DESC) AS likes_rank
	FROM [webtoon].[dbo].[webtoon_fixed]
	WHERE daily_pass = 0) 
SELECT title_id, title, genre,
	(rating_rank + subs_rank + views_rank + likes_rank) AS sum_rank
INTO temp_table
FROM
	rank_table;
```
