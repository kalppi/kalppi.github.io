---
layout: post
title:  "Fun with promises"
date:   2017-10-10 10:32:12 +0300
tags: [javascript, promise]
---

A few tips and tricks when you are working with promises.

<!--more-->

{% highlight js %}
// helper functions
const reject = v => () => Promise.reject(v);
const resolve = v => () => Promise.resolve(v);
{% endhighlight %}

## Get the first resolved value from functions

{% highlight js %}

const onlyFirst = ps => {
    return new Promise((resolve, reject) => {
        let p = ps.shift();

        if(!p) resolve(null);
        else p.call().then(v => resolve(v), () => resolve(onlyFirst(ps)));
    });
};

onlyFirst([reject(1), resolve(2), resolve(3), reject(4)]).then((a) => {
    console.log(a); // 2
});

{% endhighlight %}

## Get all resolved values from functions, and don't care if some of then are rejected

{% highlight js %}
const ps = [resolve(1), reject(2), resolve(3)];

const unrejectable = ps =>
    ps.map(p => new Promise((resolve, reject) =>  p.call().then(resolve, resolve.bind(null, null))));

Promise.all(unrejectable(ps)).then(console.log); // [1, null, 3]
{% endhighlight %}
