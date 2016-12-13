# clickhouse-test

### Clone

Make sure that you have Git LFS installed.

```
git lfs clone https://github.com/bookin/clickhouse-test.git
```

### Schema
```sql
CREATE TABLE IF NOT EXISTS user_visit (
	event_date Date DEFAULT toDate(event_time),
	event_time DateTime,
	user_id Int32,
	site_id Int32
) 
ENGINE = MergeTree(event_date, (event_time, user_id, site_id), 8192);

CREATE TABLE IF NOT EXISTS user_visit_ip_str (
	event_date Date DEFAULT toDate(event_time),
	event_time DateTime,
	user_ip String,
	site_id Int32
) 
ENGINE = MergeTree(event_date, (event_time, user_ip, site_id), 8192);


CREATE TABLE IF NOT EXISTS user_visit_ip_num (
	event_date Date DEFAULT toDate(event_time),
	event_time DateTime,
	user_ip UInt32,
	site_id Int32
) 
ENGINE = MergeTree(event_date, (event_time, user_ip, site_id), 8192);
```

### Load dumps
```bash
cat dump_user_visit.tabs | clickhouse-client -q "INSERT INTO user_visit FORMAT TabSeparated"
cat dump_user_visit_ip_str.tabs | clickhouse-client -q "INSERT INTO user_visit_ip_str FORMAT TabSeparated"
cat dump_user_visit_ip_num.tabs | clickhouse-client -q "INSERT INTO user_visit_ip_num FORMAT TabSeparated"
```

### Create dumps
```bash
clickhouse-client -q "SELECT * FROM user_visit FORMAT TabSeparated" > dump_user_visit.tabs
clickhouse-client -q "SELECT * FROM user_visit_ip_str FORMAT TabSeparated" > dump_user_visit_ip_str.tabs
clickhouse-client -q "SELECT * FROM user_visit_ip_num FORMAT TabSeparated" > dump_user_visit_ip_num.tabs
```

#### SERVER INFO
**CPU:**1 vCore
**RAM:**768 MB
**SWAP:**1 GB
**Storage:**15 GB SSD
**OS:**Ubuntu 16.04 (Xenial Xerus)

## Results

#### Run time

Query| int user id | string user ip | num user ip
--- | --- | --- | ---
**select(distinct())** between all | 1.214 | 3.840 | 4.351
**select(distinct())** between AND where | 0.329 | 0.804 | 0.583
**select(distinct())** one date | 0.039 | 0.050 | 0.031
| | |
**uniq()** between all | 0.100 | 0.582 | 0.118
**uniq()** between AND where | 0.126 | 0.329 | 0.105
**uniq()** one date | 0.013 | 0.024 | 0.015
| | |
**uniqCombined()** between all | 0.186 | 0.621 | 0.202
**uniqCombined()** between AND where | 0.104 | 0.296 | 0.102
**uniqCombined()** one date | 0.011 | 0.020 | 0.012
| | |
**group by** date and time | 0.019 | 0.037 | 0.027
**group by** days | 0.062 | 0.060 | 0.062


