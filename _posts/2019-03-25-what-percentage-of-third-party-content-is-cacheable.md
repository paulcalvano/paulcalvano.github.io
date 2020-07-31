---
title: "What Percentage of Third Party Content is Cacheable?"
date: 2019-03-25T00:00:00+00:00
author: Paul Calvano
layout: post
related_posts:
  - _posts/2018-05-15-analyzing-3rd-party-performance-via-http-archive-crux.md
  - _posts/2018-03-14-http-heuristic-caching-missing-cache-control-and-expires-headers-explained.md
  - _posts/2017-09-17-which-3rd-party-content-loads-before-render-start.md
---

When we talk about cacheability of web content, often times the discussion is around content that site operators have control over (ie, first party content). But what about third party content? How much of that is cacheable? I was chatting with @yoav about this on Friday, since it could be useful to understanding the benefits of [signed exchanges](https://developers.google.com/web/updates/2018/11/signed-exchanges) on accelerating third party content. Is it worth delivering cross origin resources on a site&rsquo;s HTTP/2 connection, avoiding the need to establish a new connection and eliminate bandwidth contention between 3rd party resources and 1st party ones? In order to answer that we need to understand how many third party resources are delivered without credentials, and therefore can be signed. We will use the resource's public cacheability as a proxy for that, and try to understand how common such third party resources are.

The concept of serving signed third party resources over the same connection as the main content is described in some more detail [here](https://docs.google.com/document/d/1ZYiksDWz-tx0CC7QVoLx8ce3GK0LHgW9U9zn16eBCkU/edit).

In order to query the HTTP Archive for this, we need to:

* Identify which resources are served from 3rd parties
* Determine which resources are considered cacheable.

**Identifying 3rd Party Content**

@igrigorik shared a technique in his post about [the distribution of 1st vs 3rd party resources. ](https://discuss.httparchive.org/t/what-is-the-distribution-of-1st-party-vs-3rd-party-resources/100). I've used it for another [analysis](https://discuss.httparchive.org/t/analyzing-3rd-party-performance-via-http-archive-crux/1359), and we'll use it again here. 

**Identifying Cacheable Content**

Examining the Cache-Control response headers, we should be able to filter out resources that are not publicly cacheable or require immediate revalidation. This would include Cache-Control headers that have 1 or more of the following directives:
* no-store
* no-cache
* max-age=0
* s-max-age=0
* must-revalidate
* private

The query for this is below, and based on it we can see that 30% of 3rd party requests are considered to be non-cacheable or cached with the private directive.

```
SELECT REGEXP_CONTAINS(resp_cache_control, r"no-store|no-cache|max-age=0|s-max-age=0|must-revalidate|private") AS is_noncacheable,
       COUNT(*) requests
FROM `httparchive.summary_requests.2019_02_01_desktop` requests 
JOIN `httparchive.summary_pages.2019_02_01_desktop` pages
ON pages.pageid = requests.pageid
WHERE STRPOS(req_host,REGEXP_EXTRACT(NET.REG_DOMAIN(pages.url), r'([\w-]+)'))<=0
GROUP BY is_noncacheable
```

![](/assets/img/blog/what-percentage-of-third-party-content-is-cacheable/1.png){:loading="lazy"}  

HTTP Response codes are another useful dimension here, and it looks like 81% of 3rd party content returning an HTTP 200 status code is cacheable.

```
SELECT status, REGEXP_CONTAINS(resp_cache_control, r"no-store|no-cache|max-age=0|s-max-age=0|must-revalidate|private") AS is_noncacheable,
       COUNT(*) requests
FROM `httparchive.summary_requests.2019_02_01_desktop` requests 
JOIN `httparchive.summary_pages.2019_02_01_desktop` pages
ON pages.pageid = requests.pageid
WHERE STRPOS(req_host,REGEXP_EXTRACT(NET.REG_DOMAIN(pages.url), r'([\w-]+)'))<=0
GROUP BY status, is_noncacheable

```

![](/assets/img/blog/what-percentage-of-third-party-content-is-cacheable/2.png){:loading="lazy"}

Taking this one step further, we can also evaluate the public cacheability of resources from popular third party domains. 

```
SELECT NET.HOST(requests.url) req_host,
       REGEXP_CONTAINS(resp_cache_control, r"no-store|no-cache|max-age=0|s-max-age=0|must-revalidate|private") AS is_noncacheable,
       COUNT(*) requests
FROM `httparchive.summary_requests.2019_02_01_desktop` requests 
JOIN `httparchive.summary_pages.2019_02_01_desktop` pages
ON pages.pageid = requests.pageid
WHERE STRPOS(req_host,REGEXP_EXTRACT(NET.REG_DOMAIN(pages.url), r'([\w-]+)'))<=0
      and status=200
GROUP BY req_host, is_noncacheable
ORDER BY requests DESC
LIMIT 10000
```

![](/assets/img/blog/what-percentage-of-third-party-content-is-cacheable/3.png){:loading="lazy"}

Based on these results, it definitely seems that there's a significant amount of third party content that is considered publicly cacheable based on the cache control headers. I'm looking forward to seeing how the proposal for signed exchanges impacts the delivery of third party content - especially the ones that sites need to load early in the critical render path. 

*Thanks to Yoav Weiss for his help with this analysis!*

_Originally published at <https://discuss.httparchive.org/t/what-percentage-of-third-party-content-is-cacheable/1614>_