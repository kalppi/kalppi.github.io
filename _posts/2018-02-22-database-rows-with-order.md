---
layout: post
title:  "Database rows with order, using Postgresql"
date:   2017-02-05 10:32:12 +0300
tags: [postgresql, sql]
---

There are times when database rows need to have a field that notates the order in which they are to be presented, for example menu items. With these Postgresql-functions you can easily move them up or down.

<!--more-->


{% highlight sql %}
CREATE TABLE menu_items (
    id SERIAL PRIMARY KEY,
    title VARCHAR(30),
    "order" INT
);
{% endhighlight %}

Set order number automatically when inserting a new row:

{% highlight sql %}
CREATE OR REPLACE FUNCTION set_order() RETURNS TRIGGER AS $$
DECLARE
    val INTEGER;
BEGIN
    IF NEW."order" IS NULL THEN
        EXECUTE 'SELECT COALESCE(MAX("order") + 1, 1) AS "order" FROM ' || quote_ident(TG_ARGV[0]) INTO val;
        NEW."order" := val;
    END IF;
    
    RETURN NEW;
END
$$ LANGUAGE plpgsql;

DROP TRIGGER IF EXISTS set_order_trigger ON "menu_items";
CREATE TRIGGER set_order_trigger BEFORE INSERT ON "menu_items"
    FOR EACH ROW
        EXECUTE PROCEDURE set_order("menu_items");
{% endhighlight %}

Functions to move the items:

{% highlight sql %}
CREATE OR REPLACE FUNCTION move_up(tablename TEXT, moveid INTEGER) RETURNS VOID AS $$
BEGIN
    EXECUTE 'UPDATE ' || quote_ident(tablename) || ' c
    SET "order" =
        CASE c."order"
            WHEN x."order" THEN x."order" - 1
            WHEN x."order" - 1 THEN x."order"
        END
    FROM (SELECT id, "order"
        FROM ' || quote_ident(tablename) || '
        WHERE id = $1
        AND "order" > 1) x
    WHERE c."order" = x."order" OR c."order" = x."order" - 1' USING moveid;
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION move_down(tablename TEXT, moveid INTEGER) RETURNS VOID AS $$
BEGIN
    EXECUTE 'UPDATE ' || quote_ident(tablename) || ' c
    SET "order" =
        CASE c."order"
            WHEN x."order" THEN x."order" + 1
            WHEN x."order" + 1 THEN x."order"
        END
    FROM (SELECT id, "order"
        FROM ' || quote_ident(tablename) || '
        WHERE id = $1
        AND "order" < (SELECT MAX("order") FROM ' || quote_ident(tablename) || ')) x
    WHERE c."order" = x."order" OR c."order" = x."order" + 1' USING moveid;
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION move_to(tablename TEXT, moveid INTEGER, toposition INTEGER) RETURNS VOID AS $$
DECLARE
    max INTEGER;
BEGIN
    EXECUTE 'SELECT MAX("order") FROM ' || quote_ident(tablename) INTO max;

    IF toposition >= 1 AND toposition <= max THEN
        EXECUTE 'UPDATE ' || quote_ident(tablename) || ' c
        SET "order" = CASE
            WHEN c."order" = x."order" THEN $2
            WHEN x."order" < $2 THEN c."order" - 1
            ELSE c."order" + 1
        END
        FROM (
            SELECT "order"
            FROM ' || quote_ident(tablename) || '
            WHERE id = $1
        ) x
        WHERE
            CASE WHEN x."order" < 2 THEN
                c."order" >= x."order" AND c."order" <= $2
            ELSE
                c."order" <= x."order" AND c."order" >= $2
            END' USING moveid, toposition;
    END IF;
END;
$$ LANGUAGE plpgsql;
{% endhighlight %}