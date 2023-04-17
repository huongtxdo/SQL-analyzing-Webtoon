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

Let's create a rank_table that contains the rating for every originals (excluding daily pass originals)! We will reuse this table later.
```sql
WITH temp_table AS
(SELECT title, title_id, genre,
		DENSE_RANK() OVER (ORDER BY rating DESC) AS rating_rank,
		DENSE_RANK() OVER (ORDER BY subscribers DESC) AS subs_rank,
		DENSE_RANK() OVER (ORDER BY views/length DESC) AS views_rank,
		DENSE_RANK() OVER (ORDER BY likes/length DESC) AS likes_rank
	FROM [dbo].[webtoon_fixed]
	WHERE daily_pass = 0)
SELECT * INTO rank_table FROM temp_table
```
Now we query to answer the question!
```sql
SELECT TOP 10 *,
	(rating_rank + subs_rank + views_rank + likes_rank) AS sum_rank
FROM rank_table
ORDER BY sum_rank
```

### Output:
| title                 | title_id | genre      | rating_rank | subs_rank | views_rank | likes_rank | sum_rank |
|-----------------------|----------|------------|-------------|-----------|------------|------------|----------|
| Lore Olympus          | 1320     | ROMANCE    | 23          | 2         | 1          | 1          | 27       |
| unOrdinary            | 679      | SUPER_HERO | 18          | 3         | 3          | 5          | 29       |
| The Remarried Empress | 2135     | FANTASY    | 7           | 10        | 10         | 4          | 31       |
| I Love Yoo            | 986      | DRAMA      | 17          | 5         | 6          | 7          | 35       |
| Down To Earth         | 1817     | ROMANCE    | 12          | 8         | 7          | 13         | 40       |
| LUMINE                | 1022     | FANTASY    | 11          | 7         | 9          | 14         | 41       |
| SubZero               | 1468     | ROMANCE    | 15          | 9         | 11         | 9          | 44       |
| Let's Play            | 1218     | ROMANCE    | 35          | 4         | 4          | 3          | 46       |
| True Beauty           | 1436     | ROMANCE    | 42          | 1         | 2          | 2          | 47       |
| Castle Swimmer        | 1499     | FANTASY    | 9           | 14        | 17         | 10         | 50       |

### Remarks:
Half of the top 10 originals are in ROMANCE genre, and 3 out of 10 are in FANTASY. SUPER_HERO has 1 in top 10 even with much less originals published. Combined with the information from Question 1, I have 2 assumptions. 

(1) With a significantly higher number of originals in FANTASY and ROMANCE, the chances to find a good one are higher as compared to other genres.

(2) With more ROMANCE in top 10, we can assume that the overall quality of originals in FANTASY genre is more skewed than those of ROMANCE, with some really good ones and many not-so-good ones.

***
### 3. What are the top 10 originals in Webtoon including those in daily pass?
With the inclusion of daily pass originals, we only rank rating and subscribers count, not the number of views and likes.
```sql
WITH rank_table AS
(SELECT title, title_id, genre,
		DENSE_RANK() OVER (ORDER BY rating DESC) AS rating_rank,
		DENSE_RANK() OVER (ORDER BY subscribers DESC) AS subs_rank
	FROM [dbo].[webtoon_fixed]
	WHERE daily_pass = 0) 
SELECT TOP 10 title_id, title, genre, rating_rank, subs_rank,
	(rating_rank + subs_rank) AS sum_rank
FROM rank_table
ORDER BY sum_rank
```

#### Output:
| title_id | title                 | genre      | rating_rank | subs_rank | sum_rank |
|----------|-----------------------|------------|-------------|-----------|----------|
| 95       | Tower of God          | FANTASY    | 7           | 6         | 13       |
| 2135     | The Remarried Empress | FANTASY    | 7           | 10        | 17       |
| 1022     | LUMINE                | FANTASY    | 11          | 7         | 18       |
| 1817     | Down To Earth         | ROMANCE    | 12          | 8         | 20       |
| 2154     | Omniscient Reader     | ACTION     | 4           | 16        | 20       |
| 679      | unOrdinary            | SUPER_HERO | 18          | 3         | 21       |
| 986      | I Love Yoo            | DRAMA      | 17          | 5         | 22       |
| 1499     | Castle Swimmer        | FANTASY    | 9           | 14        | 23       |
| 1468     | SubZero               | ROMANCE    | 15          | 9         | 24       |
| 1320     | Lore Olympus          | ROMANCE    | 23          | 2         | 25       |

#### Remarks:
(to be added)

***
### 4. For each genre, what is the top originals excluding those in daily pass?
```sql
WITH temp AS 
	(SELECT title_id, [title], [genre],
	DENSE_RANK() OVER (PARTITION BY genre ORDER BY rating_rank + subs_rank + views_rank + likes_rank) AS genre_rank
	FROM rank_table)
SELECT genre, title_id, title
FROM temp
WHERE genre_rank = 1;
```

#### Output: 
| genre         | title_id | title                    |
|---------------|----------|--------------------------|
| ACTION        | 2154     | Omniscient Reader        |
| COMEDY        | 1438     | Mage & Demon Queen       |
| DRAMA         | 986      | I Love Yoo               |
| FANTASY       | 2135     | The Remarried Empress    |
| HEARTWARMING  | 2620     | When the Day Comes       |
| HISTORICAL    | 3671     | Return of the Mad Demon  |
| HORROR        | 2578     | Everything is Fine       |
| MYSTERY       | 1621     | Purple Hyacinth          |
| ROMANCE       | 1320     | Lore Olympus             |
| SF            | 1229     | Eggnoid                  |
| SLICE_OF_LIFE | 958      | My Giant Nerd Boyfriend  |
| SPORTS        | 372      | Wind Breaker             |
| SUPER_HERO    | 679      | unOrdinary               |
| SUPERNATURAL  | 1697     | I'm the Grim Reaper      |
| THRILLER      | 2759     | Homesick                 |
| TIPTOON       | 1963     | Staying Healthy Together |

