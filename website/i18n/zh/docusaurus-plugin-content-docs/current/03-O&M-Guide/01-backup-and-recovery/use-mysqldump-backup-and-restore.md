---
id: use-mysqldump-backup-and-restore
sidebar_position: 4.31
---

# 使用mysqldump备份恢复StoneDB

## mysqldump介绍
mysqldump是MySQL 执行逻辑备份的工具，执行mysqldump后会生成一组SQL语句，可以通过这些语句来重现原始数据库定义的库表数据，它可以转储一个或者多个本地MySQL数据库或者远程可访问的数据库备份。mysqldump使用可以参考MySQL官方文档说明：[4.5.4 mysqldump — A Database Backup Program](https://dev.mysql.com/doc/refman/5.6/en/mysqldump.html)，或者参考以下mysqldump 使用参数：
```bash
# mysqldump --help
mysqldump  Ver 10.13 Distrib 5.6.24-StoneDB, for Linux (x86_64)
Copyright (c) 2000, 2022 StoneAtom Group Holding Limited
Dumping structure and contents of MySQL databases and tables.
Usage: mysqldump [OPTIONS] database [tables]
OR     mysqldump [OPTIONS] --databases [OPTIONS] DB1 [DB2 DB3...]
OR     mysqldump [OPTIONS] --all-databases [OPTIONS]

Default options are read from the following files in the given order:
/etc/stonedb.cnf /etc/mysql/stonedb.cnf /stonedb56/install/stonedb.cnf ~/.stonedb.cnf
The following groups are read: mysqldump client
The following options may be given as the first argument:
--print-defaults        Print the program argument list and exit.
--no-defaults           Don't read default options from any option file,
                        except for login file.
--defaults-file=#       Only read default options from the given file #.
--defaults-extra-file=# Read this file after the global files are read.
--defaults-group-suffix=#
                        Also read groups with concat(group, suffix)
--login-path=#          Read this path from the login file.
  -A, --all-databases Dump all the databases. This will be same as --databases
                      with all databases selected.
  -Y, --all-tablespaces
                      Dump all the tablespaces.
  -y, --no-tablespaces
                      Do not dump any tablespace information.
  --add-drop-database Add a DROP DATABASE before each create.
  --add-drop-table    Add a DROP TABLE before each create.
                      (Defaults to on; use --skip-add-drop-table to disable.)
  --add-drop-trigger  Add a DROP TRIGGER before each create.
  --add-locks         Add locks around INSERT statements.
                      (Defaults to on; use --skip-add-locks to disable.)
  --allow-keywords    Allow creation of column names that are keywords.
  --apply-slave-statements
                      Adds 'STOP SLAVE' prior to 'CHANGE MASTER' and 'START
                      SLAVE' to bottom of dump.
  --bind-address=name IP address to bind to.
  --character-sets-dir=name
                      Directory for character set files.
  -i, --comments      Write additional information.
                      (Defaults to on; use --skip-comments to disable.)
  --compatible=name   Change the dump to be compatible with a given mode. By
                      default tables are dumped in a format optimized for
                      MySQL. Legal modes are: ansi, mysql323, mysql40,
                      postgresql, oracle, mssql, db2, maxdb, no_key_options,
                      no_table_options, no_field_options. One can use several
                      modes separated by commas. Note: Requires MySQL server
                      version 4.1.0 or higher. This option is ignored with
                      earlier server versions.
  --compact           Give less verbose output (useful for debugging). Disables
                      structure comments and header/footer constructs.  Enables
                      options --skip-add-drop-table --skip-add-locks
                      --skip-comments --skip-disable-keys --skip-set-charset.
  -c, --complete-insert
                      Use complete insert statements.
  -C, --compress      Use compression in server/client protocol.
  -a, --create-options
                      Include all MySQL specific create options.
                      (Defaults to on; use --skip-create-options to disable.)
  -B, --databases     Dump several databases. Note the difference in usage; in
                      this case no tables are given. All name arguments are
                      regarded as database names. 'USE db_name;' will be
                      included in the output.
  -#, --debug[=#]     This is a non-debug version. Catch this and exit.
  --debug-check       Check memory and open file usage at exit.
  --debug-info        Print some debug info at exit.
  --default-character-set=name
                      Set the default character set.
  --delayed-insert    Insert rows with INSERT DELAYED.
  --delete-master-logs
                      Delete logs on master after backup. This automatically
                      enables --master-data.
  -K, --disable-keys  '/*!40000 ALTER TABLE tb_name DISABLE KEYS */; and
                      '/*!40000 ALTER TABLE tb_name ENABLE KEYS */; will be put
                      in the output.
                      (Defaults to on; use --skip-disable-keys to disable.)
  --dump-slave[=#]    This causes the binary log position and filename of the
                      master to be appended to the dumped data output. Setting
                      the value to 1, will printit as a CHANGE MASTER command
                      in the dumped data output; if equal to 2, that command
                      will be prefixed with a comment symbol. This option will
                      turn --lock-all-tables on, unless --single-transaction is
                      specified too (in which case a global read lock is only
                      taken a short time at the beginning of the dump - don't
                      forget to read about --single-transaction below). In all
                      cases any action on logs will happen at the exact moment
                      of the dump.Option automatically turns --lock-tables off.
  -E, --events        Dump events.
  -e, --extended-insert
                      Use multiple-row INSERT syntax that include several
                      VALUES lists.
                      (Defaults to on; use --skip-extended-insert to disable.)
  --fields-terminated-by=name
                      Fields in the output file are terminated by the given
                      string.
  --fields-enclosed-by=name
                      Fields in the output file are enclosed by the given
                      character.
  --fields-optionally-enclosed-by=name
                      Fields in the output file are optionally enclosed by the
                      given character.
  --fields-escaped-by=name
                      Fields in the output file are escaped by the given
                      character.
  -F, --flush-logs    Flush logs file in server before starting dump. Note that
                      if you dump many databases at once (using the option
                      --databases= or --all-databases), the logs will be
                      flushed for each database dumped. The exception is when
                      using --lock-all-tables or --master-data: in this case
                      the logs will be flushed only once, corresponding to the
                      moment all tables are locked. So if you want your dump
                      and the log flush to happen at the same exact moment you
                      should use --lock-all-tables or --master-data with
                      --flush-logs.
  --flush-privileges  Emit a FLUSH PRIVILEGES statement after dumping the mysql
                      database.  This option should be used any time the dump
                      contains the mysql database and any other database that
                      depends on the data in the mysql database for proper
                      restore.
  -f, --force         Continue even if we get an SQL error.
  -?, --help          Display this help message and exit.
  --hex-blob          Dump binary strings (BINARY, VARBINARY, BLOB) in
                      hexadecimal format.
  -h, --host=name     Connect to host.
  --ignore-table=name Do not dump the specified table. To specify more than one
                      table to ignore, use the directive multiple times, once
                      for each table.  Each table must be specified with both
                      database and table names, e.g.,
                      --ignore-table=database.table.
  --include-master-host-port
                      Adds 'MASTER_HOST=<host>, MASTER_PORT=<port>' to 'CHANGE
                      MASTER TO..' in dump produced with --dump-slave.
  --insert-ignore     Insert rows with INSERT IGNORE.
  --lines-terminated-by=name
                      Lines in the output file are terminated by the given
                      string.
  -x, --lock-all-tables
                      Locks all tables across all databases. This is achieved
                      by taking a global read lock for the duration of the
                      whole dump. Automatically turns --single-transaction and
                      --lock-tables off.
  -l, --lock-tables   Lock all tables for read.
                      (Defaults to on; use --skip-lock-tables to disable.)
  --log-error=name    Append warnings and errors to given file.
  --master-data[=#]   This causes the binary log position and filename to be
                      appended to the output. If equal to 1, will print it as a
                      CHANGE MASTER command; if equal to 2, that command will
                      be prefixed with a comment symbol. This option will turn
                      --lock-all-tables on, unless --single-transaction is
                      specified too (in which case a global read lock is only
                      taken a short time at the beginning of the dump; don't
                      forget to read about --single-transaction below). In all
                      cases, any action on logs will happen at the exact moment
                      of the dump. Option automatically turns --lock-tables
                      off.
  --max-allowed-packet=#
                      The maximum packet length to send to or receive from
                      server.
  --net-buffer-length=#
                      The buffer size for TCP/IP and socket communication.
  --no-autocommit     Wrap tables with autocommit/commit statements.
  -n, --no-create-db  Suppress the CREATE DATABASE ... IF EXISTS statement that
                      normally is output for each dumped database if
                      --all-databases or --databases is given.
  -t, --no-create-info
                      Don't write table creation info.
  -d, --no-data       No row information.
  -N, --no-set-names  Same as --skip-set-charset.
  --opt               Same as --add-drop-table, --add-locks, --create-options,
                      --quick, --extended-insert, --lock-tables, --set-charset,
                      and --disable-keys. Enabled by default, disable with
                      --skip-opt.
  --order-by-primary  Sorts each table's rows by primary key, or first unique
                      key, if such a key exists.  Useful when dumping a MyISAM
                      table to be loaded into an InnoDB table, but will make
                      the dump itself take considerably longer.
  -p, --password[=name]
                      Password to use when connecting to server. If password is
                      not given it's solicited on the tty.
  -P, --port=#        Port number to use for connection.
  --protocol=name     The protocol to use for connection (tcp, socket, pipe,
                      memory).
  -q, --quick         Don't buffer query, dump directly to stdout.
                      (Defaults to on; use --skip-quick to disable.)
  -Q, --quote-names   Quote table and column names with backticks (`).
                      (Defaults to on; use --skip-quote-names to disable.)
  --replace           Use REPLACE INTO instead of INSERT INTO.
  -r, --result-file=name
                      Direct output to a given file. This option should be used
                      in systems (e.g., DOS, Windows) that use carriage-return
                      linefeed pairs (\r\n) to separate text lines. This option
                      ensures that only a single newline is used.
  -R, --routines      Dump stored routines (functions and procedures).
  --set-charset       Add 'SET NAMES default_character_set' to the output.
                      (Defaults to on; use --skip-set-charset to disable.)
  --set-gtid-purged[=name]
                      Add 'SET @@GLOBAL.GTID_PURGED' to the output. Possible
                      values for this option are ON, OFF and AUTO. If ON is
                      used and GTIDs are not enabled on the server, an error is
                      generated. If OFF is used, this option does nothing. If
                      AUTO is used and GTIDs are enabled on the server, 'SET
                      @@GLOBAL.GTID_PURGED' is added to the output. If GTIDs
                      are disabled, AUTO does nothing. If no value is supplied
                      then the default (AUTO) value will be considered.
  --single-transaction
                      Creates a consistent snapshot by dumping all tables in a
                      single transaction. Works ONLY for tables stored in
                      storage engines which support multiversioning (currently
                      only InnoDB does); the dump is NOT guaranteed to be
                      consistent for other storage engines. While a
                      --single-transaction dump is in process, to ensure a
                      valid dump file (correct table contents and binary log
                      position), no other connection should use the following
                      statements: ALTER TABLE, DROP TABLE, RENAME TABLE,
                      TRUNCATE TABLE, as consistent snapshot is not isolated
                      from them. Option automatically turns off --lock-tables.
  --dump-date         Put a dump date to the end of the output.
                      (Defaults to on; use --skip-dump-date to disable.)
  --skip-opt          Disable --opt. Disables --add-drop-table, --add-locks,
                      --create-options, --quick, --extended-insert,
                      --lock-tables, --set-charset, and --disable-keys.
  -S, --socket=name   The socket file to use for connection.
  --secure-auth       Refuse client connecting to server if it uses old
                      (pre-4.1.1) protocol.
                      (Defaults to on; use --skip-secure-auth to disable.)
  --ssl               Enable SSL for connection (automatically enabled with
                      other flags).
  --ssl-ca=name       CA file in PEM format (check OpenSSL docs, implies
                      --ssl).
  --ssl-capath=name   CA directory (check OpenSSL docs, implies --ssl).
  --ssl-cert=name     X509 cert in PEM format (implies --ssl).
  --ssl-cipher=name   SSL cipher to use (implies --ssl).
  --ssl-key=name      X509 key in PEM format (implies --ssl).
  --ssl-crl=name      Certificate revocation list (implies --ssl).
  --ssl-crlpath=name  Certificate revocation list path (implies --ssl).
  --ssl-verify-server-cert
                      Verify server's "Common Name" in its cert against
                      hostname used when connecting. This option is disabled by
                      default.
  -T, --tab=name      Create tab-separated textfile for each table to given
                      path. (Create .sql and .txt files.) NOTE: This only works
                      if mysqldump is run on the same machine as the mysqld
                      server.
  --tables            Overrides option --databases (-B).
  --triggers          Dump triggers for each dumped table.
                      (Defaults to on; use --skip-triggers to disable.)
  --tz-utc            SET TIME_ZONE='+00:00' at top of dump to allow dumping of
                      TIMESTAMP data when a server has data in different time
                      zones or data is being moved between servers with
                      different time zones.
                      (Defaults to on; use --skip-tz-utc to disable.)
  -u, --user=name     User for login if not current user.
  -v, --verbose       Print info about the various stages.
  -V, --version       Output version information and exit.
  -w, --where=name    Dump only selected records. Quotes are mandatory.
  -X, --xml           Dump a database as well formed XML.
  --plugin-dir=name   Directory for client-side plugins.
  --default-auth=name Default authentication client-side plugin to use.

