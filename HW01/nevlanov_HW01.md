
postgres@HNBA110232:/etc/postgresql/17/main$ pgbench-c8 -P 6 -T 60 -U postgrespostgres
pgbench-c8: command not found
postgres@HNBA110232:/etc/postgresql/17/main$ pgbench-c8 -P 6 -T 60 -U postgres postgres
pgbench-c8: command not found
postgres@HNBA110232:/etc/postgresql/17/main$ pgbench-c8 -P 6 -T 60 -U test
pgbench-c8: command not found
postgres@HNBA110232:/etc/postgresql/17/main$ pgbench -c 50 -j 2 -P 10 -T 60 test
pgbench (17.5 (Ubuntu 17.5-1.pgdg24.04+1))
starting vacuum...end.
progress: 10.0 s, 213.2 tps, lat 225.128 ms stddev 307.560, 0 failed
progress: 20.0 s, 269.3 tps, lat 186.492 ms stddev 250.545, 0 failed
progress: 30.0 s, 271.9 tps, lat 184.642 ms stddev 182.070, 0 failed
progress: 40.0 s, 274.2 tps, lat 180.323 ms stddev 217.953, 0 failed
progress: 50.0 s, 271.2 tps, lat 186.212 ms stddev 197.726, 0 failed
progress: 60.0 s, 273.0 tps, lat 181.935 ms stddev 176.126, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 15778
number of failed transactions: 0 (0.000%)
latency average = 189.962 ms
latency stddev = 224.221 ms
initial connection time = 152.961 ms
tps = 262.785025 (without initial connection time)
postgres@HNBA110232:/etc/postgresql/17/main$ pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (17.5 (Ubuntu 17.5-1.pgdg24.04+1))
pgbench: error: could not count number of branches: ERROR:  relation "pgbench_branches" does not exist
LINE 1: select count(*) from pgbench_branches
                             ^
pgbench: hint: Perhaps you need to do initialization ("pgbench -i") in database "postgres".
postgres@HNBA110232:/etc/postgresql/17/main$ pgbench -i
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
vacuuming...
creating primary keys...
done in 0.35 s (drop tables 0.00 s, create tables 0.04 s, client-side generate 0.18 s, vacuum 0.07 s, primary keys 0.06 s).
postgres@HNBA110232:/etc/postgresql/17/main$ pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (17.5 (Ubuntu 17.5-1.pgdg24.04+1))
starting vacuum...end.
progress: 6.0 s, 285.0 tps, lat 27.883 ms stddev 18.798, 0 failed
progress: 12.0 s, 282.7 tps, lat 28.294 ms stddev 17.917, 0 failed
progress: 18.0 s, 279.7 tps, lat 28.605 ms stddev 18.960, 0 failed
progress: 24.0 s, 279.0 tps, lat 28.658 ms stddev 22.192, 0 failed
progress: 30.0 s, 277.8 tps, lat 28.780 ms stddev 18.574, 0 failed
progress: 36.0 s, 286.3 tps, lat 27.897 ms stddev 18.629, 0 failed
progress: 42.0 s, 285.7 tps, lat 28.060 ms stddev 18.903, 0 failed
progress: 48.0 s, 282.3 tps, lat 28.338 ms stddev 17.189, 0 failed
progress: 54.0 s, 290.5 tps, lat 27.502 ms stddev 18.178, 0 failed
progress: 60.0 s, 285.2 tps, lat 27.990 ms stddev 18.601, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 17013
number of failed transactions: 0 (0.000%)
latency average = 28.206 ms
latency stddev = 18.849 ms
initial connection time = 22.668 ms
tps = 283.511986 (without initial connection time)
postgres@HNBA110232:/etc/postgresql/17/main$ sudo -u postgres psql -p 5432
postgres is not in the sudoers file.
postgres@HNBA110232:/etc/postgresql/17/main$ psql
psql (17.5 (Ubuntu 17.5-1.pgdg24.04+1))
Type "help" for help.

postgres=# ALTER SYSTEM SET vacuum_cost_page_miss = 5;
ALTER SYSTEM
postgres=# ALTER SYSTEM SET vacuum_cost_page_dirty = 10;
ALTER SYSTEM
postgres=# SELECT name, setting, context, short_desc
FROM pg_settingsWHERE namelike'vacuum%';
ERROR:  syntax error at or near "'vacuum%'"
LINE 2: FROM pg_settingsWHERE namelike'vacuum%';
                                      ^
postgres=# SELECT name, setting, context, short_desc
FROM pg_settings WHERE namelike'vacuum%';
ERROR:  type "namelike" does not exist
LINE 2: FROM pg_settings WHERE namelike'vacuum%';
                               ^
