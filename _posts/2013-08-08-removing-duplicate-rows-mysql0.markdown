---
layout: post
title:  "Removing duplicate rows in MySQL"
date:   2013-08-08 11:14:55 +0000
categories: dev mysql
---

Its often the case that you find application issues at the stage they become problematic. MySQL seems to be one of the most common ones of these for me whether it be something as simple as a lack of an index or something far more fundamental with your schema. A recent issue I came across had been caused by some far from perfect code associated with updating of elements within an ecommerce CMS using an API connection. A table that should realistically have no more than 10,000 rows had grown to over 4 million. This had caused an almost inevitable slowdown with all interactions with this table. Looking at the table there was a huge amount of data duplication. The data tended to be duplicated on all but the primary key, the question was how to remove this duplicate data without having to run a long running PHP or Shell script against the production database. The answer was surprisingly simple and one of those times where a simple SQL command is all thats needed.

{% highlight SQL %}
ALTER IGNORE TABLE `table_with_duplicates`
ADD UNIQUE INDEX `remove_duplicates` (`col_1`, `col_2`,  `col_3`);
{% endhighlight %}

An explanation of how this works can be seen on the MySQL site:

>IGNORE is a MySQL extension to standard SQL. It controls how ALTER TABLE works if there are duplicates on unique keys in the new table or if warnings occur when strict mode is enabled. If IGNORE is not specified, the copy is aborted and rolled back if duplicate-key errors occur. If IGNORE is specified, only the first row is used of rows with duplicates on a unique key, The other conflicting rows are deleted. Incorrect values are truncated to the closest matching acceptable value
