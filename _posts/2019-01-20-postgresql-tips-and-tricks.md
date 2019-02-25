---
layout: post
title: 'Postgresql tips and tricks'
date: 2019-01-20 10:32:12 +0300
tags: [postgresql, psql, sql]
---

Useful things to know when using Postgresql and psql commandline tool.

<!--more-->

## Finding overloaded functions

{% highlight sql %}
SELECT UNNEST(array_agg(f.full)) AS functions
FROM (
    SELECT
        ns.nspname || '.' || proname AS name,
        ns.nspname || '.' || proname || '(' || oidvectortypes(proargtypes) || ')' AS full
    FROM pg_proc
    INNER JOIN pg_namespace ns ON (pg_proc.pronamespace = ns.oid)
    WHERE ns.nspname = 'public' -- set your schema here
    ORDER BY proname
) f
GROUP BY f.name
HAVING COUNT(f.name) > 1;
{% endhighlight %}

### Example output
{% highlight bash %}
                  functions                   
----------------------------------------------
public.sort(integer[], text)
public.sort(integer[])
public.subarray(integer[], integer)
public.subarray(integer[], integer, integer)
{% endhighlight %}

## Editing a function using psql

### List functions
{% highlight text %}
\df
{% endhighlight %}

### Edit a function
{% highlight text %}
\ef function_name
{% endhighlight %}

### Commit edited function
{% highlight text %}
;

or

\g
{% endhighlight %}

## Setting default psql text editor

{% highlight bash %}
export PSQL_EDITOR="/path/to/editor"
{% endhighlight %}

## Pretty json formatting using psql
{% highlight sql %}
\t                      -- disable header
\pset format unaligned  -- change printing format
SELECT jsonb_pretty('{"fied": "value", "array": [1, 2, 3]}'::jsonb); -- use jsonb_pretty function
{% endhighlight %}

### Example output
{% highlight json %}
{
    "fied": "value",
    "array": [
        1,
        2,
        3
    ]
}
{% endhighlight %}


## Writing query results into a file in psql
{% highlight sql %}
SELECT 'test' \g data.txt -- omitting ';' doesn't print the results
{% endhighlight %}

## Fixing out of sync primary key sequence

### Get the name of the sequence
{% highlight sql %}
SELECT pg_get_serial_sequence('table_name', 'sequence_field_name');
{% endhighlight %}

### Fix the sequence
{% highlight sql %}
SELECT setval('sequence_name', (SELECT MAX(sequence_field_name) FROM table_name) + 1);
{% endhighlight %}