postgres=# SELECT name, setting, context, short_desc
FROM pg_settings WHERE name like'vacuum%';
postgres=# SELECT name, setting, context, short_desc
FROM pg_settings WHERE name like'autovacuum%';
postgres=# ALTER SYSTEM SET log_autovacuum_min_duration= 0;
ALTER SYSTEM
postgres=# ALTER SYSTEM SET autovacuum_max_workers= 10;
ALTER SYSTEM
postgres=# ALTER SYSTEM SET autovacuum_naptime= 15s;
ERROR:  trailing junk after numeric literal at or near "15s"
LINE 1: ALTER SYSTEM SET autovacuum_naptime= 15s;
                                             ^
postgres=# ALTER SYSTEM SET autovacuum_naptime= 15;
ALTER SYSTEM
postgres=# ALTER SYSTEM SET autovacuum_vacuum_threshold= 25;
ALTER SYSTEM
postgres=# ALTER SYSTEM SET autovacuum_vacuum_scale_factor= 0.05;
ALTER SYSTEM
postgres=# ALTER SYSTEM SET autovacuum_vacuum_cost_delay= 10;
ALTER SYSTEM
postgres=# ALTER SYSTEM SET autovacuum_vacuum_cost_limit= 1000;
ALTER SYSTEM
postgres=# \q
postgres@HNBA110232:/etc/postgresql/17/main$ sudo pg_ctlcluster 17 main restart
[sudo] password for postgres:
Sorry, try again.
[sudo] password for postgres:
Sorry, try again.
[sudo] password for postgres:
sudo: 3 incorrect password attempts
postgres@HNBA110232:/etc/postgresql/17/main$ sudo pg_ctlcluster 17 main restart
[sudo] password for postgres:
Sorry, try again.
[sudo] password for postgres:
Sorry, try again.
[sudo] password for postgres:
sudo: 3 incorrect password attempts
postgres@HNBA110232:/etc/postgresql/17/main$ sudo pg_ctlcluster 17 main restart
[sudo] password for postgres:
Sorry, try again.
[sudo] password for postgres:
Sorry, try again.
[sudo] password for postgres:
sudo: 3 incorrect password attempts
postgres@HNBA110232:/etc/postgresql/17/main$ sudo pg_ctlcluster 17 main restart
[sudo] password for postgres:
Sorry, try again.
[sudo] password for postgres:
Sorry, try again.
[sudo] password for postgres:
sudo: 3 incorrect password attempts
postgres@HNBA110232:/etc/postgresql/17/main$ \q
Command 'q' not found, but can be installed with:
snap install q                       # version 1.6.3-1, or
apt  install python3-q-text-as-data  # version 3.1.6-3
See 'snap info q' for additional versions.
postgres@HNBA110232:/etc/postgresql/17/main$ su nelvanov
su: user nelvanov does not exist or the user entry does not contain all the required fields
postgres@HNBA110232:/etc/postgresql/17/main$ su evlanov
su: user evlanov does not exist or the user entry does not contain all the required fields
postgres@HNBA110232:/etc/postgresql/17/main$
exit
nevlanov@HNBA110232:/etc/postgresql/17/main$ sudo pg_ctlcluster 17 main restart
[sudo] password for nevlanov:
nevlanov@HNBA110232:/etc/postgresql/17/main$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
17  main    5432 online postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
nevlanov@HNBA110232:/etc/postgresql/17/main$ sudo -u postgres psql -p 5432
psql (17.5 (Ubuntu 17.5-1.pgdg24.04+1))
Type "help" for help.

postgres=# SELECT name, setting, context, short_desc FROM pg_settings WHERE name like 'vacuum%';
postgres=# SELECT name, setting, context, short_desc FROM pg_settings WHERE name like 'autovacuum%';
postgres=# pgbench -c8 -P 6 -T 60 -U postgres postgres
postgres-# \q
nevlanov@HNBA110232:/etc/postgresql/17/main$ pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  Peer authentication failed for user "postgres"
pgbench: error: could not create connection for setup
nevlanov@HNBA110232:/etc/postgresql/17/main$ su postgres
Password:
su: Authentication failure
nevlanov@HNBA110232:/etc/postgresql/17/main$ su postgres
Password:
su: Authentication failure
nevlanov@HNBA110232:/etc/postgresql/17/main$ su postgres
Password:
su: Authentication failure
nevlanov@HNBA110232:/etc/postgresql/17/main$ su postgres
Password:
su: Authentication failure
nevlanov@HNBA110232:/etc/postgresql/17/main$ su postgres
Password:
su: Authentication failure
nevlanov@HNBA110232:/etc/postgresql/17/main$ psql
psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  role "nevlanov" does not exist
nevlanov@HNBA110232:/etc/postgresql/17/main$ sudo -u postgres psql -p 5432
psql (17.5 (Ubuntu 17.5-1.pgdg24.04+1))
Type "help" for help.

