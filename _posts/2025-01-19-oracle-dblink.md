---
layout: post
title: How to create an Oracle DB link ?
date: 2025-01-19 15:25:00
description: How to create an Oracle DB link ?
tags: oracle, tips
categories: database
featured: false
toc:
  sidebar: left
---

## How to create an Oracle DB link ?

From a database X, we wish to display a view located in another database Y.

In the Y database:

```sql
CREATE OR REPLACE FORCE EDITIONABLE VIEW "MY_VIEW" ("CustomerName", "ContactName") AS 
  SELECT 
    C.CUSTOMERNAME AS CustomerName,
    C.CONTACTNAME AS ContactName
  FROM
    CUSTOMER C;
```

In the X database:

```sql
CREATE PUBLIC DATABASE LINK MY_DB_LINK_TO_Y CONNECT TO <remoteuser> IDENTIFIED BY "<password>" USING 'db_y.domain.lan:1521/<SID>';
```

Access the remote view through the DB link :

```sql
select * from <myschema>."MY_VIEW" @MY_DB_LINK_TO_Y;
```