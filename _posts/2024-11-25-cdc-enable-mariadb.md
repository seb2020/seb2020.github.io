---
layout: post
title: Enable CDC for MariaDB
date: 2024-11-25 11:15:00
description: Enable CDC for MariaDB for capturing data
tags: debezium, cdc
categories: database
featured: false
toc:
  sidebar: left
---

## Enable CDC for MariaDB with Galera

Edit the file `/etc/my.cnf.d/server.cnf` and add the following:

```bash
[mysqld]
server-id         = 1   # Querying variable is called server_id, e.g. SELECT variable_value FROM information_schema.global_variables WHERE variable_name='server_id';
log_bin                     = mariadb-bin
binlog_format               = ROW
binlog_row_image            = FULL
binlog_expire_logs_seconds  = 7200
```

Restart the MariaDB service on each node.

Validate the new configuration:

```bash
MariaDB [(none)]> SHOW VARIABLES LIKE '%log_bin%';
+---------------------------------+-------------------------------+
| Variable_name                   | Value                         |
+---------------------------------+-------------------------------+
| log_bin                         | ON                            |
| log_bin_basename                | /data/mysql/mariadb-bin       |
| log_bin_compress                | OFF                           |
| log_bin_compress_min_len        | 256                           |
| log_bin_index                   | /data/mysql/mariadb-bin.index |
| log_bin_trust_function_creators | OFF                           |
| sql_log_bin                     | ON                            |
+---------------------------------+-------------------------------+
7 rows in set (0.001 sec)
```

Create a MariaDB user for accessing the database:

```sql
MariaDB [(none)]> CREATE USER 'debeziumint'@'%' IDENTIFIED BY 'password';
MariaDB [(none)]> show grants for 'debeziumint'@'%';
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Grants for debeziumint@%                                                                                                                                               |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, BINLOG MONITOR ON *.* TO `debeziumint`@`%` IDENTIFIED BY PASSWORD '*XXXXX...' |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.000 sec)
```