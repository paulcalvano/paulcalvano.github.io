---
layout: post
title: "Third Parties and Single Points of Failure"
date: 2025-12-29 00:00:00 -0400

---

You’ve heard it many times - third party content can easily cause an otherwise well performing website to become sluggish and slow. And depending on how this content is loaded, it can also introduce single points of failure (SPOFs). When a large cloud provider or content delivery network (CDN) experiences a disruption, their impacts are felt across the world and often triggers headlines about the many websites that were affected. However, there are numerous secondary impacts triggered by third party content, which can be disruptive even to companies that don’t use the affected provider.

![Caution - Single Points of Failure ](/assets/img/blog/third-parties-and-single-points-of-failure/spof-warning.jpg){:loading="lazy"}

In this blog post I’ll discuss some of the performance and availability risks associated with third party content and how you can test for these single points of failure (SPOFs) on your websites. I’ll use [HTTP Archive](https://httparchive.org/) data to explore how many sites could be at risk and [RUM Archive](https://rumarchive.com/) to get a sense of how prone to slowdowns some of these third parties are! 

# Third Party Failure = SPOF

A third-party single point of failure (SPOF) can occur when a website depends on an external service for critical or render-blocking resources. If the third party content fails to load, then the page load may be stalled until the request times out. For example, in the two filmstrips (taken from a [WebPageTest private instance](https://docs.webpagetest.org/private-instances/)) you can see how a failure of a third-party affects the user experience. The SPOF measurement in this example simulates what a client would see if a render-blocking third party, such as a consent management service, became unavailable. In this case, the user would essentially see a blank screen until the third party request times out. 

![WebPageTest Private Instance Filmstrip showing a SPOF](/assets/img/blog/third-parties-and-single-points-of-failure/spof-filmstrip-wpt-private-instance.jpg){:loading="lazy"}

Most recently, people have been noticing SPOFs due to outages experienced by large cloud providers and CDNs. For example, a major CDN recently experienced a few high profile outages and many of their customers were impacted. However, many sites that don’t use their services directly were also affected due to third party content that was being delivered through them. That could have been avoided by self hosting content that is critical to loading of the website.

This risk isn’t just isolated to CDNs, nor is it a new issue. For example, many years ago websites started adding Facebook like buttons to their pages via synchronously loaded scripts. When Facebook experienced [an outage way back in 2012](https://www.forbes.com/sites/ericsavitz/2012/06/01/facebook-outage-slowed-1000s-of-retail-content-sites/), the failed third party requests slowed down a large number of websites and browsers were stalled waiting for a script to load. Facebook fixed this a long time ago, and even introduced a [non-blocking script loader pattern](https://calendar.perfplanet.com/2012/the-non-blocking-script-loader-pattern/) for other websites to adopt. Additionally, [the like button is being deprecated](https://developers.facebook.com/docs/plugins/like-button/) in February 2026. They plan to return an empty response to avoid issues on sites that will inevitably forget to remove them!

There have been other occurrences where a third party stops providing their service or is compromised as well. Using the HTTP Archive, we can see how many websites are still referencing this old content. For example: 

* RawGit [announced](https://rawgit.com/) it was shutting down in 2018. Their webpage recommends alternatives, but based on the November 2025 HTTP Archive data, there are over 25K websites still requesting content from it!
* In February 2024, the Polyfill.io service was sold to a third party and concerns were raised immediately about what that meant for sites using the service. By June 2024 it was used to deliver malicious content via a [supply chain attack](https://thehackernews.com/2024/07/polyfillio-attack-impacts-over-380000.html). Based on HTTP Archive data, I can see references to this third party on 161K websites in February 2024, and then it gradually decreased from 115K to 109K through June 2024!  All occurrences of its use stopped after July 2024 – possibly due to intervention by browser vendors and their domain registrar taking down the domain name. 

Non-web applications can also experience issues from third party failures as well. For example, a while back one of Etsy’s CDNs experienced an incident and I needed to fail over our traffic to our other CDN. During the failover, I discovered that part of the process for this CDN failover actually relied on a third party which had a dependency on the same CDN I needed to route away from! To avoid a similar occurrence, a break-glass failover mechanism was created to bypass all dependencies on CDNs during a failover event.

# But I Don’t Use &lt;Provider>?

When you add a third party to your website, you are telling the browser to load their content as if it were your own. Depending on how that content is loaded, it could block the page from rendering anything to the screen. If a render-blocking third party happens to be served by a provider that is experiencing a performance degradation or outage, then the site’s performance may be significantly affected.

Any third party content added to your website carries risks, and should be added with care and tested thoroughly. However, extra care should be taken when it comes to render-blocking requests. This isn’t new advice either - Steve Souders first [wrote about this in 2010](https://www.stevesouders.com/blog/2010/06/01/frontend-spof/)! There have been PerfPlanet articles about this dating back to 2011 (just search for [SPOF](https://calendar.perfplanet.com/?s=SPOF))! A simple recommendation remains relevant 15 years later: *avoid third party single points of failure, and test/monitor your pages to ensure that none are introduced.*


# How Prevalent are Third Party SPOFs Today? 

I’m raising a lot of concern about a problem that has been well-known and talked about for over 15 years. You might wonder how much of an issue this can still be on today’s web. According to the December 2025 [HTTP Archive](https://httparchive.org/) dataset, a shocking 67.7% (of 15.78 million websites) request at least one render-blocking third party! *Note: This classification is based on request URLs that are from a different domain name than the page URLs. This does not include subdomains of the page URLs host, which means the actual numbers may be higher!)*

Additionally, 60% (9.4 million) of sites load at least one render-blocking third party through a different CDN from their primary content! Even more shocking is that there are over 1 million websites loading render-blocking content via a third party that does not use a CDN at all!

<table>
  <tr>
   <td colspan="3" align="center"><strong>Sites with Render Blocking Third Parties - HTTP Archive December 2025</strong></td>
  </tr>
  <tr>
   <td><strong>Category</strong></td>
   <td><strong>Number of Sites</strong></td>
   <td><strong>% of Sites</strong></td>
  </tr>
  <tr>
   <td>Number of Sites</td>
   <td>15,780,490</td>
   <td></td>
  </tr>
  <tr>
   <td>Sites with a Render Blocking Third Party</td>
   <td>10,683,209</td>
   <td>67.70%</td>
  </tr>
  <tr>
   <td>Sites with a Render Blocking Third Party on a different CDN</td>
   <td>9,444,516</td>
   <td>59.85%</td>
  </tr>
  <tr>
   <td>Sites with a Render Blocking Third Party not using a CDN</td>
   <td>1,075,294</td>
   <td>6.81%</td>
  </tr>
</table>

If we focus on the websites that have a render-blocking third party hosted on a different CDN, you can see that the request types skew towards CSS and JavaScript (with many sites loading both of these types of requests).

![Third Party Render Blocking Requests by Content Type - HTTP Archive Dec 2025](/assets/img/blog/third-parties-and-single-points-of-failure/third-party-render-blocking-requests-http-archive.jpg){:loading="lazy"}

When we break this down by hostname and content type, you can see that Google Fonts is one of the larger sources of render-blocking third parties. There are also other font providers like Typekit and Fontawesome as well. Since CSS is generally render-blocking, including third party CSS for font loading may introduce a SPOF risk!!

Additionally, there are hundreds of thousands of sites that are utilizing third parties to load libraries onto their sites. For example, `cdnjs.cloudflare.com`, `cdn.jsdelivr.net`, `ajax.googleapis.com`, `code.jquery.com`, `maxcdn.bootstrapcdn.com`, and `unpkg.com`. If you find yourself using any of these to deliver content that is critical to the loading of your website, I highly encourage you to read Harry Robert’s excellent writeup about why you should [self host your static assets](https://csswizardry.com/2019/05/self-host-your-static-assets/).

<table>
  <tr>
   <td colspan="6" align="center" ><strong>Websites w/ Third Party Content - HTTP Archive December 2025</strong></td>
  </tr>
  <tr>
   <td><strong>hostname</strong></td>
   <td><strong>css</strong></td>
   <td><strong>script</strong></td>
   <td><strong>other</strong></td>
   <td><strong>html</strong></td>
   <td><strong>text</strong></td>
  </tr>
  <tr>
   <td>fonts.googleapis.com</td>
   <td>5,837,138</td>
   <td></td>
   <td></td>
   <td>22</td>
   <td></td>
  </tr>
  <tr>
   <td>www.gstatic.com</td>
   <td>1,277,310</td>
   <td>1,273,686</td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>cdnjs.cloudflare.com</td>
   <td>675,493</td>
   <td>390,541</td>
   <td>9</td>
   <td></td>
   <td>115</td>
  </tr>
  <tr>
   <td>cdn.jsdelivr.net</td>
   <td>660,322</td>
   <td>351,682</td>
   <td>20</td>
   <td>34</td>
   <td>6</td>
  </tr>
  <tr>
   <td>www.youtube.com</td>
   <td>772,925</td>
   <td>39,762</td>
   <td>5</td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>ajax.googleapis.com</td>
   <td>57,354</td>
   <td>702,890</td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>code.jquery.com</td>
   <td>67,244</td>
   <td>368,220</td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>maxcdn.bootstrapcdn.com</td>
   <td>332,198</td>
   <td>49,794</td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>use.fontawesome.com</td>
   <td>315,930</td>
   <td>21,300</td>
   <td></td>
   <td>6</td>
   <td></td>
  </tr>
  <tr>
   <td>tpc.googlesyndication.com</td>
   <td>173</td>
   <td>299,810</td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>use.typekit.net</td>
   <td>251,913</td>
   <td>43,365</td>
   <td></td>
   <td>6</td>
   <td>71</td>
  </tr>
  <tr>
   <td>static.xx.fbcdn.net</td>
   <td>147,085</td>
   <td>136,605</td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>p.typekit.net</td>
   <td>272,158</td>
   <td></td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>static1.squarespace.com</td>
   <td>236,853</td>
   <td>6,458</td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>definitions.sqspcdn.com</td>
   <td>204,076</td>
   <td>37,235</td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>unpkg.com</td>
   <td>127,853</td>
   <td>84,910</td>
   <td></td>
   <td></td>
   <td>13,738</td>
  </tr>
  <tr>
   <td>pagead2.googlesyndication.com</td>
   <td>435</td>
   <td>203,536</td>
   <td></td>
   <td>204</td>
   <td></td>
  </tr>
  <tr>
   <td>www.google.com</td>
   <td>326</td>
   <td>186,648</td>
   <td>146</td>
   <td>14,379</td>
   <td></td>
  </tr>
  <tr>
   <td>assets.squarespace.com</td>
   <td>135,575</td>
   <td>41,561</td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>c0.wp.com</td>
   <td>85,064</td>
   <td>90,345</td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>stackpath.bootstrapcdn.com</td>
   <td>101,810</td>
   <td>27,460</td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>kit.fontawesome.com</td>
   <td>4,497</td>
   <td>101,627</td>
   <td>14</td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>maps.googleapis.com</td>
   <td></td>
   <td>98,646</td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>a.amxrtb.com</td>
   <td></td>
   <td>97,849</td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>cdn-cookieyes.com</td>
   <td></td>
   <td>91,836</td>
   <td></td>
   <td>10</td>
   <td></td>
  </tr>
  <tr>
   <td>consent.cookiebot.com</td>
   <td></td>
   <td>79,041</td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
  <tr>
   <td>t1.daumcdn.net</td>
   <td>22,278</td>
   <td>54,285</td>
   <td></td>
   <td></td>
   <td></td>
  </tr>
</table>


# Third Party Slowdowns

While major cloud provider failures are not a frequent occurrence, slowdowns and microoutages absolutely are. If your content is delivered by a third party then the performance of that content is out of your control. Self hosting where possible will put you in control of the delivery of this content and reduce the risk of a slowdown triggered by a third party. If you have to load any content from a third party, you can (and should) test for single points of failure and monitor their performance.

Some Real User Monitoring (RUM) services have the ability to collect resource timing data, which makes it possible to monitor third party content across all your site’s visitors. You can also monitor via synthetic measurements as well - which may be helpful in case payload sizes drastically increase. At Etsy I use [Speedcurve’s performance budgets](https://www.speedcurve.com/blog/performance-budgets/) to detect shifts in quantifiable metrics (such as requests, sizes, etc). I often correlate alerts with our RUM data to determine if a third party change resulted in a slowdown, and also add custom metrics for third parties that may present a SPOF risk. In the example below, you can see that the number of blocking scripts and stylesheets are consistently low - but the budget is set so that any shift from the high-water mark will trigger an alert. 


![Speedcurve Performane Budgets](/assets/img/blog/third-parties-and-single-points-of-failure/speedcurve-performance-budgets.jpg){:loading="lazy"}

Speedcurve also provides the ability to [track individual third parties](https://support.speedcurve.com/docs/first-third-parties) as part of their performance budgets - which can be helpful if you have identified a third party that you want to track over time. 

I’m also using [Catchpoint](https://www.catchpoint.com/synthetic-monitoring) to trend third party domains over time. In the graph below, I’m aggregating results from a large number of Chrome synthetic measurements, grouping the results by hostname, and excluding first party domains. This type of reporting provides the ability to monitor all discovered third parties over time; collecting insights into their performance, availability, requests counts and payload sizes. In the example below you can see that one of the third parties experienced a large outage on November 25th, and another experienced a smaller outage on December 5th. Fortunately these were not render blocking and did not impact the user experience.

![Catchpoint Third Party Monitoring](/assets/img/blog/third-parties-and-single-points-of-failure/catchpoint-third-party-monitoring.jpg){:loading="lazy"}

[The RUM Archive](https://rumarchive.com/) is also an interesting project for analyzing third party performance data. This [dataset](https://rumarchive.com/datasets/) comes from approximately 100 of Akamai’s customers mPulse data. While this will not capture as many third parties as we can observe in the HTTP Archive, it does provide the ability to look at their real world performance data! When I combined these two data sources for render blocking third parties, I found 15 third parties that had at least 1 million RUM measurements.   The table below breaks down their DNS, TCP and TTFB times. Many of these third parties have relatively fast DNS times, but there’s a lot of room for improvement when it comes to TCP connection times and TTFB. 

![RUM Archive Third Party Resource Timings](/assets/img/blog/third-parties-and-single-points-of-failure/rumarchive-third-party-resource-timings.jpg){:loading="lazy"}

While the table above illustrates the p75 timings, we can also look at other percentiles as well to get a larger picture of the overall performance impact of these third parties. The graph below illustrates the inter-quartile range for TTFB, with the bars representing the p25 through p75 - essentially 50% of all measurements.    The whiskers represent the p5 and p95. A few third parties stand out for having very poor TTFB, while CookieLaw and Critero seem to have the fastest and most consistent performance.

![RUM Archive Third Party TTFB Percentiles](/assets/img/blog/third-parties-and-single-points-of-failure/rumarchive-third-party-ttfb-percentiles.jpg){:loading="lazy"}

# How to Detect Render-Blocking Third Parties

When you run a test in [WebPageTest](https://www.webpagetest.org/), you can see a yellow circle with an X in the waterfall next to each request that is render-blocking. A similar visual is used in the legacy UI if you are using a [WebPageTest private instance](https://docs.webpagetest.org/private-instances/). If you notice any render blocking third party domains, then make a note of them so that you can run SPOF tests.

![WebPageTest Render Blocking Requests](/assets/img/blog/third-parties-and-single-points-of-failure/wpt-render-blocking-requests.jpg){:loading="lazy"}

There’s also my [Third Party Analyzer](https://tools.paulcalvano.com/wpt-third-party-analysis/) tool, which will highlight SPOF risks based on resources that are render blocking and load before FCP. This works with any WebpageTest private instance tests, Catchpoint WPT shared URLs, or Speedcurve tests. You can read more about this tool [here](https://paulcalvano.com/2024-09-03-discovering-third-party-performance-risks/).

![WebPageTest Render Blocking Requests](/assets/img/blog/third-parties-and-single-points-of-failure/third-party-analyzer-render-blocking-requests.jpg){:loading="lazy"}

If you run a measurement with [DebugBear](https://www.debugbear.com/test/website-speed), you can see a “Blocking” indicator next to every resource that is render-blocking . They also illustrate the priority of each request as well as the domain name.  Similar to the previous examples, if you see render-blocking content for a third party domain name, then make a note of them so that you can run SPOF tests. 

![Debugbear Render Blocking Requests](/assets/img/blog/third-parties-and-single-points-of-failure/debugbear-render-blocking-requests.jpg){:loading="lazy"}

And lastly, you can also see requests via a [Lighthouse](https://developer.chrome.com/docs/lighthouse/overview) test. As with the other examples, just look at the list of requests and determine which of these are third parties.

![Lighthouse Render Blocking Requests](/assets/img/blog/third-parties-and-single-points-of-failure/lighthouse-render-blocking-requests.jpg){:loading="lazy"}

# How to Test for Third Party SPOFs

The most common way of testing for third party single points of failure is to identify which content is render-blocking, and then to test what would happen if that content fails. For years, the easiest way of testing for SPOFs was via WebPageTest as it had a SPOF simulation feature. Today, there are a handful of other methods of testing for them and below I’ll show a few examples. 


## WebPageTest Private Instance

In my earlier example I used a WebPageTest private instance, which has a feature that simulates a SPOF. When you add a hostname to the list, it will run two tests - one without any changes and the other with the hostname routed to a server that silently drops the request (simulating a failure). The results of this test show two filmstrips, and you can easily determine whether the page rendering is stalled when the third party failed to load. 

![WebPageTest Private Instance SPOF Test](/assets/img/blog/third-parties-and-single-points-of-failure/wpt-private-instance-spof-test.jpg){:loading="lazy"}

![WebPageTest Private Instance SPOF Results](/assets/img/blog/third-parties-and-single-points-of-failure/wpt-private-instance-spof-results.jpg){:loading="lazy"}

## Catchpoint WebPageTest

There is also a SPOF feature in the [public WebPagetest](https://webpagetest.org) instance hosted by Catchpoint. At the time of writing this, the SPOF feature is not currently working. Once the feature is fixed, it should work the same as the above example. Until then you can still use WebPageTest to simulate a third party failure by using a [script](https://docs.webpagetest.org/spof/) that overrides DNS for the third party, pointing it to <code>[blackhole.webagetest.org](http://blackhole.webagetest.org)</code>. This emulates how the SPOF feature in WebPageTest works. 

![Catchpoint WebPageTest SPOF Test](/assets/img/blog/third-parties-and-single-points-of-failure/catchpoint-wpt-spof-test.jpg){:loading="lazy"}

## Hosts File Entries

You can use a hosts file to override DNS and route one or more third parties to `blackhole.webpagetest.org`. In order to do this, look up the IP address of the blackhole hostname. At the time of writing, it resolved to `3.219.212.117`. 

Next you can update your `/etc/hosts` file (Mac) or `C:\Windows\System32\drivers\etc\hosts` file (Windows). You can add one or more third party hostnames. Once the browser picks up the new hosts file entries, you’ll be able to test for failure by simply browsing to the site you are testing. 

```
3.219.212.117 cdn.cookielaw.org cdn.optimizely.com
```

## Chrome DevTools

Chrome DevTools has recently added an [individual request throttling feature](https://developer.chrome.com/blog/throttle-individual-network-requests?hl=en) in Chrome. This feature should be enabled with Chrome 144, but if you are using an earlier version (or Chrome Canary) then you can enable an experimental flag to allow individual request throttling. You can find this flag at `chrome://flags/#devtools-individual-request-throttling`. DebugBear shared a great blog post about how to use this feature [here](https://www.debugbear.com/blog/chrome-devtools-throttle-individual-request).

![Chrome DevTools Request Throttling Flag](/assets/img/blog/third-parties-and-single-points-of-failure/chrome-devtools-request-throttling-flag.jpg){:loading="lazy"}

Once enabled, you’ll be able to throttle a request or domain by selecting a network profile for the content. The default options are Block, Fast 4G, Slow 4G and 3G.

![Chrome DevTools Request Throttling](/assets/img/blog/third-parties-and-single-points-of-failure/chrome-devtools-request-throttling.jpg){:loading="lazy"}

To simulate a single point of failure, I wanted something even slower than 3G. So in the network throttling profile section, I defined a profile that adds 10 seconds of latency. When you refresh the page with this profile, you can experience the site in your browser as if the third party was struggling to load the content. 

![Chrome DevTools Request Throttling Profile](/assets/img/blog/third-parties-and-single-points-of-failure/chrome-devtools-request-throttling-profile.jpg){:loading="lazy"}

Try configuring the network throttling for a render blocking third party, and then go to the performance panel in DevTools and capture a trace with filmstrips.    If you see a long gap in page rendering, then you know you have a SPOF! Here’s an example from a popular US ecommerce site I found in the HTTP Archive. It’s loading a JavaScript request for a consent management service at [cdn-ukwest.onetrust.com](cdn-ukwest.onetrust.com). If I slow that down by 10 seconds and measure the page load, I can see that the FCP is also delayed by the same.  Simply adding the async script tag for this third party eliminates the risk of a SPOF!

![Chrome DevTools Request Throttling SPOF Test](/assets/img/blog/third-parties-and-single-points-of-failure/chrome-devtools-request-throttling-spof-test.jpg){:loading="lazy"}

# Conclusion

Third Party Single Points of Failure have been talked about for 15 years now. The problem is well understood, and there’s lots of guidance and testing approaches around this. However, that hasn’t stopped 67% of websites from adding third parties in this way!  

Cloud provider and CDN outages happen, and it’s unfortunate for all involved when they do. However, someone else’s cloud provider outage shouldn’t take down your website. I highly recommend auditing your third party domains to ensure that a slowdown or failure will not result in a disruption of service on your websites. Beyond that, as a general preventive measure - try to self host as much of your render-blocking content as possible.

This article represents my own views and opinions and not those of Etsy.

_Originally published at https://calendar.perfplanet.com/2025/third-parties-and-single-points-of-failure/_


**HTTP Archive queries**

This section provides some details on how this analysis was performed, including SQL queries. Please be warned that some of the SQL queries process a significant amount of bytes - which can be very expensive to run.

<details>
  <summary><b>Number of sites containing render blocking third parties</b></summary>
   This query counts the number of sites that contain a render blocking third party, and also whether they are using a different CDN (or no CDN)!
  <pre><code>
   SELECT
    "Number of Sites" AS category,
    COUNT(DISTINCT page) AS sites
  FROM
    `httparchive.crawl.pages` 
  WHERE
    date = "2025-12-01"
    AND is_root_page = TRUE 
    AND client = "mobile" 
  

UNION ALL

 SELECT
    "Sites with a Render Blocking Third Party" AS category,
    COUNT(DISTINCT page) AS sites
  FROM
    `httparchive.crawl.requests`
  WHERE
    date = "2025-12-01"
    AND is_root_page = TRUE 
    AND client = "mobile" 
    AND NET.REG_DOMAIN(page) != NET.REG_DOMAIN(url)
    AND JSON_VALUE(payload._renderBlocking) = "blocking"

UNION ALL


 SELECT
    "Sites with a Render Blocking Third Party on a different CDN" AS category,
    COUNT(DISTINCT p.page) AS sites
  FROM
    `httparchive.crawl.requests` AS r
    INNER JOIN `httparchive.crawl.pages`  AS p
    ON r.page = p.page
  WHERE
    r.date = "2025-12-01" AND p.date = "2025-12-01"
    AND r.is_root_page = TRUE AND p.is_root_page = TRUE
    AND r.client = "mobile" AND p.client = "mobile"
    AND JSON_VALUE(r.payload._renderBlocking) = "blocking"
    AND NET.REG_DOMAIN(p.page) != NET.REG_DOMAIN(r.url)
    AND JSON_VALUE(p.summary.cdn) != JSON_VALUE(r.payload._cdn_provider)

UNION ALL

SELECT
    "Sites with a Render Blocking Third Party not using a CDN" AS category,
    COUNT(DISTINCT p.page) AS sites
  FROM
    `httparchive.crawl.requests` AS r
    INNER JOIN `httparchive.crawl.pages`  AS p
    ON r.page = p.page
  WHERE
    r.date = "2025-12-01" AND p.date = "2025-12-01"
    AND r.is_root_page = TRUE AND p.is_root_page = TRUE
    AND r.client = "mobile" AND p.client = "mobile"
    AND JSON_VALUE(r.payload._renderBlocking) = "blocking"
    AND NET.REG_DOMAIN(p.page) != NET.REG_DOMAIN(r.url)
    AND (
        JSON_VALUE(r.payload._cdn_provider) IS NULL
        OR JSON_VALUE(r.payload._cdn_provider) = "")

  </code></pre>
</details>

<details>
  <summary><b>Third Party Render Blocking Requests by Content Type</b></summary>
  This summarizes the request types that are used for render blocking third party requests.
  <pre><code>
 SELECT
    r.type AS requestType,
    COUNT(DISTINCT p.page) AS sites,
    COUNT(*) AS requests
  FROM
    `httparchive.crawl.requests` AS r
    INNER JOIN `httparchive.crawl.pages`  AS p
    ON r.page = p.page
  WHERE
    r.date = "2025-12-01" AND p.date = "2025-12-01"
    AND r.is_root_page = TRUE AND p.is_root_page = TRUE
    AND r.client = "mobile" AND p.client = "mobile"
    AND JSON_VALUE(r.payload._renderBlocking) = "blocking"
    AND NET.REG_DOMAIN(p.page) != NET.REG_DOMAIN(r.url)
    AND JSON_VALUE(p.summary.cdn) != JSON_VALUE(r.payload._cdn_provider)
    AND is_main_document = FALSE
GROUP BY 1
ORDER BY 2 DESC
  </code></pre>
</details>

<details>
  <summary><b>Third Party Hostnames w/ Render Blocking Content</b></summary>
  This query breaks down popular third party hostnames that are being used for render blocking content, whith a focus on requests that use a different CDN from the primary website. 
  <pre><code>

 SELECT
    NET.HOST(url) as hostname,
    r.type AS requestType,
    COUNT(DISTINCT p.page) AS sites,
    COUNT(*) AS requests
  FROM
    `httparchive.crawl.requests` AS r
    INNER JOIN `httparchive.crawl.pages`  AS p
    ON r.page = p.page
  WHERE
    r.date = "2025-12-01" AND p.date = "2025-12-01"
    -- root pages
    AND r.is_root_page = TRUE AND p.is_root_page = TRUE
    -- mobile
    AND r.client = "mobile" AND p.client = "mobile"
    -- render blocking requests
    AND JSON_VALUE(r.payload._renderBlocking) = "blocking"
    -- with a different domain name from the page
    AND NET.REG_DOMAIN(p.page) != NET.REG_DOMAIN(r.url)
    -- and a different CDN
    AND JSON_VALUE(p.summary.cdn) != JSON_VALUE(r.payload._cdn_provider)
GROUP BY 1,2
ORDER BY 3 DESC 
  </code></pre>
</details>

<details>
  <summary><b>RawGit usage</b></summary>
  <pre><code>
 
 SELECT
    COUNT(DISTINCT page)
  FROM
    `httparchive.crawl.requests` AS r    
  WHERE
    date = "2025-11-01"
    AND is_root_page = TRUE
    AND client = "mobile" 
  AND NET.HOST(url) LIKE "%rawgit.com"

  </code></pre>
</details>

<details>
  <summary><b>Polyfill.io Usage</b></summary>
  <pre><code>
 
 SELECT
    date,
    COUNT(DISTINCT page)
  FROM
    `httparchive.crawl.requests` AS r
    
  WHERE
    date BETWEEN "2024-01-01" AND "2025-01-01"
    AND is_root_page = TRUE
    AND client = "mobile" 
  AND NET.HOST(url) LIKE "%polyfill.io"
GROUP BY date
ORDER BY date
  </code></pre>
</details>


<details>
  <summary><b>RUM Archive stats for Render Blocking Third Parties</b></summary>
  <pre><code>
 SELECT 
  NET.HOST(url) AS hostname, 
  SUM(fetches) AS freq,

  -- DNS
  PARSE_NUMERIC(`akamai-mpulse-rumarchive.rumarchive.PERCENTILE_APPROX`(
    ARRAY_AGG(DNSHISTOGRAM),
    [0.75],
    10,
    false
  )) / 1000 AS dns_p75,

  PARSE_NUMERIC(`akamai-mpulse-rumarchive.rumarchive.PERCENTILE_APPROX`(
    ARRAY_AGG(DNSHISTOGRAM),
    [0.95],
    10,
    false
  )) / 1000 AS dns_p95,

  PARSE_NUMERIC(`akamai-mpulse-rumarchive.rumarchive.PERCENTILE_APPROX`(
    ARRAY_AGG(DNSHISTOGRAM),
    [0.99],
    10,
    false
  )) / 1000 AS dns_p99,

  -- TCP
  PARSE_NUMERIC(`akamai-mpulse-rumarchive.rumarchive.PERCENTILE_APPROX`(
    ARRAY_AGG(TCPHISTOGRAM),
    [0.75],
    10,
    false
  )) / 1000 AS tcp_p75,

  PARSE_NUMERIC(`akamai-mpulse-rumarchive.rumarchive.PERCENTILE_APPROX`(
    ARRAY_AGG(TCPHISTOGRAM),
    [0.95],
    10,
    false
  )) / 1000 AS tcp_p95,

  PARSE_NUMERIC(`akamai-mpulse-rumarchive.rumarchive.PERCENTILE_APPROX`(
    ARRAY_AGG(TCPHISTOGRAM),
    [0.99],
    10,
    false
  )) / 1000 AS tcp_p99,

  -- TLS
  PARSE_NUMERIC(`akamai-mpulse-rumarchive.rumarchive.PERCENTILE_APPROX`(
    ARRAY_AGG(TLSHISTOGRAM),
    [0.75],
    10,
    false
  )) / 1000 AS tls_p75,

  PARSE_NUMERIC(`akamai-mpulse-rumarchive.rumarchive.PERCENTILE_APPROX`(
    ARRAY_AGG(TLSHISTOGRAM),
    [0.95],
    10,
    false
  )) / 1000 AS tls_p95,

  PARSE_NUMERIC(`akamai-mpulse-rumarchive.rumarchive.PERCENTILE_APPROX`(
    ARRAY_AGG(TLSHISTOGRAM),
    [0.99],
    10,
    false
  )) / 1000 AS tls_p99,

  -- TTFB
  PARSE_NUMERIC(`akamai-mpulse-rumarchive.rumarchive.PERCENTILE_APPROX`(
    ARRAY_AGG(TTFBHISTOGRAM),
    [0.75],
    10,
    false
  )) / 1000 AS ttfb_p75,

  PARSE_NUMERIC(`akamai-mpulse-rumarchive.rumarchive.PERCENTILE_APPROX`(
    ARRAY_AGG(TTFBHISTOGRAM),
    [0.95],
    10,
    false
  )) / 1000 AS ttfb_p95,

  PARSE_NUMERIC(`akamai-mpulse-rumarchive.rumarchive.PERCENTILE_APPROX`(
    ARRAY_AGG(TTFBHISTOGRAM),
    [0.99],
    10,
    false
  )) / 1000 AS ttfb_p99

FROM `akamai-mpulse-rumarchive.rumarchive.rumarchive_resources` 
WHERE date = "2025-12-20" 
  AND NET.HOST(url) IN (
    SELECT
        NET.HOST(url) as hostname,
      FROM
        `httparchive.crawl.requests` AS r
        INNER JOIN `httparchive.crawl.pages`  AS p
        ON r.page = p.page
      WHERE
        r.date = "2025-12-01" AND p.date = "2025-12-01"
        -- root pages
        AND r.is_root_page = TRUE AND p.is_root_page = TRUE
        -- mobile
        AND r.client = "mobile" AND p.client = "mobile"
        -- render blocking requests
        AND JSON_VALUE(r.payload._renderBlocking) = "blocking"
        -- with a different domain name from the page
        AND NET.REG_DOMAIN(p.page) != NET.REG_DOMAIN(r.url)
        -- and a different CDN
        AND JSON_VALUE(p.summary.cdn) != JSON_VALUE(r.payload._cdn_provider)
    GROUP BY 1
    ORDER BY COUNT(DISTINCT p.page) DESC
    )
GROUP BY 1
HAVING SUM(fetches) >= 1000000
ORDER BY 2 DESC;

  </code></pre>
</details>


