# NBA Shot Logs

- (현재는) 데이터가 매우 일부만 존재...

---
### Best 3-Point Player

```sql
SELECT player_id
      ,player_name
		,SUM(CASE WHEN SHOT_RESULT = 'made' THEN 1 ELSE 0 END)/
		 COUNT(SHOT_RESULT) AS SHOT
FROM nba_shot_logs
WHERE PTS_TYPE = 3
GROUP BY player_id
ORDER BY SHOT DESC;
```
| player_id | player_name | SHOT | 
| -: | - | -: | 
| 204037 | travis wear | 1.0000 | 
| 101138 | brandon bass | 1.0000 | 
| 2446 | rasual butler | 1.0000 | 
| 2744 | al jefferson | 1.0000 | 
| 202326 | demarcus cousins | 1.0000 | 
| 2210 | richard jefferson | 1.0000 | 
| 202087 | alonzo gee | 1.0000 | 
| 201587 | nicolas batum | 1.0000 | 

---
### 최다 득점(골 수 기준)
```sql
SELECT player_id
      ,player_name
		,SUM(CASE WHEN SHOT_RESULT = 'made' THEN 1 ELSE 0 END) AS GOAL
FROM nba_shot_logs
GROUP BY player_id
ORDER BY GOAL DESC;
```
| player_id | player_name | GOAL | 
| -: | - | -: | 
| 101108 | chris paul | 38 | 
| 2544 | lebron james | 36 | 
| 201939 | stephen curry | 28 | 
| 2744 | al jefferson | 24 | 
| 200755 | jj redick | 23 | 
| 200752 | rudy gay | 23 | 
| 201945 | gerald henderson | 22 | 
| 203506 | victor oladipo | 22 | 
| 202391 | jeremy lin | 21 | 
| 200746 | lamarcus aldridge | 21 | 
| 201936 | tyreke evans | 21 | 
| 201568 | danilo gallinai | 20 | 
---

### 최고 스코어러

```sql
SELECT player_id
      ,player_name
		,SUM(PTS) AS SCORE
FROM nba_shot_logs
GROUP BY player_id
ORDER BY SCORE DESC;
```
| player_id | player_name | SCORE | 
| -: | - | -: | 
| 101108 | chris paul | 80 | 
| 2544 | lebron james | 79 | 
| 201939 | stephen curry | 73 | 
| 200755 | jj redick | 56 | 
| 200752 | rudy gay | 50 | 
| 201568 | danilo gallinai | 49 | 
| 2744 | al jefferson | 49 | 
| 201945 | gerald henderson | 49 | 
| 203506 | victor oladipo | 49 | 
| 202391 | jeremy lin | 47 | 
| 201567 | kevin love | 45 | 
| 201936 | tyreke evans | 44 | 
| 200746 | lamarcus aldridge | 42 | 
| 202738 | isaiah thomas | 41 | 
| 202691 | klay thompson | 40 | 


---

### 홈 & 어웨이 별 득점현황
```sql
SELECT LOCATION
      ,COUNT(PTS_TYPE)
      ,SUM(CASE WHEN PTS_TYPE=2 THEN 1 ELSE 0 END)/COUNT(PTS_TYPE) AS Try2_rate
      ,SUM(CASE WHEN PTS_TYPE=3 THEN 1 ELSE 0 END)/COUNT(PTS_TYPE) AS Try3_rate
      ,SUM(CASE WHEN PTS!=0 THEN 1 ELSE 0 END)/COUNT(PTS_TYPE) AS Goal_rate
      ,SUM(CASE WHEN PTS=2 THEN 1 ELSE 0 END)/
		SUM(CASE WHEN PTS_TYPE=2 THEN 1 ELSE 0 END) AS Goal2_rate
      ,SUM(CASE WHEN PTS=3 THEN 1 ELSE 0 END)/
		SUM(CASE WHEN PTS_TYPE=3 THEN 1 ELSE 0 END) AS Goal3_rate
FROM nba_shot_logs
GROUP BY LOCATION DESC;
```
| LOCATION | COUNT(PTS_TYPE) | Try2_rate | Try3_rate | Goal_rate | Goal2_rate | Goal3_rate | 
| - | -: | -: | -: | -: | -: | -: | 
| H | 1928 | 0.7350 | 0.2650 | 0.4378 | 0.4757 | 0.3327 | 
| A | 2098 | 0.7369 | 0.2631 | 0.4461 | 0.4871 | 0.3315 | 


