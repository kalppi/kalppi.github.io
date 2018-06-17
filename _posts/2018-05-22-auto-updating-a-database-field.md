---
layout: post
title:  "Auto-updating a database field using Postgresql"
date:   2018-05-22 10:32:12 +0300
tags: [sql, postgresql, trigger]
---

If you have a database table consisting of all your blog posts, you can create a trigger that updates a field automatically when you update your blog's title or content.

<!--more-->

{% highlight sql %}
CREATE TABLE blog_article (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255),
    content TEXT,
    date_published TIMESTAMPTZ DEFAULT now(),
    date_updated TIMESTAMPTZ,
    published BOOLEAN DEFAULT FALSE
);

CREATE OR REPLACE FUNCTION set_date_updated()   
RETURNS TRIGGER AS $$
BEGIN
    NEW.date_updated = now();
    RETURN NEW; 
END;
$$ language 'plpgsql';

CREATE TRIGGER update_date_updated BEFORE UPDATE OF title, content ON blog_article
    FOR EACH ROW
        WHEN(OLD.content IS DISTINCT FROM NEW.content OR OLD.title IS DISTINCT FROM NEW.title)
            EXECUTE PROCEDURE set_date_updated();
{% endhighlight %}