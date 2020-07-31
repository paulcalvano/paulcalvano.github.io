---
title: "How Is Server-Timing used on the web?"
date: 2017-07-27T00:00:00+00:00
author: Paul Calvano
layout: post
related_posts:
  - _posts/2019-08-12-chrome-image-lazy-loading-sites-already-using-it-on-week-1.md
  - _posts/2019-05-22-exploring-usage-of-the-console-api.md
  - _posts/2018-08-24-how-many-sites-are-still-using-appcache.md


---

I was curious to see where[ Server-Timing](https://w3c.github.io/server-timing/) was implemented on the web, so I started searching the HTTP Archive for sites using it. Interestingly enough, there were no sites in the HTTP Archive that had Server-Timing response headers before 3/1/2017.  Since then it's usage has been gradually increasing each month.  As of July 2017, there are 72 sites and 352 HTTP responses containing Server-Timing headers.  

Here's a query that I wrote to find a list of sites with Server-Timing headers.  [You can run it on BigQuery here](https://bigquery.cloud.google.com/savedquery/264421607889:45510569d166427594842ad55582f2f2) :
```
    SELECT p.url, p.rank, count(*) as freq
    FROM 
      ( SELECT pageid, url, rank 
        FROM [httparchive:runs.latest_pages] 
      ) p
    JOIN  
      ( SELECT pageid,url,respOtherHeaders 
        FROM [httparchive:runs.latest_requests] 
        WHERE LOWER(respOtherHeaders) CONTAINS "server-timing"
      ) r
    ON p.pageid = r.pageid
    GROUP BY p.url, p.rank
    ORDER BY freq DESC
```

![](/assets/img/blog/how-is-server-timing-used-on-the-web/1.png){:loading="lazy"}

Next I was curious to see the actual Server-Timing headers to understand how they are being used.   I wrote this query to strip out the server-timing headers from the respOtherHeaders column.   The query has a lot of string manipulations to extract the text. I've added some comments to the query to help make it easier to follow - 
```
    SELECT HOST(url), status, mimeType,
    IF (
        -- In respOtherHeaders, remove everything to the left of the server-timing header.   Is there a ; in the remaining string?
        RIGHT(respOtherHeaders,LENGTH(respOtherHeaders)-INSTR(LOWER(respOtherHeaders),"server-timing")+1) CONTAINS ";",
        -- Then strip out everything to the left of the server-timing header and everything to the right of the ";" character.
        LEFT(
            RIGHT(respOtherHeaders,
                LENGTH(respOtherHeaders)-INSTR(LOWER(respOtherHeaders),"server-timing")+1),
            INSTR(
                RIGHT(respOtherHeaders,
                    LENGTH(respOtherHeaders)-INSTR(LOWER(respOtherHeaders),
                    "server-timing")+1),
                ";")
            ), 
        -- If there are no ; chars, then the respOtherHeaders column ends with server-timing.  Remove everything to the left of server-timing
        RIGHT(respOtherHeaders,
            LENGTH(respOtherHeaders)-INSTR(LOWER(respOtherHeaders),
            "server-timing")+1)
    ) as ServerTimingHeader
    FROM [httparchive:runs.latest_requests] 
    WHERE LOWER(respOtherHeaders) CONTAINS "server-timing"
```
[You can see it on BigQuery here](https://bigquery.cloud.google.com/savedquery/264421607889:a85d7b23c59849e0bd2827c89a9c0c64) 

![](/assets/img/blog/how-is-server-timing-used-on-the-web/2.png){:loading="lazy"}

Looking at the Server-Timing headers, I can see 3 different types of data being sent in these headers:

**Timing Metrics**
(such as application time, gateway time, image resize time, etc)
	
	Server-Timing = app=133.99219512939;
	Server-Timing = gateway-time=25;
	Server-Timing = resizer=0.015;
	server-timing = view=2.188, db=0.000, total=61.743, x-runtime = 0.069661, x-ua-compatible = ie=edge,chrome=1, x-frame-options = SAMEORIGIN
	Server-Timing = app=53.393840789795;

**Caching Information**

	server-timing = hit, cf-cache-status = HIT
	server-timing = cold, cf-cache-status = MISS
	server-timing = hit, cf-cache-status = HIT
	server-timing = cold, cf-cache-status = MISS

**Informational**

	server-timing = ibs_5318e9209631=1
	server-timing = ibs_1964fd84c9cea3=1
	server-timing = ibs_1880bf4b0f6311=1

Looking at all of the data in the 7/1/2017 tables, I can see that the current examples in the wild are split between these 3 types of data, although the caching ones are mostly on images, the informational ones are set on JS and responses w/ no content (ie, 204s -likely beacon responses), and the actual timing metrics are spread across a wide range of content.
![](/assets/img/blog/how-is-server-timing-used-on-the-web/3.png){:loading="lazy"}

It'll be interesting to see how this evolves as Server-Timing adoption increases over time.

_Originally published at <https://discuss.httparchive.org/t/how-is-server-timing-used-on-the-web/1034>_