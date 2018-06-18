---
layout: post
title:  "Dangers of JavaScript's type-converting comparisons i.e. why you should always use strict comparison (===)"
date:   2018-03-16 13:32:12 +0300
tags: [DANGER DANGER, javascript, design choises, security]
---

When using JavaScript, you should pretty much never use the abstract equality comparison operator (`==`). Most of the time, it doesn't really matter. But it's better to learn good habits for the times when it does indeed matter.

<!--more-->

Let's imagine you are building your fancy new app's login component, and it looks something like this:

{% highlight js %}
const user = await userService.findByUsername(request.username);

if(user !== null) {
    if(userService.passwordMatches(user, request.password)) {
        letUserIn();
    } else {
        keepUserOut();
    }
}
{% endhighlight %}

It seems very basic, so what's the problem? You've forgotten that your `passwordMatches` method is asynchronous and returns a promise, and you've forgotten to use the await-keyword, so the if-statement will always evaluate to true. If you don't have good tests (well, even not-so-good tests would probaby suffice), you might not notice that you are letting anyone in with just knowing any username.

A lot of headache can be prevented, if you just use strict comparison:

{% highlight js %}
if(userService.passwordMatches(user, request.password) === true) {
{% endhighlight %}

Now the returned promise is compared strictly against `true`, and you'll eventually notice the missing `await` keyword because you can't seem to be able to login. Then you can fix your code:

{% highlight js %}
if(await userService.passwordMatches(user, request.password) === true) {
{% endhighlight %}

This totally didn't happen. Not to me at least.