----
### Best Defencer
```sql
SELECT player_id, player_name
      ,COUNT(player_id) AS The_Number_of_Def
      ,AVG(CLOSE_DEF_DIST) AS DEF_DIST_Avg
      ,MAX(CLOSE_DEF_DIST) AS DEF_DIST_MAX
      ,SUM(CASE WHEN PTS=0 THEN 1 ELSE 0 END)/COUNT(PTS) AS Defence_Success_Rate
FROM nba_shot_logs
GROUP BY CLOSEST_DEFENDER
HAVING COUNT(player_id) > 8
ORDER BY Defence_Success_Rate DESC, DEF_DIST_Avg
```
| player_id | player_name | The_Number_of_Def | DEF_DIST_Avg | DEF_DIST_MAX | Defence_Success_Rate | 
| -: | - | -: | -: | -: | -: | 
| 101145 | mnta ellis | 11 | 3.24545 | 6.9 | 0.8182 | 
| 202683 | enes kanter | 11 | 3.73636 | 8.1 | 0.8182 | 
| 1889 | andre miller | 10 | 3.46000 | 10.1 | 0.8000 | 
| 203084 | harrison barnes | 20 | 3.93500 | 11.2 | 0.8000 | 
| 202390 | gary neal | 28 | 4.35714 | 16.5 | 0.7857 | 
| 201574 | jason thompson | 9 | 2.58889 | 6.5 | 0.7778 | 
| 202703 | nikola mirotic | 9 | 2.86667 | 5.3 | 0.7778 | 
| 202390 | gary neal | 9 | 4.74444 | 10.4 | 0.7778 | 
| 203148 | brian roberts | 19 | 4.53158 | 12.5 | 0.7368 | 
| 201945 | gerald henderson | 15 | 3.74667 | 12.2 | 0.7333 | 
| 203082 | terrence ross | 15 | 4.32000 | 10.6 | 0.7333 | 
| 203148 | brian roberts | 11 | 3.12727 | 7.9 | 0.7273 | 
| 202330 | gordon hayward | 22 | 4.49545 | 11.6 | 0.7273 | 
| 202683 | enes kanter | 11 | 4.50909 | 8.3 | 0.7273 | 
| 2550 | kirk hinrich | 22 | 4.60455 | 9.0 | 0.7273 | 
| 202681 | kyrie irving | 11 | 5.05455 | 10.0 | 0.7273 | 
| 203143 | pablo prigioni | 11 | 5.16364 | 7.4 | 0.7273 | 
| 202330 | gordon hayward | 29 | 3.14138 | 11.0 | 0.7241 | 
| 202340 | avery bradley | 21 | 3.49524 | 8.4 | 0.7143 | 
| 203463 | ben mclemore | 21 | 4.31905 | 10.4 | 0.7143 | 
| 202390 | gary neal | 14 | 5.06429 | 18.1 | 0.7143 | 
| 2430 | carlos boozer | 24 | 3.38750 | 8.9 | 0.7083 | 


----

### Best Dribbler
```sql
SELECT player_id, player_name
      ,AVG(DRIBBLES) AS DRIBBLES_AVG
      ,MAX(DRIBBLES) AS DRIBBLES_MAX
FROM nba_shot_logs
GROUP BY player_id
ORDER BY DRIBBLES_AVG DESC, DRIBBLES_MAX DESC;
```
| player_id | player_name | DRIBBLES_AVG | DRIBBLES_MAX | 
| -: | - | -: | -: | 
| 201952 | jeff teague | 8.3333 | 22 | 
| 101108 | chris paul | 7.8824 | 25 | 
| 202709 | cory joseph | 7.3333 | 13 | 
| 203143 | pablo prigioni | 7.2500 | 29 | 
| 201566 | russell westbrook | 6.6667 | 22 | 
| 2544 | lebron james | 6.4054 | 24 | 
| 200826 | jose juan barea | 6.4000 | 15 | 
| 202704 | reggie jackson | 6.1250 | 24 | 


----
### Shot_Table
```sql
SELECT PTS_TYPE AS Shot_Type
      ,AVG(SHOT_DIST) AS SHOT_DIST_AVG
      ,SUM(CASE WHEN PTS!=0 THEN 1 ELSE 0 END)/COUNT(PTS) AS Goal_Rate
FROM nba_shot_logs
GROUP BY Shot_Type
ORDER BY SHOT_DIST;
```
| Shot_Type | SHOT_DIST_AVG | Goal_Rate | 
| -: | -: | -: | 
| 2 | 9.52717 | 0.4816 | 
| 3 | 24.42023 | 0.3321 | 


----
### Home & Away Goal Board
```sql
SELECT LOCATION AS Home_Away
      ,PTS_TYPE AS Shot_Type
      ,AVG(SHOT_DIST) AS SHOT_DIST_AVG
      ,SUM(CASE WHEN PTS!=0 THEN 1 ELSE 0 END)/COUNT(PTS) AS Goal_Rate
FROM nba_shot_logs
GROUP BY Home_Away, Shot_Type
ORDER BY Home_Away, Shot_Type;
```
| Home_Away | Shot_Type | SHOT_DIST_AVG | Goal_Rate | 
| - | -: | -: | -: | 
| A | 2 | 9.49858 | 0.4871 | 
| A | 3 | 24.29873 | 0.3315 | 
| H | 2 | 9.55836 | 0.4757 | 
| H | 3 | 24.55147 | 0.3327 | 




