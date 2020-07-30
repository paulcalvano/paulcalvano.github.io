---
title: "How Many Sites Are Still Using AppCache?"
date: 2018-08-24T00:00:00+00:00
author: Paul Calvano
layout: post
---

The [Application Cache](https://developer.mozilla.org/en-US/docs/Web/HTML/Using_the_application_cache) has been deprecated and removed from the web standards. While some browsers still support it - that support is going away.  For example, starting with Firefox 44 a console warning advised developers to use Service Workers instead. In Chrome v68, when an HTTP page loads with AppCache configured, the browser provides a warning that v69 will restrict AppCache to secure context only.  
![](/assets/img/blog/how-many-sites-are-still-using-appcache/1.png) 

And this evening I saw the intent to fully deprecate AppCache in Blink. 
https://twitter.com/intenttoship/status/1032822524567011328

After reading this, I started wondering how many sites are still using AppCache today...

**Finding AppCache Manifests in Response Bodies**
The application cache is enabled by including a manifest HTML attribute in a page.   That mainfest then lists resources that should be cached. I decided to see if I could craft a regular expression to both identify the presence of the mainfest and also extract the manifest filename.  The following looked promising - 

![](/assets/img/blog/how-many-sites-are-still-using-appcache/2.png) 


I used that regular expression in the following query, which outputs a list of all URLs where such a manifest is defined.   **_(Note, this query processes 2.5 TB of data which exceeds the free tier by 1.5TB )_**

```
SELECT page,url, appcache_manifest
FROM (
  SELECT page, url, 
       REGEXP_EXTRACT(LOWER(body), r'<html.*manifest=["](.*)["] ') appcache_manifest
  FROM `httparchive.response_bodies.2018_08_01_desktop`
 )
WHERE appcache_manifest IS NOT null
```

The results look like this, and you can see that cache.appcache seems to be a popular manifest name. I also found examples like `cache.manifest`, `mainfest.appcache`, `offline.appcache` and others.

![](/assets/img/blog/how-many-sites-are-still-using-appcache/3.png) 

I only found 268 pages out of 1,275,374 pages in the HTTP Archive that matched this pattern. 65 of them are HTTP pages, and 203 of then are HTTPS. That seemed low, so I decided to check another way.

**Lighthouse to the Rescue**

[Lighthouse has an audit that indicates whether AppCache is used](https://developers.google.com/web/tools/lighthouse/audits/appcache).   They also have a [ServiceWorker audit](https://developers.google.com/web/tools/lighthouse/audits/registered-service-worker).  So I decided to give that a try.   Here'a query that will aggregate the scores for  the AppCache and ServiceWorker audits and group it by protocol.  **_Note: Processes 197GB of data_**

```
SELECT SUBSTR(url, 0, 5) p,
       JSON_EXTRACT_SCALAR(report, "$.audits.appcache-manifest.score") AS appcache,
       JSON_EXTRACT_SCALAR(report, "$.audits.service-worker.score") AS serviceWorkers,
       count(*)     
FROM `httparchive.lighthouse.2018_08_01_mobile` 
GROUP BY p, appcache, serviceWorkers
```

The results look like this - 

![](/assets/img/blog/how-many-sites-are-still-using-appcache/4.png) 

When we clean up the output and display it in a cross tabular format, it's a lot easier to analyze. From the results below, we can see that there are 1,126 sites still using AppCache (and only 23 of them are using ServiceWorkers).  301 of the sites using AppCache are still serving content over HTTP - which means they will experience the deprecation first.

![](/assets/img/blog/how-many-sites-are-still-using-appcache/5.png) 

There are 14,844 sites that registered a service worker though, which is promising!  The 346 sites trying to register service workers on HTTP pages is interesting since the service worker will not run on insecure pages. Perhaps there's more to investigate here...

_Originally published at <https://discuss.httparchive.org/t/how-many-sites-are-still-using-appcache/1443>_