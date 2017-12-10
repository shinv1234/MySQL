# SQL_Tips

### Make ID
```SQL
SELECT @ID:=@ID+1 AS ID, a.*
FROM nba_shot_logs_sorted AS a, 
     (SELECT @ID := 0) AS id1;
```

### Numbering of groups using self join
```SQL
CREATE VIEW IMSI AS
SELECT a.ID, count(*) as shot_number2, a.GAME_ID, a.PERIOD, a.GAME_CLOCK
FROM nba_shot_logs a
JOIN nba_shot_logs b 
ON a.GAME_ID = b.GAME_ID 
AND a.PERIOD = b.PERIOD
AND a.GAME_CLOCK >= b.GAME_CLOCK
GROUP BY a.GAME_ID, a.PERIOD, a.GAME_CLOCK
ORDER BY a.ID;
```