Variables (--variable-name=value)
and boolean options {FALSE|TRUE}  Value (after reading options)
--------------------------------- ----------------------------------------
all-databases                     FALSE
all-tablespaces                   FALSE
no-tablespaces                    FALSE
add-drop-database                 FALSE
add-drop-table                    TRUE
add-drop-trigger                  FALSE
add-locks                         TRUE
allow-keywords                    FALSE
apply-slave-statements            FALSE
bind-address                      (No default value)
character-sets-dir                (No default value)
comments                          TRUE
compatible                        (No default value)
compact                           FALSE
complete-insert                   FALSE
compress                          FALSE
create-options                    TRUE
databases                         FALSE
debug-check                       FALSE
debug-info                        FALSE
default-character-set             utf8
delayed-insert                    FALSE
delete-master-logs                FALSE
disable-keys                      TRUE
dump-slave                        0
events                            FALSE
extended-insert                   TRUE
fields-terminated-by              (No default value)
fields-enclosed-by                (No default value)
fields-optionally-enclosed-by     (No default value)
fields-escaped-by                 (No default value)
flush-logs                        FALSE
flush-privileges                  FALSE
force                             FALSE
hex-blob                          FALSE
host                              (No default value)
include-master-host-port          FALSE
insert-ignore                     FALSE
lines-terminated-by               (No default value)
lock-all-tables                   FALSE
lock-tables                       TRUE
log-error                         (No default value)
master-data                       0
max-allowed-packet                536870912
net-buffer-length                 1046528
no-autocommit                     FALSE
no-create-db                      FALSE
no-create-info                    FALSE
no-data                           FALSE
order-by-primary                  FALSE
port                              3308
quick                             TRUE
quote-names                       TRUE
replace                           FALSE
routines                          FALSE
set-charset                       TRUE
single-transaction                FALSE
dump-date                         TRUE
socket                            /stonedb56/install/tmp/mysql.sock
secure-auth                       TRUE
ssl                               FALSE
ssl-ca                            (No default value)
ssl-capath                        (No default value)
ssl-cert                          (No default value)
ssl-cipher                        (No default value)
ssl-key                           (No default value)
ssl-crl                           (No default value)
ssl-crlpath                       (No default value)
ssl-verify-server-cert            FALSE
tab                               (No default value)
triggers                          TRUE
tz-utc                            TRUE
user                              (No default value)
verbose                           FALSE
where                             (No default value)
plugin-dir                        (No default value)
default-auth                      (No default value)
```
## 备份StoneDB注意事项
StoneDB不支持lock tables 操作，所以需要在备份时加上--skip-opt 或者--skip-add-locks 参数，去除备份文件中的LOCK TABLES `xxxx` WRITE;  否则备份数据将无法导入到StoneDB。
## 使用示例
### 备份
#### 创建备份库表和测试数据
```bash
# /stonedb56/install/bin/mysql -uroot -p***** -P3306
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 878
Server version: 5.7.36-StoneDB-log build-

