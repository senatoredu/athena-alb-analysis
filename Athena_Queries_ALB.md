# AWS' Application Load Balancer is a Layer 7 Based Load Balancer.
When an Access Log is enabled entries are populated in the log that contain information about each access made to the ALB (HTTP, HTTPS, Websocket etc).
For troubleshooting and analysis purposes Amazon Athena can be used to analyze the ALB's access logs. 
Athena is AWS' Big Data Service that gives developers speed and cost benefits when analyzing huge chunks of data. 
The below are some SQL queries i wrote/use when analyzing logs: 

Table name: alb_log

### View all access log entries chronologically.
```sql
SELECT *
FROM alb_log
ORDER BY time ASC;
```

### See how many requests or connections the ALB processed.
```sql
select count() from alb_log;
```

### The timestamps from when the first and last requests or connections were processed by  ALB.
```sql
 SELECT  (
    SELECT min(time)
    FROM   alb_log
    ) AS request_first_entrytime,
    (
    SELECT max(time)
    FROM  alb_log
    ) AS request_last_entrytime;
```

### List the type of requests and connections used to access ALB and how many times each type accessed the ALB.
```sql
SELECT DISTINCT type, count(*) as count from alb_log
GROUP by type
ORDER by count(*);
```

### List all backends or targets and how many times each was selected by the ALB's load balancing algorithm to dispatch requests or connections.
```sql
SELECT DISTINCT target_ip, count(*) as count from alb_log
GROUP by target_ip
ORDER by count(*) DESC;
```

### List minimum, maximum, and latency times.
```sql
SELECT  (
   SELECT min(target_processing_time)
   FROM   alb_log
   ) AS minimum_latency,
   (
   SELECT avg(target_processing_time)
   FROM   alb_log
   ) AS average_latency,
   (
   SELECT max(target_processing_time)
   FROM  alb_log
   ) AS maximum_latency;
```

### List the count of all client IP addresses, as a percentage of all requests or connections processed by the ALB.
```sql
SELECT client_ip, (Count(client_ip)* 100.0 / (Select Count(*) From alb_log)) 
as client_traffic_percentage
FROM alb_log
GROUP by client_ip
ORDER By count() DESC;
```

### List the average amount of data (in bytes) that is passing through the ALB in request/response pairs. 
```sql
SELECT (avg(sent_bytes) + avg(received_bytes)) as prewarm_bytes from alb_log;
```

### Arrange clients in descending order, by the amount of data that they sent in their requests to the ALB (in megabytes).
```sql
SELECT client_ip, sum(received_bytes/1000000.0) as client_datareceived_megabytes
FROM alb_log
GROUP by client_ip
ORDER by client_datareceived_megabytes DESC;
```

### List the access log entries between example date: 2018-08-08-00:00:00 and 2018-08-08-02:00:00 where the target processing time or latency was more than 5 seconds 
```sql
SELECT * from alb_log 
WHERE (parse_datetime(time,'yyyy-MM-dd''T''HH:mm:ss.SSSSSS''Z') 
     BETWEEN parse_datetime('2018-08-08-00:00:00','yyyy-MM-dd-HH:mm:ss') 
     AND parse_datetime('2018-08-08-02:00:00','yyyy-MM-dd-HH:mm:ss'))
AND (target_processing_time >= 5.0)
```


