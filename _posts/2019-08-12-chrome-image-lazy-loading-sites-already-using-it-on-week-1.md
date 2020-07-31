---
title: "Chrome Image Lazy Loading - Sites Already Using it on Week 1!"
date: 2019-08-12T-00:00:00+00:00
author: Paul Calvano
layout: post
related_posts:
  - _posts/2019-01-11-correlating-performance-metrics-to-page-characteristics.md
  - _posts/2018-07-02-impact-of-page-weight-on-load-time.md
  - _posts/2017-08-16-tracking-page-weight-over-time.md

---

Earlier this year the Chrome team announced plans to [support lazy loading natively](https://addyosmani.com/blog/lazy-loading/) in the browser. The plan was to add a loading attribute in both `<img>` and `<iframe>` elements. Chrome 75 included it behind a feature flag so that developers could test it out. Last week, with the release of [Chrome 76 this feature became generally available](https://twitter.com/hdjirdeh/status/1158984255214460928). I was surprised to see that it is already in use by more than 1000 sites. Lazy loading is an easy web performance win, so you may want to try this out on your sites.

Image lazy loading works by deferring the loading of images that are outside of the viewport until the user begins to scroll down the page. Historically, implementations for lazy loading involved setting the `<img>` src attribute to a small placeholder image and then using JavaScript to swap the image as it is needed. As a result, there have been many different implementations of lazy loading. Jeremy Wagner wrote an [excellent guide](https://developers.google.com/web/fundamentals/performance/lazy-loading-guidance/images-and-video/) on how this works along with examples and links to open source libraries.

However Chrome’s native lazy loading changes this by making it incredibly easy to configure for your site. Houssein Djirdeh, Addy Osmani and Mathias Bynens [wrote about it here](https://web.dev/native-lazy-loading). Enabling it is as simple as adding the loading=”lazy” attribute to your images or iFrames:

```

<img src="image.png" loading="lazy" alt="…" width="200" height="200">

<iframe src="https://example.com" loading="lazy"></iframe>

```

When I saw the announcement that it was shipped in Chrome 76, I was curious to see how many sites have Chrome’s lazy loading attribute in production. After querying the HTTP Archive response_bodies table, I was surprised to find that more than 1000 sites already implemented the image lazy loading feature. Since this was against the July 2019 dataset, that means that these sites enabled the feature before it was available in the browser. Additionally 52 sites have implemented iFrame lazy loading. Unsurprisingly, most of the usage is `loading=lazy`.

![624x612](/assets/img/blog/chrome-image-lazy-loading-sites-already-using-it-on-week-1/1.png){:loading="lazy"}

Let’s step back and take a look at some of the stats around image weight and lazy loading. From a web performance perspective there are a few usual suspects slowing sites down - including but not limited to page weight, third parties, and CPU bottlenecks. I doubt this is a surprise to anyone anymore, but according to the[ HTTP Archive](https://httparchive.org/reports/state-of-the-web?start=earliest&end=latest&view=list#bytesTotal) page weight has been increasing for years. We’ve known for a while that [heavier sites are more likely to be slower](https://paulcalvano.com/index.php/2018/07/02/impact-of-page-weight-on-load-time/) and images are the largest source of page weight. In fact the graphs below show that both desktop and mobile home pages contain more than 70% image bytes!

![624x228](/assets/img/blog/chrome-image-lazy-loading-sites-already-using-it-on-week-1/2.png){:loading="lazy"}

Pages containing a large amount of image bytes also tend to include a large number of image assets, rather than just a few large images. For example, at the 90th percentile pages with at least 3MB of images contained over 110 image resources, and that number increases with every added MB!

![624x420](/assets/img/blog/chrome-image-lazy-loading-sites-already-using-it-on-week-1/3.png){:loading="lazy"}

It’s likely that many of the images loaded on large image heavy sites are not displayed within the viewport of a user’s browser, and only appear when a user scrolls or interacts with a page. We can confirm this with the [Lighthouse](https://developers.google.com/web/tools/lighthouse/) audit for [off-screen images](https://developers.google.com/web/tools/lighthouse/audits/offscreen-images). Since there is a cost to loading images, reducing their impact can help reduce bandwidth/data usage for clients as well as speed up the load times for web pages.

The graphic below shows the relationship of image weight to offscreen images, as measured by Lighthouse. With each additional MB of images, we can see a consistent increase in the amount of off-screen images. In fact, 80-90% of pages with > 3MB images are loading more than 1MB of those bytes off screen. That’s a lot of wasted bytes!

![624x163](/assets/img/blog/chrome-image-lazy-loading-sites-already-using-it-on-week-1/4.png){:loading="lazy"}

Chrome’s lazy loading feature was literally just released this week, and it's already in use by more than 1000 sites. That’s quite impressive, and it will be interesting to track the adoption of this feature by other websites and browsers in the future.

See where your homepage sits on the table above and if you have a lot of off-screen bytes go ahead and give Chrome’s new lazy loading feature a try!



**Queries:** 

Usage of the Lazy Loading Attribute in Chrome
*Note: The response_bodies  table is quite large, so I’ve saved a table containing all the lazy loading occurrences in httparchive.scratchspace.lazy_loading_2019_07_desktop if anyone would like to dig into some specific examples of pages that are using the loading attribute.*

```
SELECT page,
       url,
       REGEXP_EXTRACT_ALL(LOWER(body), r'(?i)<img.*\sloading\s*=\s*"(lazy|eager|auto)".*\/>') img_lazy_loading,
       REGEXP_EXTRACT_ALL(LOWER(body), r'(?i)<iframe.*\sloading\s*=\s*"(lazy|eager|auto)".*\/>') iframe_lazy_loading
FROM `httparchive.response_bodies.2019_07_01_desktop` bodies
WHERE url IN (
       SELECT p.url as url
       FROM `httparchive.summary_pages.2019_07_01_desktop`   p
       INNER JOIN `httparchive.summary_requests.2019_07_01_desktop` r
       ON p.pageid = r.pageid
       WHERE  firstHtml = true
       )
```

Lazy Loading Summary
```
SELECT "img" type, loading, count(distinct page) pages
FROM `httparchive.scratchspace.lazy_loading_2019_07_desktop`
 CROSS JOIN
  UNNEST(img_lazy_loading) loading
GROUP BY loading
UNION ALL 
SELECT "iframe" type, loading, count(distinct page) pages
FROM `httparchive.scratchspace.lazy_loading_2019_07_desktop` 
 CROSS JOIN
  UNNEST(iframe_lazy_loading) loading
GROUP BY loading
```


Average page weight by content type
```
SELECT  "desktop" as desktop_mobile,
        ROUND(AVG(bytesHtml/1024),2) HTML,
        ROUND(AVG(bytesJS/1024),2) JavaScript,
        ROUND(AVG(bytesCSS/1024),2) CSS,
        ROUND(AVG(bytesImg/1024),2) Images,
        ROUND(AVG(bytesFont/1024),2) Fonts,
        ROUND(AVG((bytesFlash+bytesJson+bytesOther)/1024),2) AS Other
FROM `httparchive.summary_pages.2019_07_01_desktop` 
UNION ALL
SELECT  "mobile" as desktop_mobile,
        ROUND(AVG(bytesHtml/1024),2) HTML,
        ROUND(AVG(bytesJS/1024),2) JavaScript,
        ROUND(AVG(bytesCSS/1024),2) CSS,
        ROUND(AVG(bytesImg/1024),2) Images,
        ROUND(AVG(bytesFont/1024),2) Fonts,
        ROUND(AVG((bytesFlash+bytesJson+bytesOther)/1024),2) AS Other
FROM `httparchive.summary_pages.2019_07_01_mobile` 
```

Pages with the high image weights tend to have a large number of images
```
SELECT  "desktop" as desktop_mobile,
        ROUND(bytesImg/1024/1024) img_weight,
        count(*),
        APPROX_QUANTILES(reqImg, 100)[SAFE_ORDINAL(25)] AS pct25th,
        APPROX_QUANTILES(reqImg, 100)[SAFE_ORDINAL(75)] AS pct75th,
        APPROX_QUANTILES(reqImg, 100)[SAFE_ORDINAL(90)] AS pct90th
FROM `httparchive.summary_pages.2019_07_01_desktop`
GROUP BY img_weight
ORDER BY img_weight ASC
```

Lighthouse OffScreen Image Audit
```
SELECT ROUND(bytesImg/1024/1024) img_weight,
       ROUND(CAST(JSON_EXTRACT_SCALAR(report, "$.audits.offscreen-images.details.overallSavingsBytes")as INT64)/1024/1024) offscreenMB,
       count(*)   
FROM `httparchive.lighthouse.2019_07_01_mobile`  lh 
INNER JOIN `httparchive.summary_pages.2019_07_01_mobile`  p 
ON lh.url = p.url
GROUP BY img_weight,offscreenMB
ORDER BY img_weight,offscreenMB
```

Sites Using Image Lazy Loading
```
SELECT page, loading, count(*)
FROM `httparchive.scratchspace.lazy_loading_2019_07_desktop` 
 CROSS JOIN
  UNNEST(img_lazy_loading) loading
GROUP BY page, loading
```

Sites using iFrame Lazy Loading
```
SELECT page, loading, count(*)
FROM `httparchive.scratchspace.lazy_loading_2019_07_desktop` 
 CROSS JOIN
  UNNEST(iframe_lazy_loading) loading
GROUP BY page, loading
```

_Originally published at <https://discuss.httparchive.org/t/chrome-image-lazy-loading-sites-already-using-it-on-week-1/1707/>_