---
title: 'Cache Control Immutable &#8211; A Year Later'
date: 2018-01-07T15:25:03+00:00
author: Paul Calvano
layout: post
related_posts:
  - _posts/2019-03-25-what-percentage-of-third-party-content-is-cacheable.md
  - _posts/2018-08-24-how-many-sites-are-still-using-appcache.md
  - _posts/2018-03-14-http-heuristic-caching-missing-cache-control-and-expires-headers-explained.md

---
In January 2017, [Facebook wrote about a new Cache-Control directive](https://code.facebook.com/posts/557147474482256/this-browser-tweak-saved-60-of-requests-to-facebook) &#8211; immutable &#8211; which was designed to tell supported browsers not to attempt to revalidate an object on a normal reload during it&#8217;s freshness lifetime. [Firefox 49](https://hacks.mozilla.org/2017/01/using-immutable-caching-to-speed-up-the-web/) implemented it, while  [Chrome went ahead with a different approach](https://blog.chromium.org/2017/01/reload-reloaded-faster-and-leaner-page_26.html) by changing the behavior of the reload button. Additionally it seems that [WebKit has also implemented the immutable directive](https://bugs.webkit.org/show_bug.cgi?id=167497) since then.

So it&#8217;s been a year &#8211; let&#8217;s see where Cache-Control immutable is being used in the wild!


**How Widely Used is Cache-Control: Immutable Today?**  

To determine this, I ran a simple query that counts the number of pages and objects in the _requests table where the Cache-Control header contains the string &#8220;immutable&#8221;. The UNION query is also outputting a summary of all pages and request records in the table, which I used to calculate 30% of pages contain at least 1 request with a Cache-Control immutable response. 2% of all HTTP requests contain this directive.

    SELECT "CacheControl Immutable" description, count(distinct pageid) pages, count(*) objects
    FROM `httparchive.runs.2017_12_15_requests`
    WHERE LOWER(resp_cache_control) LIKE "%immutable%"
    UNION ALL 
    SELECT "All Requests" description, count(distinct pageid) pages, count(*) objects
    FROM `httparchive.runs.2017_12_15_requests`
    

<img loading="lazy" src="/assets/wp-content/uploads/2018/01/4e307ea2698549c952342260d6efd03cb593f59b.png" alt="" width="336" height="86" class="alignnone size-full wp-image-165" /> 

**Which Domains are Using Cache-Control: Immutable**

I modified the above query to aggregate by the hostname of the request. I also included the stats from the previous query in the percentage calculations. From this, we can see that Facebook requests (www.facebook.com, staticxx.facebook.com and static.xx.fbcdn.net) account for 62% of all Cache Control Immutable responses. Google (apis.google.com, securepubads.g.doubleclick.net, tpc.googlesyndication.com, www.google.com) account for 24%. Pinterest, Shopify, Tumblr, Disqus, Ebay, and Yelp accounted for another 8%. The remaining 6% of responses were spread out across 4,150 domain names.

    SELECT NET.HOST(url) domain, 
           count(distinct pageid) pages, 
           count(*) objects, 
           ROUND(count(distinct pageid) / 143055,2) as PercentPages, 
           ROUND(count(*) / 1078004,2) as PercentRequests
    FROM `httparchive.runs.2017_12_15_requests`
    WHERE LOWER(resp_cache_control) LIKE "%immutable%"
    GROUP BY domain
    ORDER BY objects DESC
    

<img loading="lazy" src="/assets/wp-content/uploads/2018/01/e6363990092a1bfb33b6fa3bfd38815111e69e93_1_690x359.png" alt="" width="690" height="359" class="alignnone size-full wp-image-164" /> 

**What Type of Content is Immutable Being Used For?**

So now that we know where it&#8217;s being used, lets explore how it&#8217;s being used. Here&#8217;s a breakdown of all the mime types. The top 12 mime types from this list account for 99.6% of Immutable responses and 87% of all responses.

    SELECT mimeType, count(*) objects
    FROM `httparchive.runs.2017_12_15_requests`
    WHERE LOWER(resp_cache_control) LIKE "%immutable%"
    GROUP BY mimeType
    ORDER BY objects DESC
    

<img loading="lazy" src="/assets/wp-content/uploads/2018/01/f4e78456ff172314d2e53cce474ad89b99aaf592.png" alt="" width="264" height="347" class="alignnone size-full wp-image-163" /> 

If we take the results from the above query, compare `All Requests` to the `Immutable Requests`, and then compare the distribution of mime types for each &#8211; we can see that JavaScript and HTML have a higher % of immutable responses compared to non-immutable responses.

<img loading="lazy" src="/assets/wp-content/uploads/2018/01/c47d3fc148afe73786e6ce76ea9847f021d76ea4_1_690x231.png" alt="" width="690" height="231" class="alignnone size-full wp-image-162" /> 

**What Cache-Control TTLs and directives are used alongside the immutable directive?**

So now that we know where it&#8217;s being used, and how it&#8217;s being used, let&#8217;s explore what other directives are being used alongside immutable responses. Here&#8217;s a query that uses regular expressions to extract the max-age directive, as well as public, private and must-revalidate. Note that the 1078004 value used in the percentage calculation is from the first query in this post.

        SELECT mimeType, maxage, pubpriv, mustrevalidate, SUM(objects) objects, SUM(PercentRequests) PercentRequests
        FROM (
            SELECT mimeType, 
                   REGEXP_EXTRACT(resp_cache_control,r"max-age=(\d+)") maxage,
                   REGEXP_EXTRACT(resp_cache_control,r"(public|private)") pubpriv,
                   REGEXP_EXTRACT(resp_cache_control,r"(must-revalidate)") mustrevalidate,
                   count(*) objects, 
                   ROUND(count(*) / 1078004,2) as PercentRequests
            FROM `httparchive.runs.2017_12_15_requests`
            WHERE LOWER(resp_cache_control) LIKE "%immutable%"
            GROUP BY mimeType,resp_cache_control
        )
        GROUP BY mimeType, maxage, pubpriv, mustrevalidate
        ORDER BY objects DESC
    

That returns 568 rows containing the following:<img loading="lazy" src="/assets/wp-content/uploads/2018/01/be0216bec0d9efcbc863171e902630bf02f94971_1_690x318.png" alt="" width="690" height="318" class="alignnone size-full wp-image-161" /> 

Most of the immutable responses are loaded with a public directive, except for text/javascript which contains a fair amount of cache-control private responses. A small percentage of overall responses appear to be including must-revalidate directives as well, which tells the browser that it must revalidate the resource when it&#8217;s freshness expires.<img loading="lazy" src="/assets/wp-content/uploads/2018/01/0de0728a05669fb31bc5f3804289534d26757cbd_1_690x219.png" alt="" width="690" height="219" class="alignnone size-full wp-image-160" /> 

For a resource that should not be revalidated, one would expect that the max-age value should be a high one. The table below shows (on the left) that there are a large number of max-age directives in use, and that they tend to lean towards longer durations. The majority of immutable resources have a max-age value >= 365 days (31536000 seconds), except for some application/javascript responses that are set to 1 day (86400 seconds).

<img loading="lazy" src="/assets/wp-content/uploads/2018/01/5275057f4d963bc5115a56a4bd09be6818b2ee4e_1_690x470.png" alt="" width="690" height="470" class="alignnone size-full wp-image-159" /> 

**Excluding Facebook and Google**

During one of the earlier queries, we learned that Facebook accounts for 62% of these stats and Google accounts for 24%. You can add the following to the WHERE clause on the above queries to remove them from the analysis.

    AND NET.HOST(url) NOT IN ("www.facebook.com", "staticxx.facebook.com", "static.xx.fbcdn.net", "apis.google.com", "securepubads.g.doubleclick.net", "tpc.googlesyndication.com", "www.google.com")
    

When doing this, I found that application/javascript accounted for 36.2% of immutable responses. Followed by image/jpeg (24.2%), text/css (10.7%) and image/png (7.8%). Furthermore, exploring the results by domain name using the following query &#8230;

        SELECT domain, mimeType, maxage, pubpriv, mustrevalidate, SUM(objects) objects, SUM(PercentRequests) PercentRequests
        FROM (
            SELECT NET.HOST(url) domain,
                   mimeType, 
                   REGEXP_EXTRACT(resp_cache_control,r"max-age=(\d+)") maxage,
                   REGEXP_EXTRACT(resp_cache_control,r"(public|private)") pubpriv,
                   REGEXP_EXTRACT(resp_cache_control,r"(must-revalidate)") mustrevalidate,
                   count(*) objects, 
                   ROUND(count(*) / 1078004,2) as PercentRequests
            FROM `httparchive.runs.2017_12_15_requests`
            WHERE LOWER(resp_cache_control) LIKE "%immutable%" 
            AND NET.HOST(url) NOT IN ("www.facebook.com", "staticxx.facebook.com", "static.xx.fbcdn.net", "apis.google.com", "securepubads.g.doubleclick.net", "tpc.googlesyndication.com", "www.google.com")
            GROUP BY domain, mimeType,resp_cache_control
        )
        GROUP BY domain, mimeType, maxage, pubpriv, mustrevalidate
        ORDER BY objects DESC       
    

&#8230; I was able to find that most sites appear to be doing a good job at keeping a max-age duration high for immutable responses (95% of responses have a max-age > 1 day). Additionally it seems that 1,136 domains (27%) are using must-revalidate alongside immutable, which tells browsers that as soon as the freshness time expires they must revalidate the resource.

<img loading="lazy" src="/assets/wp-content/uploads/2018/01/164a714c8bee0374ada27b8118ee47e13ce10cf3.png" alt="" width="667" height="189" class="alignnone size-full wp-image-158" /> 

**Conclusion**

In a year since [Facebook&#8217;s blog post](https://code.facebook.com/posts/557147474482256/this-browser-tweak-saved-60-of-requests-to-facebook) about Cache-Control Immutable, we can see that it&#8217;s usage has spread beyond Facebook and is being used by a handful of large 3rd parties and ~4150 additional domains. Across all of the pages in the HTTP Archive, 2% of requests and 30% of sites appear to include at least 1 immutable response. Additionally, most of the sites that are using it have the directive set on assets that have a long freshness lifetime.

_Originally published at <https://discuss.httparchive.org/t/cache-control-immutable-a-year-later/1195>_