Copyright (c) 2000, 2022 StoneAtom Group Holding Limited
No entry for terminal type "xterm";
using dumb terminal settings.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database dumpdb;
Query OK, 1 row affected (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| cache              |
| dumpdb             |
| innodb             |
| mysql              |
| performance_schema |
| sys_stonedb        |
| test               |
+--------------------+
8 rows in set (0.00 sec)

mysql> use dumpdb
Database changed

mysql> create table dumptb(id int primary key,vname varchar(20))engine=StoneDB;
Query OK, 0 rows affected (0.00 sec)

mysql> insert into dumpdb.dumptb(id,vname) values(1,'zhangsan'),(2,'lisi'),(3,'wangwu');
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> select * from dumpdb.dumptb;
+----+----------+
| id | vname    |
+----+----------+
|  1 | zhangsan |
|  2 | lisi     |
|  3 | wangwu   |
+----+----------+
3 rows in set (0.01 sec)


```

#### 使用mysqldump 备份指定库
```bash
/stonedb56/install/bin/mysqldump  -uroot -p***** -P3306 --skip-opt --master-data=2 --single-transaction --set-gtid-purged=off  --databases dumpdb > /tmp/dumpdb.sql
```

#### 备份指定库的表结构
```
/stonedb56/install/bin/mysqldump  -uroot -p***** -P3306   -d --databases dumpdb > /tmp/dumpdb_table.sql
```
#### 备份指定库的表数据(不包含表结构)
```
/stonedb56/install/bin/mysqldump  -uroot -p***** -P3306 --skip-opt --master-data=2 --single-transaction --set-gtid-purged=off  -t dumpdb > /tmp/dumpdb_table.sql
```


#### 使用mysqldump 备份除系统库（mysql、performation_schema、information_schema）外其他库
```bash
/stonedb56/install/bin/mysql  -uroot -p****** -P3306 -e "show databases;" | grep -Ev "sys|performance_schema|information_schema|Database|test" | xargs /stonedb56/install/bin/mysqldump  -uroot -p****** -P3306 --master-data=1 --skip-opt --databases > /tmp/ig_sysdb.sql
```

***扩展***

使用Mysqldump 备份innodb 导入StoneDB 表小的可以基于上面的mysqldump 备份，大表建议单独备份表结构和数据备份文件，然后使用`sed -i 's/原字符串/新字符串/g' 文件` 命令修改备份文件中的引擎,例如:
```
sed -i 's/ENGINE=InnoDB/ENGINE=STONEDB/g' 文件
```
修改引擎后按照下面恢复方式导入到StoneDB。


### 恢复
#### 数据导入到StoneDB
```bash
/stonedb56/install/bin/mysql  -uroot -p****** -P3306 dumpdb < /tmp/dumpdb.sql
```