postgres=# \dt
              List of relations
 Schema |       Name       | Type  |  Owner
--------+------------------+-------+----------
 public | pgbench_accounts | table | postgres
 public | pgbench_branches | table | postgres
 public | pgbench_history  | table | postgres
 public | pgbench_tellers  | table | postgres
(4 rows)

postgres=# pgbench -c8 -P 6 -T 60 -U postgres postgres
postgres-# \q
nevlanov@HNBA110232:/etc/postgresql/17/main$ sudo su postgres
postgres@HNBA110232:/etc/postgresql/17/main$ pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (17.5 (Ubuntu 17.5-1.pgdg24.04+1))
starting vacuum...end.
progress: 6.0 s, 251.3 tps, lat 31.592 ms stddev 22.723, 0 failed
progress: 12.0 s, 277.7 tps, lat 28.802 ms stddev 19.858, 0 failed
progress: 18.0 s, 285.8 tps, lat 27.946 ms stddev 17.918, 0 failed
progress: 24.0 s, 287.0 tps, lat 27.907 ms stddev 18.989, 0 failed
progress: 30.0 s, 290.5 tps, lat 27.481 ms stddev 18.087, 0 failed
progress: 36.0 s, 289.8 tps, lat 27.628 ms stddev 18.772, 0 failed
progress: 42.0 s, 286.7 tps, lat 27.819 ms stddev 18.390, 0 failed
progress: 48.0 s, 289.5 tps, lat 27.669 ms stddev 18.510, 0 failed
progress: 54.0 s, 293.7 tps, lat 27.308 ms stddev 18.632, 0 failed
progress: 60.0 s, 290.7 tps, lat 27.485 ms stddev 18.662, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 17064
number of failed transactions: 0 (0.000%)
latency average = 28.116 ms
latency stddev = 19.077 ms
initial connection time = 29.626 ms
tps = 284.418132 (without initial connection time)
postgres@HNBA110232:/etc/postgresql/17/main$ psql
psql (17.5 (Ubuntu 17.5-1.pgdg24.04+1))
Type "help" for help.

postgres=# CREATE TABLE testHW01 (id SERIAL , text TEXT );
CREATE TABLE
postgres=# select * from testHW01
postgres-# select * from testHW01;
ERROR:  syntax error at or near "select"
LINE 2: select * from testHW01;
        ^
postgres=# ls
postgres-# ls data
postgres-# select * from testHW01;
ERROR:  syntax error at or near "ls"
LINE 1: ls
        ^
postgres=# select * from testHW01;
 id | text
----+------
(0 rows)

postgres=# INSERT INTO testHW01(text) select 'x3x' from generate_series(1,1000000);
INSERT 0 1000000
postgres=# select * from testHW01;
postgres=# select * from testHW01;
postgres=# SELECT pg_size_pretty(pg_total_relation_size('testHW01'));
 pg_size_pretty
----------------
 35 MB
(1 row)

postgres=# UPDATE testHW01 SET text = text || '4';
UPDATE 1000000
postgres=# select * from testHW01;
postgres=# UPDATE testHW01 SET text = text || 'x';
UPDATE 1000000
postgres=# UPDATE testHW01 SET text = text || '5';
UPDATE 1000000
postgres=# UPDATE testHW01 SET text = text || 'x';
UPDATE 1000000
postgres=# UPDATE testHW01 SET text = text || '6';
UPDATE 1000000
postgres=# select * from testHW01;
postgres=# SELECT     relname AS table_name, n_dead_tup AS dead_rows, last_autovacuum AS last_autovacuum_time FROM pg_stat_user_tables
WHERE
    relname = 'testHW01';
 table_name | dead_rows | last_autovacuum_time
------------+-----------+----------------------
(0 rows)

postgres=# select c.relname,
current_setting('autovacuum_vacuum_threshold') as av_base_thresh,
current_setting('autovacuum_vacuum_scale_factor') as av_scale_factor,
(current_setting('autovacuum_vacuum_threshold')::int +
(current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples)) as av_thresh,
s.n_dead_tup
from pg_stat_user_tables s join pg_class c ON s.relname = c.relname
where s.n_dead_tup > (current_setting('autovacuum_vacuum_threshold')::int
+ (current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples));
 relname | av_base_thresh | av_scale_factor | av_thresh | n_dead_tup
---------+----------------+-----------------+-----------+------------
(0 rows)

postgres=# select c.relname,
current_setting('autovacuum_vacuum_threshold') as av_base_thresh,
current_setting('autovacuum_vacuum_scale_factor') as av_scale_factor,
(current_setting('autovacuum_vacuum_threshold')::int +
(current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples)) as av_thresh,
s.n_dead_tup
from pg_stat_user_tables s join pg_class c ON s.relname = c.relname
where s.n_dead_tup > (current_setting('autovacuum_vacuum_threshold')::int
+ (current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples));
 relname | av_base_thresh | av_scale_factor | av_thresh | n_dead_tup
