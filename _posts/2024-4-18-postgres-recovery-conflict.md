---
layout: article
title: Postgres 复制冲突
---
## conflict_lock

On a primary server, create the following table:

```
postgres=# CREATE TABLE foo (id INT);
CREATE TABLE
postgres=# INSERT INTO foo VALUES(generate_series(1,100));
INSERT 0 100
```

On a standby, begin a transaction which holds a lock on the table, e.g.:

```
postgres=# BEGIN ;
BEGIN

postgres=*# SELECT * FROM foo WHERE id < 2;
 id
----
  1
(1 row)
```

On the primary server, drop the table:

```
postgres=# DROP TABLE foo;
DROP TABLE
```

After a while (at most the interval defined by `max_standby_streaming_delay`, default 30s) on the standby a further attempt to access the table will result in:

```
postgres=*# SELECT * FROM foo WHERE id < 2;
FATAL:  terminating connection due to conflict with recovery
DETAIL:  User was holding a relation lock for too long.
HINT:  In a moment you should be able to reconnect to the database and repeat your command.
server closed the connection unexpectedly
        This probably means the server terminated abnormally
        before or while processing the request.
The connection to the server was lost. Attempting reset: Succeeded.
```

This will be recorded in the `confl_lock` column:

```
postgres=# SELECT * FROM pg_stat_database_conflicts WHERE datname=CURRENT_DATABASE();
 datid | datname  | confl_tablespace | confl_lock | confl_snapshot | confl_bufferpin | confl_deadlock 
-------+----------+------------------+------------+----------------+-----------------+----------------
 13579 | postgres |                0 |          1 |              0 |               0 |              0
(1 row)
```





```
select n_tup_del, n_tup_hot_upd, n_live_tup, n_dead_tup from pg_stat_all_tables where relname = 'f';
```

