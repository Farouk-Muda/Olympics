# Olympics
A proof of my SQL skills to analyse the datasets.
![](https://github.com/Farouk-Muda/Olympics/blob/main/olympics%20png.jpg)

## Background
This is a real datasets detailing the 120 years of Olympics History dataset from Kaggle from the user [rgriffin](https://www.kaggle.com/datasets/heesoo37/120-years-of-olympic-history-athletes-and-results) that will help us to appreciate the insights we gleaned from the data. 
This dataset contains two files: 
- athlete_events.csv, which contains 271116 rows and 15 columns. Each row corresponds to an individual athlete competing in an individual Olympic event (athlete-events) and 
- noc_regions.csv also contains 230 rows and 3 columns (National Olympic Committee, NOC ;country name and notes).

 By leveraging PostgreSQL and advanced SQL techniques like window functions, joins, and aggregations, we explore everything from participation trends, medal distributions, athlete performances, and country rankings across 51 Olympic Games from 1896 to 2016. Executed SQL queries to answer 23 distinct questions including:
- 1.How many olympics games have been held?
- 2.List down all Olympics games held so far.
- 3.Mention the total no of nations who participated in each olympics game?
- 4.Which year saw the highest and lowest no of countries participating in olympics?
- 5.Which nation has participated in all of the olympic games?
- 6.Identify the sport which was played in all summer olympics.
- 7.Which Sports were just played only once in the olympics?
- 8.Fetch the total no of sports played in each olympic games.
- 9.Fetch details of the oldest athletes to win a gold medal.
- 10.Find the Ratio of male and female athletes participated in all olympic games.
- 11.Fetch the top 5 athletes who have won the most gold medals.
- 12.Fetch the top 5 athletes who have won the most medals (gold/silver/bronze).
- 13.Fetch the top 5 most successful countries in olympics. Success is defined by no of medals won.
- 14.List down total gold, silver and broze medals won by each country.
- 15.List down total gold, silver and broze medals won by each country corresponding to each olympic games.
- 16.Identify which country won the most gold, most silver and most bronze medals in each olympic games.
- 17.Identify which country won the most gold, most silver, most bronze medals and the most medals in each olympic games.
- 18.Which countries have never won gold medal but have won silver/bronze medals?
- 19.In which Sport/event, India has won highest medals.
- 20.Break down all olympic games where india won medal for Hockey and how many medals in each olympic games.

### Some interesting questions answered

**Q5 - Which nation has participated in all of the olympic games?**

```sql
with total as
    (select count(distinct games) as total_games from athlete_events),
country as
    (select t2.region, count(distinct t1.games) as no_of_countries
     from athlete_events t1 
    join noc_regions t2 on t1.noc=t2.noc
     group by t2.region)
select region, no_of_countries 
from country c 
join total t on c.no_of_countries = t.total_games;
```


**Q17 -  Identify which country won the most gold, most silver, most bronze medals and the most medals in each olympic games.?**

```sql
with medals as
	 (select substring(game_country, 1, position(' - ' in game_country) -1) as games,
	  substring(game_country, position(' - ' in game_country) +3 ) as country,
	  coalesce(gold, 0) as gold,
	  coalesce(bronze, 0) as bronze,
	  coalesce(silver, 0) as silver
	from crosstab 
		('select concat(t1.games, '' - '', t2.region) as game_country,  
			 medal,
			 count(medal) as no_of_medal
		from athlete_events t1 
		join noc_regions t2 on t1.noc=t2.noc
		where medal <> ''NA''
		group by  t1.games, t2.region, medal order by t1.games, t2.region, medal',
		'values(''Gold''),(''Bronze''), (''Silver'')')
		as result (game_country text, Gold bigint, Silver bigint, Bronze bigint)),
finals as	
	(select t1.games, t2.region, count(medal) as total_medals 
	 from athlete_events t1 
	 join noc_regions t2 on t1.noc=t2.noc
	 where medal <> 'NA'
	 group by  t1.games, t2.region order by t1.games, t2.region)
select distinct m.games, 
   concat(first_value(m.country) over (partition by m.games order by m.gold desc),
		  ' - ', 
		  first_value(m.gold) over (partition by m.games order by m.gold desc)) as max_gold,
   concat(first_value(m.country) over (partition by m.games order by m.bronze desc),
		  ' - ', 
		  first_value(m.bronze) over (partition by m.games order by m.bronze desc)) as max_bronze,
   concat(first_value(m.country) over (partition by m.games order by m.silver desc),
		  ' - ', 
		  first_value(m.silver) over (partition by m.games order by m.silver desc)) as max_silver,
   concat(first_value(m.country) over (partition by m.games order by f.total_medals desc),
		  ' - ', 
		  first_value(f.total_medals) over (partition by m.games order by f.total_medals desc)) as max_medals
from medals m 
join finals f on m.games= f.games and m.country=f.region
order by games;
```

### Relevance
- Sports management are able to track athlete performance over different Olympic games to identify trends, areas of improvements, and the effectiveness of training programs.
- Effective in talent identification.
- Sports scientist used this data to conduct research on physiology and performance metrics. 

## Insights
- Analyse the number of medals won by each country over different Olympic Games.
- Compare individual athlete performances over multiple events to assess consistency and improvement.
- insights into the participation rates to improving or declining sports.
