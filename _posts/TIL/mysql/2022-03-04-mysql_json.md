---
title:  "[MySql] Querying Json data type"
date: 2022-02-08
excerpt: ""
tags: [til, mysql]
categories: [til/mysql]
---

JSON data type is not exists in oracle, so I have no idea to query json data in mysql. In this article, we will look at how to query JSON column in MySQL.

[How to Query JSON column in MySQL](https://ubiq.co/database-blog/how-to-query-json-column-in-mysql/#:~:text=How%20to%20Retrieve%20data%20from,will%20fetch%20value%20without%20quotes.&text=As%20you%20can%20see%20%2D%3E%3E,returns%20values%20as%20they%20are.)

`{field_name} -> '$.name'` or `{field_name} --> '$.name'` can extract attribute in json data

```
mysql> select id,
       details->>'$.name' as browser_str,
       details->'$.name' as browser_name
       from users;
+----+--------------+--------------+
| id | browser_str  | browser_name |
+----+--------------+--------------+
|  1 | "Safari"     |  Safari      |
|  2 | "Chrome"     |  Chrome      |
|  3 | "Firefox"    |  Firefox     |
+----+--------------+--------------+
```