---------+----------------+-----------------+-----------+------------
(0 rows)

postgres=# SELECT     relname AS table_name, n_dead_tup AS dead_rows, last_autovacuum AS last_autovacuum_time FROM pg_stat_user_tables
WHERE
    relname = 'testHW01';
 table_name | dead_rows | last_autovacuum_time
------------+-----------+----------------------
(0 rows)

postgres=# ALTER TABLE testHW01 SET (autovacuum_enabled = off);
ALTER TABLE
postgres=# SELECT     relname AS table_name, n_dead_tup AS dead_rows, last_autovacuum AS last_autovacuum_time FROM pg_stat_user_tables
WHERE
    relname = 'testHW01';
 table_name | dead_rows | last_autovacuum_time
------------+-----------+----------------------
(0 rows)

postgres=# SELECT pg_size_pretty(pg_total_relation_size('testHW01'));
 pg_size_pretty
----------------
 169 MB
(1 row)

postgres=# UPDATE testHW01 SET text = text || 'x';
UPDATE 1000000
postgres=# UPDATE testHW01 SET text = text || '7';
UPDATE 1000000
postgres=# UPDATE testHW01 SET text = text || 'x';
UPDATE 1000000
postgres=# UPDATE testHW01 SET text = text || '8';
UPDATE 1000000
postgres=# UPDATE testHW01 SET text = text || 'x';
UPDATE 1000000
postgres=# SELECT pg_size_pretty(pg_total_relation_size('testHW01'));
 pg_size_pretty
----------------
 268 MB
(1 row)

postgres=# SELECT     relname AS table_name, n_dead_tup AS dead_rows, last_autovacuum AS last_autovacuum_time FROM pg_stat_user_tables
WHERE
    relname = 'testHW01';
 table_name | dead_rows | last_autovacuum_time
------------+-----------+----------------------
(0 rows)

postgres=# select c.relname,current_setting('autovacuum_vacuum_threshold') as av_base_thresh,
current_setting('autovacuum_vacuum_scale_factor') as av_scale_factor,
(current_setting('autovacuum_vacuum_threshold')::int +
(current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples)) as av_thresh,
s.n_dead_tup
from pg_stat_user_tables s join pg_class c ON s.relname = c.relname
where s.n_dead_tup > (current_setting('autovacuum_vacuum_threshold')::int
+ (current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples));
 relname  | av_base_thresh | av_scale_factor |     av_thresh     | n_dead_tup
----------+----------------+-----------------+-------------------+------------
 testhw01 | 25             | 0.05            | 75016.90000000001 |    4998988
(1 row)

postgres=# select c.relname,
current_setting('autovacuum_vacuum_threshold') as av_base_thresh,
current_setting('autovacuum_vacuum_scale_factor') as av_scale_factor,
(current_setting('autovacuum_vacuum_threshold')::int +
(current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples)) as av_thresh,
s.n_dead_tup
from pg_stat_user_tables s join pg_class c ON s.relname = c.relname
where s.n_dead_tup > (current_setting('autovacuum_vacuum_threshold')::int
+ (current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples));
 relname  | av_base_thresh | av_scale_factor |     av_thresh     | n_dead_tup
----------+----------------+-----------------+-------------------+------------
 testhw01 | 25             | 0.05            | 75016.90000000001 |    4998988
(1 row)

postgres=# ALTER TABLE testHW01 SET (autovacuum_enabled = on);
ALTER TABLE
postgres=# select c.relname,
current_setting('autovacuum_vacuum_threshold') as av_base_thresh,
current_setting('autovacuum_vacuum_scale_factor') as av_scale_factor,
(current_setting('autovacuum_vacuum_threshold')::int +
(current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples)) as av_thresh,
s.n_dead_tup
from pg_stat_user_tables s join pg_class c ON s.relname = c.relname
where s.n_dead_tup > (current_setting('autovacuum_vacuum_threshold')::int
+ (current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples));
 relname | av_base_thresh | av_scale_factor | av_thresh | n_dead_tup
---------+----------------+-----------------+-----------+------------
(0 rows)

postgres=# SELECT pg_size_pretty(pg_total_relation_size('testHW01'));
 pg_size_pretty
----------------
 269 MB
(1 row)

postgres=# vacuum full testHW01;
VACUUM
postgres=# SELECT pg_size_pretty(pg_total_relation_size('testHW01'));
 pg_size_pretty
----------------
 50 MB
(1 row)

postgres=#


# с включенным автовакуумом "мертвые записи" из таблицы удаляются сами
с выключенным - остаются. при включении автовакуума обратно записи чистятся, но размер таблицы не уменьшается - уменьшаем через вакуум фулл