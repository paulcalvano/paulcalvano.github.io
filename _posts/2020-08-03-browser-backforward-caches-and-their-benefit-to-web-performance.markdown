---
layout: post
title: "Browser Back/Forward Caches and their Benefit to Web Performance"
date: 2020-08-03 14:30:00 -0400
related_posts:
 
  - _posts/2019-03-25-what-percentage-of-third-party-content-is-cacheable.md
  - _posts/2018-03-14-http-heuristic-caching-missing-cache-control-and-expires-headers-explained.md
  - _posts/2018-01-07-cache-control-immutable-a-year-later.md

---

**Overview**

There are a few different types of top-level navigation events that browsers handle. The most common one is a simple “navigation”. You enter a URL in the address bar and hit enter, or you click on a hyperlink to visit another webpage. Another is a “reload”, which causes the browser to revalidate it’s cached entries. There’s also a “hard reload”, where the browser ignores cache and reloads the page. And another type of navigation, which we will discuss in this post, is “Back/Forward” navigation. These occur when you click the back or forward buttons in your browser to return to a previously visited page.

If you are using a real user measurement (RUM) service like[ mPulse](https://www.akamai.com/us/en/products/performance/mpulse-real-user-monitoring.jsp) then you are likely collecting the stats for every navigation type that results in a page load. The[ navigation timing API](https://www.w3.org/TR/navigation-timing/) used by RUM services also includes the[ navigation type](https://www.w3.org/TR/navigation-timing/#sec-navigation-info-interface) which makes it possible to determine which one was used. You can see the value in your browser console by checking the value of `performance.navigation.type`. The values defined by the specification are below:


<table>
  <tr>
   <td>Navigation Event
   </td>
   <td>Type Attribute
   </td>
   <td>Description
   </td>
  </tr>
  <tr>
   <td>TYPE_NAVIGATE
   </td>
   <td>0
   </td>
   <td>Navigation started by clicking a link, entering a URL in the address bar, submitting a form, or triggered by a script (except the ones used by reload or back_forward)
   </td>
  </tr>
  <tr>
   <td>TYPE_RELOAD
   </td>
   <td>1
   </td>
   <td>Navigation through the reload operation or the `location.reload()` method.
   </td>
  </tr>
  <tr>
   <td>TYPE_BACK_FORWARD
   </td>
   <td>2
   </td>
   <td>Navigation through a history traversal operation.
   </td>
  </tr>
  <tr>
   <td>TYPE_RESERVED
   </td>
   <td>255
   </td>
   <td>Any navigation types not defined above.
   </td>
  </tr>
</table>


mPulse collects the navigation types for every real user measurement across thousands of websites. Analyzing this data can provide an interesting perspective on the performance of back/forward navigations as well as how their usage varies by device and browser. 

**Back/Forward Caches in Popular Browsers**

The general idea behind a back/forward cache is to preserve the state of the page so that the DOM does not need to be rebuilt when a user returns to a previously visited page. With a back/forward cache the render tree and layout remains intact. The page is not technically loaded again because it was already loaded.

[Chrome](https://www.chromestatus.com/feature/5815270035685376),[ Firefox](https://developer.mozilla.org/en-US/docs/Archive/Misc_top_level/Working_with_BFCache) and[ Safari](https://webkit.org/blog/427/webkit-page-cache-i-the-basics/) all have implementations of a back/forward cache, although Chrome’s is currently hidden behind an experimental flag (chrome://flags/#back-forward-cache). 

The Chrome team has announced its intention to enable the back/forward cache feature for Chrome Mobile users[ starting with Chrome 86 (October 2020)](https://groups.google.com/a/chromium.org/forum/#!msg/blink-dev/S9qRFx4ozXk/DNT8tiR3BAAJ). In this post we will examine how frequently back/forward navigations are used across different devices and browsers. We’ll also look at the performance opportunities that the back/forward cache implementation may be able to solve.

_Note: Because of how back/forward caches work, we do not have real user measurements for back/forward cache hits. In this analysis we’ll explore the opportunity presented by back/forward caches, and not the performance gained by any specific browser’s implementation. _

**Summary of Navigation Types**

The graph below illustrates the breakdown of different navigation types by device type. While the content of this graph is very high level and seems to be simple, it’s important to note that this is an aggregation of tens of billions of real user measurements from thousands of websites, measured during the month of June 2020.

There are a few interesting observations we can make from this graph:

*   Across all device types, ~85% of navigations are type=navigation.  
*   There is a greater percentage of back/forward navigations on mobile and tablet devices compared to desktop
*   Desktop browsers had 60% more reloads compared to mobile/tablet. 



![Navigation Types by Device Type](/assets/img/blog/browser-backforward-caches-and-their-benefit-to-web-performance/image1.jpg){:loading="lazy"}


**Desktop Browsers**

The most popular desktop browsers are Chrome, Safari, Edge and Firefox.   In fact, combined they account for 91.4% of all Desktop pages!

_Note: When looking at this data it’s also important to note that in Firefox and Safari we only have the back/forward navigation events for navigations that did not result in a back/forward cache hit. _ 

When we look at this by desktop browser, we can see that

*   Chrome and Opera have a large % of reloads compared to other browsers
*   Almost 17% of Opera navigations are back/forward!
*   All browsers except for Safari Desktop and legacy IE have at least 10% of back/forward navigations.
*   There were 50% less back/forward navigations measured for Safari compared to Chrome. This seems to indicate that approximately half the time users are getting a cache hit from this feature.
*   Firefox had a similar rate of back/forward navigations to Chrome. This seems to indicate that either Back/Forward navigation are much more prevalent in Firefox, or that their implementation is not resulting in many cache hits.


![Navigation Types by Desktop Browser](/assets/img/blog/browser-backforward-caches-and-their-benefit-to-web-performance/image2.jpg){:loading="lazy"}


The graph below illustrates the median load times by navigation types for these browsers. For each browser, the reload events are significantly slower. On Chrome, reloads took 30% longer than navigations. Firefox saw the biggest degradations with reload events, taking almost twice as long as navigations.

Comparing the navigation and back/forward load times, we can see that legacy IE and Opera back/forward load times were only slightly faster than a navigation. With Chrome, the back/forward navigations are 16% faster than navigations. Safari back/forward navigations are 27% faster. That’s without any back/forward cache, but merely relying on the efficiency of the browser cache.

![Load Times per Desktop Navigation Types](/assets/img/blog/browser-backforward-caches-and-their-benefit-to-web-performance/image3.jpg){:loading="lazy"}

**Mobile Browsers**

The most popular mobile browsers are Mobile Safari and Chrome Mobile, which together accounts for 67.6% of mobile page views. The graph below also includes other popular mobile browsers such as Samsung Internet, UC Browser, Crosswalk, and Firefox Mobile.   Additionally included are webviews and in-app browsers are included - such as Facebook, Instagram, LINE, Flipboard, Snapchat, Pinterest, etc.

Some observations: 



*   Back/Forward usage is much more prominent in mobile browsers compared to desktop browsers.
*   Mobile browsers have a much higher back/forward usage compared to WebViews and in-app browsers.
*   Considering the difference between Chrome Mobile iOS vs Mobile Safari (both Wekbit based), as well as Chrome Mobile vs Samsung Internet vs Crosswalk (all Chromium based), the amount of back/forward navigation events may be influenced by the browser UI.

![Navigation Types by Mobile Browser](/assets/img/blog/browser-backforward-caches-and-their-benefit-to-web-performance/image4.jpg){:loading="lazy"}

**Is this a Blind Spot for Synthetic?**

When measuring the performance of a synthetic transaction, you are usually starting a new session - which means that the page loads will be navigations.  Since most synthetic services identify themselves in their User-Agent, we’re able to compare navigation types across popular synthetic solutions using mPulse data. In the graph below you can see that the vast majority of requests from synthetic services are navigation events. There are a small percentage of reloads, likely due to multi-step scripts that repeat a page view. 


![Navigation Types by Desktop Synthetic](/assets/img/blog/browser-backforward-caches-and-their-benefit-to-web-performance/image5.jpg){:loading="lazy"}

Regardless of what we see today, on the measurement platform you use it may be possible to script these types of navigations. For example, in[ WebPageTest](https://webpagetest.org/) I can trigger a back/forward navigation by[ using a script](https://sites.google.com/a/webpagetest.org/docs/using-webpagetest/scripting)<span style="text-decoration:underline;"> </span>such the one below:

```
// Turn off logging of results
logData	0   
// Navigate to the first page.  Then navigate to another page.
navigate	https://www.google.com/
navigate	https://www.google.com/search?q=back+forward+cache
// Wait for 2 seconds
sleep	2 
// Turn on logging of results
logData	1 
// Use JavaScript to trigger the back button
execAndWait	window.history.back();
```

**What does this mean for RUM?**

Many RUM implementations, such as mPulse, will aggregate and report on all user navigations.  So the results that you are looking at will include a mix of navigation, reload, back/forward and reserved navigation.   Based on the data here, ~ 10% of Desktop and ~20% of Mobile page loads utilize Back/Forward navigation. 

If a navigation is served from the Back/Forward cache, then it will not trigger a beacon since the browser is unfreezing the browser state.  In this case there will be no load event to trigger a beacon.

We may look at ways to support timing back/forward navigation from cache in the future - but for now, I would expect the following:

*   Once Back/Forward cache rolls out for a browser, you might see a drop in page loads from that browser (since those navigation types will no longer be reported)
*   Additionally, load times may appear to be slightly higher because some of the faster experiences (a back/forward navigation utilizing the browser cache) will no longer be reported. Essentially, these measurements will be affected by[ survivorship bias.](https://en.wikipedia.org/wiki/Selection_bias#Attrition)

I would expect this to apply to other analytics tools as well.

**Conclusion**

While we often think of navigating the web as the simple operation of loading a web page, there are different types of navigation events which can be measured.  Reloads are slower, and many of us are accustomed to that in normal use. However we expect the Back/Forward operations to be fast - and they are faster. The implementations of Back/Forward caches in popular browsers are helping to improve this experience even further - which has the benefit of significantly speeding up the web for up to 20% of navigations! 


_Many thanks to [Nic Jansma](https://twitter.com/nicj) and [Utkarsh Goel](https://www.utkarshgoel.in/) for reviewing this._
