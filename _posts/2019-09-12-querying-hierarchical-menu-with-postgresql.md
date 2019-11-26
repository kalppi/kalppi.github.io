---
layout: post
title:  'Querying a hierarchical menu with Postgresql'
date:   2019-09-12 12:00:00 +0300
tags: [sql, cte, psql, postgresql]
---

How to use recursive CTE's (Common Table Expressions) to get each menu item's ancestors and descendants, and also order the items.

<!--more-->

First, have a table. Every item in the menu needs an id, text to display, parent item's id, and an order number relative to all the other items under the same parent.

{% highlight sql %}
CREATE TABLE example_menu (
	id SERIAL PRIMARY KEY,
	title TEXT NOT NULL,
	parent_id INT REFERENCES example_menu (id) NULL,
	"order" INT NOT NULL
);
{% endhighlight %}

Then, let's add some test data.

{% highlight sql %}
INSERT INTO example_menu (title, parent_id, "order") VALUES
('A', NULL, 1),
('B', NULL, 2),
('C', NULL, 3),
('D', NULL, 4),
('BA', 2, 1),
('BB', 2, 2),
('BBA', 6, 1),
('CA', 3, 1),
('CAA', 8, 1),
('CAAA', 9, 1);
{% endhighlight %}

{% highlight bash %}
 id | title | parent_id | order 
----+-------+-----------+-------
  1 | A     |           |     1
  2 | B     |           |     2
  3 | C     |           |     3
  4 | D     |           |     4
  5 | BA    |         2 |     1
  6 | BB    |         2 |     2
  7 | BBA   |         6 |     1
  8 | CA    |         3 |     1
  9 | CAA   |         8 |     1
 10 | CAAA  |         9 |     1
(10 rows)
{% endhighlight %}

Next, let's query. There might be a cleaner and faster way to do this, and feel free to tell me if you figure it out.

The query can be simplified quite a lot, depending what data you need. For example, `roots` and `descendants` CTE's can be completely removed if you don't need to know the descendants, but knowing them can be very useful in a case when you want to mark all the parent items when you only know the child item's id.

{% highlight sql %}

WITH RECURSIVE data AS (
	SELECT
		m.id, m.title, m.parent_id, m.order,
		0 AS depth, ARRAY[m.order]::integer[] AS order_path,
		ARRAY[]::integer[] AS ancestors, m.id AS root
	FROM example_menu m
	WHERE m.parent_id IS NULL

	UNION ALL

	SELECT m.id, m.title, m.parent_id, m.order,
		d.depth + 1, d.order_path || m.order, d.ancestors || m.parent_id, d.root
	FROM data d
	INNER JOIN example_menu m ON m.parent_id = d.id
), roots AS (
	SELECT c.id, c.id AS root
	FROM example_menu c

	UNION ALL 

	SELECT c.id, r.root
	FROM roots r
	INNER JOIN example_menu c ON c.parent_id = r.id
), descendants AS (
	SELECT DISTINCT m.id, ARRAY_REMOVE(ARRAY_AGG(r.id), m.id) AS descendants
	FROM roots r
	INNER JOIN example_menu m ON m.id = r.root
	GROUP BY r.root, m.id
)
SELECT
	d.id, d.title, d.parent_id, d.order, d.depth, d.order_path, d.ancestors, de.descendants, d.root,
	CASE WHEN d.depth = 0 THEN '' ELSE REPEAT('--', d.depth) || ' ' END || d.title AS visual
FROM data d
LEFT JOIN descendants de ON de.id = d.id
ORDER BY d.order_path
{% endhighlight %}

{% highlight bash %}
 id | title | parent_id | order | depth | order_path | ancestors | descendants | root |   visual    
----+-------+-----------+-------+-------+------------+-----------+-------------+------+-------------
  1 | A     |           |     1 |     0 | {1}        | {}        | {}          |    1 | A
  2 | B     |           |     2 |     0 | {2}        | {}        | {6,5,7}     |    2 | B
  5 | BA    |         2 |     1 |     1 | {2,1}      | {2}       | {}          |    2 | -- BA
  6 | BB    |         2 |     2 |     1 | {2,2}      | {2}       | {7}         |    2 | -- BB
  7 | BBA   |         6 |     1 |     2 | {2,2,1}    | {2,6}     | {}          |    2 | ---- BBA
  3 | C     |           |     3 |     0 | {3}        | {}        | {8,9,10}    |    3 | C
  8 | CA    |         3 |     1 |     1 | {3,1}      | {3}       | {9,10}      |    3 | -- CA
  9 | CAA   |         8 |     1 |     2 | {3,1,1}    | {3,8}     | {10}        |    3 | ---- CAA
 10 | CAAA  |         9 |     1 |     3 | {3,1,1,1}  | {3,8,9}   | {}          |    3 | ------ CAAA
  4 | D     |           |     4 |     0 | {4}        | {}        | {}          |    4 | D
(10 rows)
{% endhighlight %}

