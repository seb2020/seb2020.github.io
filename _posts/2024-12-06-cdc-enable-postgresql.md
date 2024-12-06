---
layout: post
title: Enable CDC for PostgreSQL
date: 2024-12-06 13:15:00
description: Enable CDC for PostgreSQL for capturing data
tags: debezium, cdc
categories: database
featured: false
toc:
  sidebar: left
---

## Enable CDC for PostgreSQL with Patroni

```bash
# Check cluster
/usr/local/bin/patronictl -c /etc/patroni/patroni.yml list
 
# Edit configuration
/usr/local/bin/patronictl -c /etc/patroni/patroni.yml edit-config
```

Add a permanent slot:

```bash
slots:
  permanent_logical_slot_<db name>:
    database: <db name>
    plugin: pgoutput
    type: logical
```

Exit text editor and validate change.

```bash
# Check if a reboot is required
/usr/local/bin/patronictl -c /etc/patroni/patroni.yml list
 
# Restart the cluster
/usr/local/bin/patronictl -c /etc/patroni/patroni.yml restart <cluster-name>
 
# Check cluster
/usr/local/bin/patronictl -c /etc/patroni/patroni.yml list
 
# Check log
tail -f /data/postgres/pg_log/postgresql-*.log
```

Create a PostgreSQL user for accessing the database:

```sql
GRANT CREATE ON DATABASE <db> TO debezium;
 
-- connect to correct database
\c <db name> --switch to correct db
GRANT ALL ON SCHEMA public TO debezium_group;
 
GRANT debezium_group TO <current object owner>;
```

Validate the replication slots and publication:

```sql
-- check output of the command
\c postgres
select * from pg_replication_slots;
 
\c <db name>
select * from pg_publication;
select * from pg_publication_tables;
```