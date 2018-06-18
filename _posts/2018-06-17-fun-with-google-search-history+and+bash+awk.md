---
layout: post
title:  "Fun with Google search history (and bash + awk)"
date:   2018-06-17 13:32:12 +0300
tags: [google, history, bash, awk]
---

Years ago Google used to provide interesting information about your own search history (assuming you are logged into Google when googling). For example, you could see how many searches you've made and what are the top sites your searches lead you to. But then they updated their activity page for the worse.

But let's see how we can search and process the history data.

1. Download your history ([https://takeout.google.com](https://takeout.google.com)). Generate an archive file, and extract the search history html-file.

2. Use these easy bash one-liners

<!--more-->

**Note: times and dates in my data are in finnish format, so these may need some updating if you want to try them.**

## Convert searches to easier to process format

{% highlight bash %}
> cat MyActivity.html | egrep -o 'search\?q=([^"]+)">[^>]+</a><br>[^ ]+ klo [^<]+'  | cut -d ">" -f2,4 | sed -r 's/<\/a>/|/g' | sed -r 's/ klo / /g' | sed -r 's/&quot;/"/g' > search-data
{% endhighlight %}

## Convert opened urls to easier to process format
{% highlight bash %}
> cat MyActivity.html | egrep -oi 'url\?q=([^"]+)'  | cut -d '=' -f2 | cut -d '&' -f1 > url-data
{% endhighlight %}

## How many searches I've made

{% highlight bash %}
> cat search-data | wc -l
> 69623
{% endhighlight %}

Yes... I like to google alot.

## Top 10 searches

{% highlight bash %}
> cat search-data | cut -d '|' -f1 | sort | uniq -c | sort -rn | head
>   92 poe unique
    92 poe affix
    84 sää
    35 poe aura
    28 poe unique jewel
    28 a
    25 poe jewel
    23 poe unique flask
    23 google jquery
    22 poe poison
{% endhighlight %}

## Top 10 sites

{% highlight bash %}
> cat url-data |  grep -Poi 'https?://(www\.)?(fi\.|en\.)?(m\.)?\K([^/]+)' | sort | uniq -c | sort -nr | head
>  7036 stackoverflow.com
   5173 pathofexile.gamepedia.com
   3246 wikipedia.org
   2751 reddit.com
   1590 github.com
   1581 pathofexile.com
   1544 imdb.com
    849 warframe.wikia.com
    741 leagueoflegends.wikia.com
    725 gamefaqs.com
{% endhighlight %}

## Top 10 search words, 3 characters or more

{% highlight bash %}
> cat search-data | cut -d '|' -f1 | grep -o '[[:alpha:]]*' | awk 'length($0)>=3' | sort | uniq -c | sort -nr | head
>  6546 poe
   1297 node
   1137 java
    998 the
    785 lol
    768 not
    709 warframe
    699 how
    494 google
    487 react
{% endhighlight bash %}


## Top 10 search words, 3 characters or more and without stop-words

{% highlight bash %}
> file=$(mktemp);wget -q https://raw.githubusercontent.com/Alir3z4/stop-words/master/english.txt -O $file && cat search-data | cut -d '|' -f1 | grep -o '[[:alpha:]\-]*'| grep -v -x -f "$file" | awk 'length($0)>=3' | sort | uniq -c | sort -nr | head
>  6546 poe
   1249 node
   1137 java
    785 lol
    709 warframe
    491 google
    462 react
    445 jquery
    422 terraria
    386 canvas
{% endhighlight bash %}

## Top 10 search words, 10 characters or more

{% highlight bash %}
> cat search-data | cut -d '|' -f1 | grep -o '[[:alpha:]]*' | awk 'length($0)>=10' | sort | uniq -c | sort -nr | head
>   376 javascript
    356 stackoverflow
    223 pakkotoisto
    205 borderlands
     80 postgresql
     78 browserify
     71 handlebars
     60 calculator
     57 controller
     56 opiskelija
{% endhighlight bash %}


## Top 10 word pairs, without stop-words

{% highlight bash %}
> file=$(mktemp);wget -q https://raw.githubusercontent.com/Alir3z4/stop-words/master/english.txt -O $file && cat search-data | cut -d '|' -f1 | grep -o '[[:alpha:]\-]*' | grep -v -x -f "$file" | awk 'NR>1{print l " " $0}{l=$1}' | sort | uniq -c | sort -nr | head
>   307 poe unique
    173 react native
    162 java spring
    160 lbs kg
    154 path exile
    153 helsingin yliopisto
    143 google maps
    112 socket io
     99 new vegas
     94 poe affix

{% endhighlight bash %}

## Top 10 word trios 

{% highlight bash %}
> cat search-data | cut -d '|' -f1 | awk '{print $0 "\n#"}'| grep -oP '[[:alpha:]\-#]*' | awk 'NR>2{print s " " l " " $0}{s=l}{l=$1}' | sort | sed 's/\# //g' | sed 's/ \#//g'| awk 'NF==3' | uniq -c| sort -nr | head
>   159 lbs to kg
    154 path of exile
     58 poe how much
     44 how to get
     40 fallout new vegas
     38 battle angel alita
     32 pounds to eur
     32 google maps api
     30 dollars to eur
     28 poe unique jewel

{% endhighlight bash %}

## Top 10 search times, rounded to nearest 30 minutes

{% highlight bash %}
> cat search-data | cut -d '|' -f2 | sed -r 's/([0-9]+)\.([0-9]+)\.([0-9]+) ([0-9]+)\.([0-9]+)\.([0-9]+)/\4-\5/g' | awk '{split($0, d, "-");hour=d[1];min=int((d[2]/30) + 0.5)*30; if(min==60) { hour++; min=0 };  printf("%02d.%02d\n", hour, min)}' | sort | uniq -c  | sort -nr | head
>   2568 15.30
    2464 14.30
    2461 16.00
    2460 19.00
    2448 14.00
    2406 13.00
    2399 20.30
    2391 13.30
    2379 18.30
    2357 12.30
{% endhighlight bash %}


## Bottom 10 search times, rounded to nearest 30 minutes

{% highlight bash %}
> cat search-data | cut -d '|' -f2 | sed -r 's/([0-9]+)\.([0-9]+)\.([0-9]+) ([0-9]+)\.([0-9]+)\.([0-9]+)/\4-\5/g' | awk '{split($0, d, "-");hour=d[1];min=int((d[2]/30) + 0.5)*30; if(min==60) { hour++; min=0 };  printf("%02d.%02d\n", hour, min)}' | sort | uniq -c  | sort -n | head
>   117 05.00
    118 05.30
    127 06.30
    146 06.00
    175 07.00
    205 04.00
    216 04.30
    287 07.30
    312 03.30
    366 08.00
{% endhighlight bash %}

## Searches by year

{% highlight bash %}
> cat search-data | cut -d '|' -f2 | cut -d '.' -f3 | cut -d ' ' -f1 | sort | uniq -c
>  5298 2013
   9545 2014
  11545 2015
  17074 2016
  17346 2017
   8812 2018
{% endhighlight bash %}

## Searches by month

{% highlight bash %}
> cat search-data | cut -d '|' -f2 | cut -d '.' -f2 | sort | uniq -c | sort  -k2n
>  7238 1
   5244 2
   6161 3
   6481 4
   5504 5
   3722 6
   3310 7
   4456 8
   5985 9
   6903 10
   7212 11
   7404 12
{% endhighlight bash %}

## Searches by month as percents

{% highlight bash %}
> cat search-data | cut -d '|' -f2 | cut -d '.' -f2 | sort | uniq -c | sort  -k2n  | awk 'BEGIN {t=0} {v[NR]=$1;t+=$1} END {for(i=1;i<=NR;i++) {printf("%s %.1f%\n", i,  (v[i] / t)*100)}}'
>  1 10.4%
   2 7.5%
   3 8.8%
   4 9.3%
   5 7.9%
   6 5.3%
   7 4.8%
   8 6.4%
   9 8.6%
   10 9.9%
   11 10.4%
   12 10.6%
{% endhighlight bash %}