***
### 5. What are the weekday performances of Webtoon so far (including every original regardless of their status)?
```sql
SELECT value AS weekday,
	count(*) AS originals_published,
	ROUND(AVG(rating),2) AS avg_rating,
	AVG(subscribers) AS avg_subscribers,
	AVG(views/length) AS avg_views,
	AVG(likes/length) AS avg_likes
FROM [dbo].[webtoon_fixed]
CROSS APPLY string_split(weekdays, ',')
GROUP BY value
```

### Output: 
| weekday   | originals_published | avg_rating | avg_subscribers | avg_views | avg_likes |
|-----------|---------------------|------------|-----------------|-----------|-----------|
| WEDNESDAY | 183                 | 9,35       | 370980          | 1092237   | 91549     |
| SATURDAY  | 168                 | 9,39       | 377894          | 788955    | 67856     |
| MONDAY    | 165                 | 9,37       | 411622          | 1064254   | 85045     |
| SUNDAY    | 171                 | 9,29       | 430437          | 1054164   | 87265     |
| FRIDAY    | 175                 | 9,4        | 367587          | 828291    | 77914     |
| THURSDAY  | 170                 | 9,32       | 387146          | 1323782   | 105351    |
| TUESDAY   | 171                 | 9,31       | 386598          | 892506    | 73418     |

### Remarks:
SATURDAY seems to be the least exciting day for Webtoon as the average subscribers, views and likes are quite modest as compared to other weekdays. THURSDAY attracts the highest average views and likes, with almost 300,000 
more views than WEDNESDAY in second place.

Overall, Webtoon has been keeping a similar number of originals published for each weekday. Interestingly, the average rating for each weekday is almost the same.

***
### 6. What are the weekday performances of ONGOING Webtoon?
```sql
SELECT value AS weekday_ongoing,
	COUNT(*) AS originals_count,
	ROUND(AVG(rating), 2) AS avg_rating,
	AVG(subscribers) AS avg_subscribers,
	AVG(views/length) AS avg_views,
	AVG(likes/length) AS avg_likes
FROM [dbo].[webtoon_fixed]
CROSS APPLY string_split(weekdays, ',')
WHERE status = 'ONGOING'
GROUP BY value
```

#### Output:
| weekday_ongoing | originals_count | avg_rating | avg_subscribers | avg_views | avg_likes |
|-----------------|-----------------|------------|-----------------|-----------|-----------|
| WEDNESDAY       | 55              | 9,45       | 472575          | 318612    | 28072     |
| SATURDAY        | 55              | 9,52       | 476583          | 370495    | 35261     |
| MONDAY          | 55              | 9,52       | 382656          | 274657    | 27674     |
| SUNDAY          | 64              | 9,23       | 578749          | 442743    | 38804     |
| FRIDAY          | 59              | 9,43       | 515396          | 395562    | 36353     |
| THURSDAY        | 54              | 9,51       | 563467          | 835493    | 65582     |
| TUESDAY         | 58              | 9,39       | 509624          | 376298    | 31621     |

#### Remarks:
SUNDAY has the highest originals that are currently ongoing, however the average rating is the worst among all weekdays. MONDAY is the least exciting day on Webtoon with the least amount of average subscribers, views and likes. 

THURSDAY is again a very busy day on Webtoon with the highest average views that almost double that of the second place.

***
### 7. Which authors attract the most views?
Here, I did not calculate the average of subscribers, views and likes but rather the total of them. 
```sql
SELECT TOP 10
	value as author_name,
	COUNT(*) AS originals_published,
	ROUND(AVG(rating),2) AS avg_rating,
	SUM(subscribers) AS total_subscribers,
	SUM(views) AS total_views,
	SUM(likes) AS total_likes
FROM [dbo].[webtoon_fixed]
CROSS APPLY string_split(authors, ',')
GROUP BY value
order by total_views desc
```

#### Output: 
| author_name                | originals_published | avg_rating | total_subscribers | total_views | total_likes |
|----------------------------|---------------------|------------|-------------------|-------------|-------------|
| Rachel Smythe              | 1                   | 9,7        | 5970676           | 1121117616  | 55367904    |
| uru-chan                   | 1                   | 9,75       | 5624872           | 1092922415  | 52920191    |
| SIU                        | 1                   | 9,86       | 3216820           | 1018532519  | 51902659    |
| Yaongyi                    | 1                   | 9,51       | 7128661           | 914658370   | 48705268    |
| Yongje Park                | 1                   | 9,69       | 2683666           | 743008133   | 28814309    |
| fishball                   | 1                   | 9,73       | 2253761           | 725884348   | 59844916    |
| Taejun Pak                 | 3                   | 9,82       | 3470575           | 652623377   | 47141710    |
| Merryweather               | 4                   | 9,56       | 3758100           | 650113146   | 55190458    |
| Leeanne M. Krecic (Mongie) | 1                   | 9,58       | 4768519           | 649163466   | 39990605    |
| Shen                       | 2                   | 9,73       | 2007073           | 641467028   | 53961746    |

#### Remarks:
Top 4 authors are very close to each other. Author Yaongyi seems to stand out with a significantly high amount of subscribers that is even higher than the top 1 author Rachel Smythe, while their average rating is the lowest among top 10 authors. Even so, their rating at 9,51 is still very high.
