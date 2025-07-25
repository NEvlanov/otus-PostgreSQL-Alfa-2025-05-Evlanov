

# Создаем тестовую таблицу для работы с блокировками

postgres=# \c otus_nevlanov1
You are now connected to database "otus_nevlanov1" as user "postgres".
otus_nevlanov1=#
CREATE TABLE test2 (i serial, amount int);
INSERT INTO test2(amount) VALUES (100),(500);
CREATE TABLE
INSERT 0 2
otus_nevlanov1=#
otus_nevlanov1=#
otus_nevlanov1=# SELECT * FROM test2;
 i | amount
---+--------
 1 |    100
 2 |    500
(2 rows)


# Настройка сервера для сохранения журнала и таймаута в 200мс

otus_nevlanov1=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)

otus_nevlanov1=# ALTER SYSTEM SET log_lock_waits = on;
ALTER SYSTEM SET deadlock_timeout = '200ms';
ALTER SYSTEM
ALTER SYSTEM
otus_nevlanov1=# SHOW log_lock_waits;
 log_lock_waits
----------------
 off
(1 row)

otus_nevlanov1=# SHOW deadlock_timeout;
 deadlock_timeout
------------------
 1s
(1 row)

otus_nevlanov1=# \q
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


# Создаем блокировку

postgres=# \c otus_nevlanov1
You are now connected to database "otus_nevlanov1" as user "postgres".
otus_nevlanov1=# SHOW log_lock_waits;
 log_lock_waits
----------------
 on
(1 row)

otus_nevlanov1=# SHOW deadlock_timeout;
 deadlock_timeout
------------------
 200ms
(1 row)

otus_nevlanov1=# SELECT * FROM test2;
 i | amount
---+--------
 1 |    100
 2 |    500
(2 rows)

otus_nevlanov1=# BEGIN;
UPDATE  test2 SET amount = '0' WHERE id = 1;
BEGIN
ERROR:  column "id" does not exist
LINE 1: UPDATE  test2 SET amount = '0' WHERE id = 1;
                                             ^
HINT:  Perhaps you meant to reference the column "test2.i".
otus_nevlanov1=!# BEGIN;
UPDATE  test2 SET amount = '0' WHERE i = 1;
ERROR:  current transaction is aborted, commands ignored until end of transaction block
ERROR:  current transaction is aborted, commands ignored until end of transaction block
otus_nevlanov1=!# commit;
ROLLBACK
otus_nevlanov1=# SELECT * FROM test2;
 i | amount
---+--------
 1 |    100
 2 |    500
(2 rows)

otus_nevlanov1=# UPDATE  test2 SET amount = '0' WHERE i = 1;
UPDATE 1
otus_nevlanov1=# SELECT * FROM test2;
 i | amount
---+--------
 2 |    500
 1 |      0
(2 rows)

otus_nevlanov1=#  BEGIN;
UPDATE  test2 SET amount = '100' WHERE i = 1;
BEGIN
UPDATE 1
otus_nevlanov1=*# SELECT * FROM test2;
 i | amount
---+--------
 2 |    500
 1 |    100
(2 rows)

otus_nevlanov1=*# commit;
COMMIT
otus_nevlanov1=# SELECT * FROM test2;
 i | amount
---+--------
 2 |    500
 1 |    100
(2 rows)

otus_nevlanov1=# SELECT * FROM test2;
 i | amount
---+--------
 2 |    500
 1 |    200
(2 rows)

otus_nevlanov1=#  select * from pg_locks;
otus_nevlanov1=#  select * from pg_locks;


# Смотрим итоги по блокировке в представлении pg_locks

 locktype  | database | relation | page | tuple | virtualxid | transactionid | classid | objid | objsubid | virtualtransaction |  pid  |      mode       | granted | fastpath | waitstart
------------+----------+----------+------+-------+------------+---------------+---------+-------+----------+--------------------+-------+-----------------+---------+----------+-----------
 relation   |    16388 |    12073 |      |       |            |               |         |       |          | 12/6               | 11057 | AccessShareLock | t       | t        |
 virtualxid |          |          |      |       | 12/6       |               |         |       |          | 12/6               | 11057 | ExclusiveLock   | t       | t        |