#### user_visit
```sql
SELECT count() FROM (SELECT distinct(user_id) FROM user_visit WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12'));
-- 997533 rows - Elapsed: 1.214 sec. Processed 6.00 million rows, 36.00 MB (4.94 million rows/s., 29.65 MB/s.)

SELECT count() FROM (SELECT distinct(user_id) FROM user_visit WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12') AND site_id=4);
-- 451824 rows - Elapsed: 0.329 sec. Processed 6.00 million rows, 60.00 MB (18.22 million rows/s., 182.15 MB/s.)

SELECT count() FROM (SELECT distinct(user_id) FROM user_visit WHERE event_date = toDate('2016-11-10'));
-- 93427 rows - Elapsed: 0.039 sec. Processed 2.95 million rows, 6.36 MB (75.33 million rows/s., 162.37 MB/s.)




SELECT uniq(user_id) FROM user_visit WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12');
-- 999616 rows - Elapsed: 0.100 sec. Processed 6.00 million rows, 36.00 MB (59.79 million rows/s., 358.73 MB/s.)

SELECT uniq(user_id) FROM user_visit WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12') AND site_id=4;
-- 453801 rows - Elapsed: 0.126 sec. Processed 6.00 million rows, 60.00 MB (47.76 million rows/s., 477.61 MB/s.)

SELECT uniq(user_id) FROM user_visit WHERE event_date = toDate('2016-11-10');
-- 93265 rows - Elapsed: 0.013 sec. Processed 2.95 million rows, 6.36 MB (219.90 million rows/s., 473.98 MB/s.)




SELECT uniqCombined(user_id) FROM user_visit WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12');
-- 998595 rows - Elapsed: 0.186 sec. Processed 6.00 million rows, 36.00 MB (32.24 million rows/s., 193.45 MB/s.)

SELECT uniqCombined(user_id) FROM user_visit WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12') AND site_id=4;
-- 451620 rows - Elapsed: 0.104 sec. Processed 6.00 million rows, 60.00 MB (57.51 million rows/s., 575.13 MB/s.)

SELECT uniqCombined(user_id) FROM user_visit WHERE event_date = toDate('2016-11-10');
-- 93544 rows - Elapsed: 0.011 sec. Processed 2.95 million rows, 6.36 MB (279.21 million rows/s., 601.81 MB/s.)


SELECT toHour(event_time) as h, toMinute(event_time) as m, count() FROM user_visit WHERE event_date = toDate('2016-10-13') group by h,m ORDER BY h,m;
-- Elapsed: 0.019 sec. Processed 2.95 million rows, 6.36 MB (153.72 million rows/s., 331.34 MB/s.)

SELECT toDayOfMonth(event_time) as d, count() FROM user_visit WHERE event_date between toDate('2016-11-01') AND toDate('2016-11-30') group by d ORDER BY d;
-- 30 rows - Elapsed: 0.062 sec. Processed 2.95 million rows, 17.71 MB (47.47 million rows/s., 284.83 MB/s.)
```

#### user_visit_ip_str
```sql
SELECT count() FROM (SELECT distinct(IPv4StringToNum(user_ip)) FROM user_visit_ip_str WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12'));
-- 5995771 rows - Elapsed: 3.840 sec. Processed 6.00 million rows, 145.65 MB (1.56 million rows/s., 37.93 MB/s.)

SELECT count() FROM (SELECT distinct(IPv4StringToNum(user_ip)) FROM user_visit_ip_str WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12') AND site_id=4);
-- 600554 rows - Elapsed: 0.804 sec. Processed 6.00 million rows, 169.65 MB (7.47 million rows/s., 211.09 MB/s.)

SELECT count() FROM (SELECT distinct(IPv4StringToNum(user_ip)) FROM user_visit_ip_str WHERE event_date = toDate('2016-11-10'));
-- 98380 rows - Elapsed: 0.050 sec. Processed 2.95 million rows, 8.83 MB (58.93 million rows/s., 176.37 MB/s.)



SELECT uniq(IPv4StringToNum(user_ip)) FROM user_visit_ip_str WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12');
-- 6021353 rows - Elapsed: 0.582 sec. Processed 6.00 million rows, 145.65 MB (10.32 million rows/s., 250.44 MB/s.)

SELECT uniq(IPv4StringToNum(user_ip)) FROM user_visit_ip_str WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12') AND site_id=4;
-- 597905 rows - Elapsed: 0.329 sec. Processed 6.00 million rows, 169.65 MB (18.25 million rows/s., 515.87 MB/s.)

SELECT uniq(IPv4StringToNum(user_ip)) FROM user_visit_ip_str WHERE event_date = toDate('2016-11-10');
-- 98513 rows - Elapsed: 0.024 sec. Processed 2.95 million rows, 8.83 MB (123.08 million rows/s., 368.33 MB/s.)



SELECT uniqCombined(IPv4StringToNum(user_ip)) FROM user_visit_ip_str WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12');
-- 6003838 rows - Elapsed: 0.621 sec. Processed 6.00 million rows, 145.65 MB (9.66 million rows/s., 234.61 MB/s.)

SELECT uniqCombined(IPv4StringToNum(user_ip)) FROM user_visit_ip_str WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12') AND site_id=4;
-- 601209 rows - Elapsed: 0.296 sec. Processed 6.00 million rows, 169.65 MB (20.23 million rows/s., 572.09 MB/s.)

SELECT uniqCombined(IPv4StringToNum(user_ip)) FROM user_visit_ip_str WHERE event_date = toDate('2016-11-10');
-- 98196 rows - Elapsed: 0.020 sec. Processed 2.95 million rows, 8.83 MB (147.86 million rows/s., 442.52 MB/s.)


SELECT toHour(event_time) as h, toMinute(event_time) as m, count() FROM user_visit_ip_str WHERE event_date = toDate('2016-10-13') group by h,m ORDER BY h,m;
-- Elapsed: 0.037 sec. Processed 1.87 million rows, 4.23 MB (50.73 million rows/s., 114.81 MB/s.)

SELECT toDayOfMonth(event_time) as d, count() FROM user_visit_ip_str WHERE event_date between toDate('2016-11-01') AND toDate('2016-11-30') group by d ORDER BY d;
-- 30 rows - Elapsed: 0.060 sec. Processed 2.95 million rows, 17.71 MB (48.97 million rows/s., 293.85 MB/s.)
```

