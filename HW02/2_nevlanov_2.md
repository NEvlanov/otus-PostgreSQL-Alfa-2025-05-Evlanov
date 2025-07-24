nevlanov@HNBA110232:~$ sudo -u postgres psql -p 5432
[sudo] password for nevlanov:
Sorry, try again.
[sudo] password for nevlanov:
psql (17.5 (Ubuntu 17.5-1.pgdg24.04+1))
Type "help" for help.

postgres=# \c otus_nevlanov1
You are now connected to database "otus_nevlanov1" as user "postgres".
otus_nevlanov1=# BEGIN;
UPDATE  test2 SET amount = '2000' WHERE i = 1;
BEGIN
UPDATE 1
otus_nevlanov1=*# commit;
COMMIT
otus_nevlanov1=# BEGIN;
UPDATE  test2 SET amount = '2000' WHERE i = 3;
BEGIN
UPDATE 1
otus_nevlanov1=*# UPDATE  test2 SET amount = '2000' WHERE i = 1;
ERROR:  deadlock detected
DETAIL:  Process 11611 waits for ShareLock on transaction 66155; blocked by process 12861.
Process 12861 waits for ShareLock on transaction 66156; blocked by process 13172.
Process 13172 waits for ShareLock on transaction 66157; blocked by process 11611.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,8) in relation "test2"
otus_nevlanov1=!# commit;
ROLLBACK
otus_nevlanov1=#  select * from test2;
 i | amount
---+--------
 2 |    500
 1 |   2000
 3 |     30
(3 rows)

otus_nevlanov1=#