(2 rows)



# Моделируем ситуацию с тремя апдейтами в разных сенсах - видим блокировки в pg_locks и в логе

postgres=# \c otus_nevlanov1
You are now connected to database "otus_nevlanov1" as user "postgres".
otus_nevlanov1=# select * from pg_locks;
otus_nevlanov1=# select * from test2;
 i | amount
---+--------
 2 |    500
 1 |    200
(2 rows)

otus_nevlanov1=# BEGIN;
UPDATE  test2 SET amount = '100' WHERE i = 1;
BEGIN
UPDATE 1
otus_nevlanov1=*# commit;
COMMIT
otus_nevlanov1=# select * from test2;
 i | amount
---+--------
 2 |    500
 1 |   2000
(2 rows)

otus_nevlanov1=# select * from pg_locks;

  locktype  | database | relation | page | tuple | virtualxid | transactionid | classid | objid | objsubid | virtualtransaction |  pid  |      mode       | granted | fastpath | waitstart
------------+----------+----------+------+-------+------------+---------------+---------+-------+----------+--------------------+-------+-----------------+---------+----------+-----------
 relation   |    16388 |    12073 |      |       |            |               |         |       |          | 8/6                | 11539 | AccessShareLock | t       | t        |
 virtualxid |          |          |      |       | 8/6        |               |         |       |          | 8/6                | 11539 | ExclusiveLock   | t       | t        |
(2 rows)

otus_nevlanov1=# \q
nevlanov@HNBA110232:/etc/postgresql/17/main$ tail -n 10 /var/log/postgresql/postgresql-17-main.log
2025-07-24 17:53:46.160 MSK [11611] postgres@otus_nevlanov1 DETAIL:  Process holding the lock: 11523. Wait queue: 11611.
2025-07-24 17:53:46.160 MSK [11611] postgres@otus_nevlanov1 CONTEXT:  while locking tuple (0,6) in relation "test2"
2025-07-24 17:53:46.160 MSK [11611] postgres@otus_nevlanov1 STATEMENT:  UPDATE  test2 SET amount = '2000' WHERE i = 1;
2025-07-24 17:53:55.188 MSK [11611] postgres@otus_nevlanov1 LOG:  process 11611 acquired ShareLock on transaction 66151 after 9227.852 ms
2025-07-24 17:53:55.188 MSK [11611] postgres@otus_nevlanov1 CONTEXT:  while locking tuple (0,6) in relation "test2"
2025-07-24 17:53:55.188 MSK [11611] postgres@otus_nevlanov1 STATEMENT:  UPDATE  test2 SET amount = '2000' WHERE i = 1;
2025-07-24 17:58:13.502 MSK [11448] LOG:  checkpoint starting: time
2025-07-24 17:58:14.057 MSK [11448] LOG:  checkpoint complete: wrote 6 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.509 s, sync=0.011 s, total=0.556 s; sync files=5, longest=0.007 s, average=0.003 s; distance=11 kB, estimate=11 kB; lsn=0/74BE0AE0, redo lsn=0/74BE0A88
2025-07-24 18:03:13.157 MSK [11448] LOG:  checkpoint starting: time
2025-07-24 18:03:19.111 MSK [11448] LOG:  checkpoint complete: wrote 59 buffers (0.4%); 0 WAL file(s) added, 0 removed, 0 recycled; write=5.843 s, sync=0.056 s, total=5.954 s; sync files=17, longest=0.011 s, average=0.004 s; distance=339 kB, estimate=339 kB; lsn=0/74C35A40, redo lsn=0/74C359B0
nevlanov@HNBA110232:/etc/postgresql/17/main$ sudo -u postgres psql -p 5432
[sudo] password for nevlanov:
psql (17.5 (Ubuntu 17.5-1.pgdg24.04+1))
Type "help" for help.


