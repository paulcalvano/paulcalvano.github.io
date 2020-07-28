---
title: Measuring the Performance of Firefox Quantum with RUM
date: 2017-11-29T14:42:10+00:00
author: Paul Calvano
layout: post
---
On Nov 14th, Mozilla released [Firefox Quantum](https://www.mozilla.org/en-US/firefox/). On launch day, I personally felt that the new version was rendering pages faster and I heard anecdotal reports indicating the same. There have also been a few benchmarks which seem to show that this latest Firefox version is getting content to screens faster than its predecessor. But I wanted to try a different approach to measurement.

Given the vast amount of performance information that we collect at Akamai, I thought it would be interesting to benchmark the performance of Firefox Quantum with a large set of real end-user performance data. The results were dramatic: the new browser improved DOM Content Loaded time by an extremely impressive 24%. Let’s take a look at how those results were achieved.  


<center>
  <br /> <img src="/assets/wp-content/uploads/2017/11/Firefox.jpg" alt="" width="519" height="182" class="alignnone size-full wp-image-172" /><br />
</center>

During my initial analysis, I compared the performance for active browser versions in the wild using RUM (real user monitoring) data from

[Akamai mPulse](https://www.akamai.com/us/en/products/web-performance/mpulse.jsp). The top three Firefox browser versions we observed traffic for immediately after the release were v57 (Quantum), v56 and v52. Note that v52 is the latest [Firefox Extended Support Release (ESR)](https://www.mozilla.org/en-US/firefox/organizations/), and the performance statistics for this version would be skewed by slow connections, firewalls and older machines in university labs and enterprises. This chart shows the findings of this initial analysis, with a red callout to the skewed stats for v52:<img src="/assets/wp-content/uploads/2017/11/Firefox_ESR_Performance.jpg" alt="" width="1467" height="663" class="alignnone size-full wp-image-173"  /> 

Next I decided to dive into some historical performance data comparing the performance of Firefox’s 2017 releases. To do this, first I needed to establish when each browser version reached its peak usage so I could determine the best dates on which to measure each version’s performance.

Using the mPulse RUM data, I summarized the traffic from all Firefox desktop browser versions we saw traffic from between February and November 2017. This gave me insight into the experience of millions of Firefox users and billions of pages served during that seven-month period.

You can see each browser version’s daily traffic percentage distribution below, with the start date of 1/31/17 on the far left and the end date of 11/26/17 on the far right. There are a few interesting things to note here:

  * The latest browser eventually accounts for ~75% to 80% of all Firefox traffic.
  * mozilla uses a throttle mechanism called [Balrog](http://mozilla-balrog.readthedocs.io/en/latest/database.html#rules) to release new versions of their software in throttle phases. In most cases, Firefox appears to reach saturation within a week.
  * Some releases are rolled out faster than others. For example v55 was staggered for days, while v52 and v57 were released much faster.
  * The ESR releases (v45 and v52) have a much slower update cycle and account for approximately 10% of all Firefox traffic.  This means that up to 10% of your users may be running a much older browser!
  * Less than 1% of traffic is running a future version of the browser (Nightly, Dev releases, etc) 

<img src="/assets/wp-content/uploads/2017/11/FirefoxBrowserVersionTrafficHistory_20171126.jpg" alt="" width="1234" height="644" class="alignnone size-full wp-image-174" /> 

In the chart above, we can see that the recent Firefox Quantum v57 release seems to be getting installed at a much faster rate compared to previous browsers. I suspect that this may be due to the excitement that the new release has generated. However, to give an accurate performance comparison, I wanted to wait for the browser to reach saturation, with approximately 70% of users on v57. The two-week chart below shows that the saturation was reached six days after the release, when v57 hit 72%.<img src="/assets/wp-content/uploads/2017/11/FirefoxQuantumRelease_20171126.jpg" alt="" width="1186" height="644" class="alignnone size-full wp-image-175" /> 

In order to provide an apples-to-apples comparison of the page-load performance data, I used the browser version history data to determine which dates to compare traffic from. Then I selected a representative day from each release to analyze. I used the same day of the week (Monday) to avoid skews in traffic patterns from impacting the results. I also decided to use 11/20 instead of 11/27 so that the results would not be skewed by Cyber Monday traffic. And since we’re comparing performance experienced by millions of users on multiple days, I decided to illustrate the data based on percentile histograms to illustrate the timing differences clearly. I’m also looking at both the DOM Content Loaded metric as well as the onLoad metric for reasons I’ll discuss below.

_Note: you may be wondering why I’m not evaluating this based on metrics such as the First Meaningful Paint or Time to Interactive. While these metrics would definitely be an excellent way to compare browser performance improvements, the metrics are not yet instrumented in Firefox so we need to work with what has been available historically._

[The DOM Content Loaded metric](https://developer.mozilla.org/en-US/docs/Web/Events/DOMContentLoaded) is fired when the initial HTML document has been completely loaded and parsed, without waiting for stylesheets, images, and subframes to finish loading. This metric helps us understand how quickly the browser is parsing the page. Firefox Quantum v57 appears to be significantly moving the needle on this metric (see chart below), which may explain why the pages appear faster to render. One other thing worth noting in this chart, which shows DOM Content Loaded times for six different Firefox versions: the Mozilla team appears to have been improving with each release.<img src="/assets/wp-content/uploads/2017/11/Firefox_Version_DOMContentLoadedTimes.jpg" alt="" width="1473" height="616" class="alignnone size-full wp-image-176" /> 

The onLoad metric measures the time from the navigation start until the onLoad event is fired, which is when a page is considered ready. Based on the performance data we’ve analyzed, we are not seeing a significant difference in time to onLoad with Quantum, although load times have dropped 10% since Firefox v52 likely due to improvements earlier in the year.<img src="/assets/wp-content/uploads/2017/11/Firefox_Version_PageLoadTimes.jpg" alt="" width="1492" height="608" class="alignnone size-full wp-image-177" /> 

Looking at the DOM Content Loaded times again in the spreadsheet format below, we can see that the median time decreased from 2.969 seconds to 2.262 seconds since v52. That’s an extremely impressive 24% performance improvement.<img src="/assets/wp-content/uploads/2017/11/Firefox_Version_LoadTimeTable_updated-1.jpg" alt="" width="756" height="161" class="alignnone size-full wp-image-178" /> 

So the obvious question now is, _“Why has DOM Content Loaded improved but not Page Load Time?”_ I suspect that the devil lies in the details for each site being measured, and that some of the complexity built into today’s websites will cause delays on any browser that loads them. It’s important to note that the DOM Content Loaded metric shows a definitive improvement in performance during a critical part of the page load. And since users feel like pages are loading faster, this metric may be ultimately related to that; perhaps we’ll tackle this question in a future blog post.

**Conclusion** Progressive performance improvements like this do not happen by accident; they are the clear result of a dedicated team working to make their product faster and more efficient. Firefox Quantum v57 is a great step forward for Firefox, and I’m excited to see what’s coming next. Congratulations to the entire Mozilla team for doing such a great job in making the web faster!

_Originally published at <https://developer.akamai.com/blog/2017/11/29/measuring-performance-firefox-quantum-rum/>_