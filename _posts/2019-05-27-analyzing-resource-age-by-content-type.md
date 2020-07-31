---
title: "Analyzing Resource Age by Content Type"
date: 2019-05-27T00:00:00+00:00
author: Paul Calvano
layout: post
related_posts:
  - _posts/2019-03-25-what-percentage-of-third-party-content-is-cacheable.md
  - _posts/2018-03-14-http-heuristic-caching-missing-cache-control-and-expires-headers-explained.md
  - _posts/2018-01-07-cache-control-immutable-a-year-later.md
  
---

Have you ever wondered how old web content is? One could probably assume that HTML is going to be newer than images or JavaScript, but is that really true?  How does resource age vary by content type or by first or third party content? And why does this matter?

Resource age is an important heuristic in developing a good caching strategy. It applies both to CDN edge caching as well as browser caching. When I discuss caching strategies with my clients, I frequently ask: "How often are you updating these assets?" and "what is their content sensitivity?". For example, if a hero image is going to be modified infrequently - then cache it with a very long TTL. If you expect a JavaScript resource to change frequently, then version it and cache it with a long TTL or cache it with a shorter TTL. 

I was recently asked whether script resources are changed frequently, or if they persist for  a long time. I imagined that it would vary greatly by 1st and 3rd party content, so I decided to look at both.I  used the difference between the `Date` and `Last-Modified` response headers to calculate the age.  You might be wondering why I didn't use the aptly named `Age` header for this analysis, and that would be because it's only present on 14% of HTTP responses. Comparatively, `Last-Modified` is present on 72% of HTTP responses. 

```
SELECT ROUND(SUM(IF(resp_date <> "",1,0)) / count(*),2) date_pct,
       ROUND(SUM(IF(resp_last_modified <> "",1,0)) / count(*),2) lastmodified_pct,
       ROUND(SUM(IF(resp_age  <> "",1,0)) / count(*),2) age_pct,
       count(*) requests
FROM `httparchive.summary_requests.2019_04_01_mobile` r
```
![377x55](/assets/img/blog/analyzing-resource-age-by-content-type/1.png){:loading="lazy"}

The query below groups [resources by 1st or 3rd party](https://discuss.httparchive.org/t/what-is-the-distribution-of-1st-party-vs-3rd-party-resources/100), content type and the relative age of the resource in weeks. A user defined function converts the timestamp to epoch seconds and throws out any timestamps after the year 2050 in order to prevent int64 overflows from erroneous timestamps in the data. And finally, we subtract the time value of Last-Modified from Date and calculate the age in weeks.

```
CREATE TEMP FUNCTION dateConversion(ts STRING)
RETURNS STRING
LANGUAGE js AS """
  epoch = Math.floor(Date.parse(ts)/1000);
  if (epoch <= 2558874097 && epoch > 0) 
  return  epoch
""";

SELECT type,
       IF (STRPOS(NET.HOST(r.url),REGEXP_EXTRACT(NET.REG_DOMAIN(p.url), r'([\w-]+)'))>0, 1, 3) AS party,
          ROUND(TIMESTAMP_DIFF(
              TIMESTAMP_SECONDS(SAFE_CAST(dateConversion(resp_date) as INT64)), 
              TIMESTAMP_SECONDS(SAFE_CAST(dateConversion(resp_last_modified) as INT64)), 
              DAY)/7) age_weeks,
          count(*) requests
FROM `httparchive.summary_requests.2019_04_01_mobile` r
INNER JOIN `httparchive.summary_pages.2019_04_01_mobile` p
ON r.pageid = p.pageid
WHERE resp_last_modified <> "" and resp_date <> ""
GROUP BY type, party, age_weeks`
```

Graphically, these results look like this. Note that because there are 55 bars in this stacked chart, there is some overlap in the legend. I used a 10 color palette to display this, so that you can distinguish the week based on it's position in the chart.  For example, the yellow bars to the left are resources < 1 week old.  The orange bar all the way to the right is > 2 years old. 
![690x366](/assets/img/blog/analyzing-resource-age-by-content-type/2.jpeg){:loading="lazy"} 

There are some interesting observations in this data:
* With the exception of HTML, third party content has a smaller resource age compared to first party content. One can only hope that HTML is being cached somewhere... 
* Audio and Video resources tend to be older and more likely to be cacheable. Even more so with 1st party videos.
* Some of the longest lived first party content on the web are the traditionally cacheable objects - images, scripts, css, web fonts, audio and video. 
* There is a significant gap in the 1st vs 3rd party resource age of CSS and web fonts.  95% of first party fonts are older than 1 week compared to 50% of 3rd party fonts which are less than 1 week old! *This makes a strong case for self hosting web fonts!*   

Since a question about scripts was what led me down this path, I thought it would be interesting to expand the bar charts for 1st vs 3rd party scripts.  35% of third party script resources and  8% of first party scripts are less than 1 week old. This explains why 3rd party script resources are less likely to be cached for long periods.   

![690x404](/assets/img/blog/analyzing-resource-age-by-content-type/3.jpg){:loading="lazy"} 

Resource age is an important heuristic in developing a good caching strategy.  How frequently do certain resources change? How sensitive is your application to these changes. Would you be able to cache longer if you serve a third party resource as a first party? And how much non-cacheable third party content should your users have to load?  Answering these questions about your own site's data will help you tweak your caching policies, achieve higher offload and faster page load times.

_Originally published at <https://discuss.httparchive.org/t/analyzing-resource-age-by-content-type/1659>_