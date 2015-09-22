---
layout: post
title:  "Simple SQL injection examples using MySQL"
date:   2015-09-22 18:00:00
categories:
tags:
- mysql
- sql-injection
excerpt: Basic working examples of SQL injection using MySQL.
---

I was describing SQL injection to some trainees today and couldn't find any simple examples which used a variable to replicate a "slightly" realistic scenario (those quotes can be tricky!). So, here are some basic ones. Uncomment <code>@val</code> as necessary!


{% highlight sql %}

use personDb;

-- set @val = 'john';
-- set @val = 'john"; -- '; -- remove any further criteria checking
-- set @val = 'john" or id = 3 -- '; -- guess column names, error means column doesn't exist
-- set @val = 'john" or "1" = "1'; -- get all records

set @dynamic_sql=CONCAT('select * from person where name = "', @val, '"');

-- select @dynamic_sql;

PREPARE statement FROM @dynamic_sql; 
EXECUTE statement; 
DEALLOCATE PREPARE statement;

{% endhighlight sql %}

As I'm using a prepared statement here, it's not possible to end the statement and do other more serious operations like <code>delete from person;</code> but still enough to do some damange!