# Воспроизводим взаимоблокировку трех транзакций - видим результат в логе

postgres=# \c otus_nevlanov1
You are now connected to database "otus_nevlanov1" as user "postgres".
otus_nevlanov1=# select * from test2;
 i | amount
---+--------
 2 |    500
 1 |   2000
(2 rows)

otus_nevlanov1=# INSERT INTO  test2 VALUES (3,30);
INSERT 0 1
otus_nevlanov1=# select * from test2;
 i | amount
---+--------
 2 |    500
 1 |   2000
 3 |     30
(3 rows)

otus_nevlanov1=# BEGIN;
UPDATE  test2 SET amount = '0' WHERE id = 1;
BEGIN
ERROR:  column "id" does not exist
LINE 1: UPDATE  test2 SET amount = '0' WHERE id = 1;
                                             ^
HINT:  Perhaps you meant to reference the column "test2.i".
otus_nevlanov1=!# rollback;
ROLLBACK
otus_nevlanov1=# select * from test2;
 i | amount
---+--------
 2 |    500
 1 |   2000
 3 |     30
(3 rows)

otus_nevlanov1=# BEGIN;
UPDATE  test2 SET amount = '0' WHERE i = 1;
BEGIN
UPDATE 1
otus_nevlanov1=*# UPDATE  test2 SET amount = '0' WHERE i = 2;
UPDATE 1
otus_nevlanov1=*# ROLLBACK;
ROLLBACK
otus_nevlanov1=# select * from test2;
 i | amount
---+--------
 2 |    500
 1 |   2000
 3 |     30
(3 rows)


# Ошибка в другой сессии

ERROR:  deadlock detected
DETAIL:  Process 11611 waits for ShareLock on transaction 66155; blocked by process 12861.
Process 12861 waits for ShareLock on transaction 66156; blocked by process 13172.
Process 13172 waits for ShareLock on transaction 66157; blocked by process 11611.
HINT:  See server log for query details.


# Результат из лога

otus_nevlanov1=# \q
nevlanov@HNBA110232:/etc/postgresql/17/main$ tail -n 10 /var/log/postgresql/postgresql-17-main.log
2025-07-24 18:27:51.247 MSK [11611] postgres@otus_nevlanov1 CONTEXT:  while updating tuple (0,8) in relation "test2"
2025-07-24 18:27:51.247 MSK [11611] postgres@otus_nevlanov1 STATEMENT:  UPDATE  test2 SET amount = '2000' WHERE i = 1;
2025-07-24 18:27:51.248 MSK [13172] postgres@otus_nevlanov1 LOG:  process 13172 acquired ShareLock on transaction 66157 after 6871.801 ms
2025-07-24 18:27:51.248 MSK [13172] postgres@otus_nevlanov1 CONTEXT:  while updating tuple (0,9) in relation "test2"
2025-07-24 18:27:51.248 MSK [13172] postgres@otus_nevlanov1 STATEMENT:  UPDATE  test2 SET amount = '1000' WHERE i = 3;
2025-07-24 18:28:13.532 MSK [11448] LOG:  checkpoint starting: time
2025-07-24 18:28:13.681 MSK [11448] LOG:  checkpoint complete: wrote 2 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.108 s, sync=0.011 s, total=0.149 s; sync files=2, longest=0.007 s, average=0.006 s; distance=1 kB, estimate=305 kB; lsn=0/74C35F88, redo lsn=0/74C35F28
2025-07-24 18:28:39.062 MSK [12861] postgres@otus_nevlanov1 LOG:  process 12861 acquired ShareLock on transaction 66156 after 61903.723 ms
2025-07-24 18:28:39.062 MSK [12861] postgres@otus_nevlanov1 CONTEXT:  while updating tuple (0,2) in relation "test2"
2025-07-24 18:28:39.062 MSK [12861] postgres@otus_nevlanov1 STATEMENT:  UPDATE  test2 SET amount = '0' WHERE i = 2;
nevlanov@HNBA110232:/etc/postgresql/17/main$