#### user_visit_ip_num
```sql
SELECT count() FROM (SELECT distinct(user_ip) FROM user_visit_ip_num WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12'));
-- 5995848 rows - Elapsed: 4.351 sec. Processed 6.00 million rows, 36.00 MB (1.38 million rows/s., 8.27 MB/s.)

SELECT count() FROM (SELECT distinct(user_ip) FROM user_visit_ip_num WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12') AND site_id=4);
-- 600601 rows - Elapsed: 0.583 sec. Processed 6.00 million rows, 60.00 MB (10.29 million rows/s., 102.88 MB/s.)

SELECT count() FROM (SELECT distinct(user_ip) FROM user_visit_ip_num WHERE event_date = toDate('2016-11-10'));
-- 98917 rows - Elapsed: 0.031 sec. Processed 2.95 million rows, 6.43 MB (94.43 million rows/s., 205.90 MB/s.)



SELECT uniq(user_ip) FROM user_visit_ip_num WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12');
-- 6033965 rows - Elapsed: 0.118 sec. Processed 6.00 million rows, 36.00 MB (50.86 million rows/s., 305.14 MB/s.)

SELECT uniq(user_ip) FROM user_visit_ip_num WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12') AND site_id=4;
-- 600037 rows - Elapsed: 0.105 sec. Processed 6.00 million rows, 60.00 MB (57.12 million rows/s., 571.21 MB/s.)

SELECT uniq(user_ip) FROM user_visit_ip_num WHERE event_date = toDate('2016-11-10');
-- 99260 rows Elapsed: 0.015 sec. Processed 2.95 million rows, 6.43 MB (190.89 million rows/s., 416.22 MB/s.)



SELECT uniqCombined(user_ip) FROM user_visit_ip_num WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12');
-- 5995942 rows - Elapsed: 0.202 sec. Processed 6.00 million rows, 36.00 MB (29.70 million rows/s., 178.23 MB/s.)

SELECT uniqCombined(user_ip) FROM user_visit_ip_num WHERE event_date between toDate('2016-10-13') AND toDate('2016-12-12') AND site_id=4;
-- 600463 rows - Elapsed: 0.102 sec. Processed 6.00 million rows, 60.00 MB (58.66 million rows/s., 586.60 MB/s.)

SELECT uniqCombined(user_ip) FROM user_visit_ip_num WHERE event_date = toDate('2016-11-10');
-- 98728 rows - Elapsed: 0.012 sec. Processed 2.95 million rows, 6.43 MB (252.62 million rows/s., 550.81 MB/s.)


SELECT toHour(event_time) as h, toMinute(event_time) as m, count() FROM user_visit_ip_num WHERE event_date = toDate('2016-10-13') group by h,m ORDER BY h,m;
-- Elapsed: 0.027 sec. Processed 1.87 million rows, 4.20 MB (68.27 million rows/s., 153.31 MB/s.)

SELECT toDayOfMonth(event_time) as d, count() FROM user_visit_ip_num WHERE event_date between toDate('2016-11-01') AND toDate('2016-11-30') group by d ORDER BY d;
-- 30 rows - Elapsed: 0.062 sec. Processed 2.95 million rows, 17.70 MB (47.72 million rows/s., 286.34 MB/s.)
```
