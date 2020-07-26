---
title: Impact of Page Weight on Load Time
date: 2018-07-02T16:31:01+00:00
author: Paul Calvano
layout: post
---
Over the years it has been fun to track website page weight by comparing it to milestones such as the size of a floppy disk (1.44MB), the size of the original install size of DOOM (2.39MB) and when it hit 3MB last summer.

[<img src="/assets/wp-content/uploads/2018/05/avg_page_weight_floppy_doom_.jpg" alt="" width="646" height="336" class="alignnone size-full wp-image-460" srcset="http://paulcalvano.com/wp-content/uploads/2018/05/avg_page_weight_floppy_doom_.jpg 646w, http://paulcalvano.com/wp-content/uploads/2018/05/avg_page_weight_floppy_doom_-300x156.jpg 300w" sizes="(max-width: 646px) 100vw, 646px" />](http://paulcalvano.com/wp-content/uploads/2018/05/avg_page_weight_floppy_doom_.jpg)

<!--more-->

When we talk about page weight, we are often talking about high resolution images, large hero videos, excessive 3rd party content, JavaScript bloat &#8211; and the list goes on.

I recently did some research to show that [sites with more 3rd party content are more likely to be slower](https://discuss.httparchive.org/t/analyzing-3rd-party-performance-via-http-archive-crux/1359). And then a few days later [USAToday showed us an extreme example](https://twitter.com/paulcalvano/status/1000094333524201473) by publishing a GDPR friendly version of their site for EU visitors. The EU version has no 3rd party content, substantially less page weight and is blazing fast compared to the US version.

[<img src="/assets/wp-content/uploads/2018/05/usatoday_eu_us.jpg" alt="" width="1200" height="731" class="alignnone size-full wp-image-447" srcset="http://paulcalvano.com/wp-content/uploads/2018/05/usatoday_eu_us.jpg 1200w, http://paulcalvano.com/wp-content/uploads/2018/05/usatoday_eu_us-300x183.jpg 300w, http://paulcalvano.com/wp-content/uploads/2018/05/usatoday_eu_us-768x468.jpg 768w, http://paulcalvano.com/wp-content/uploads/2018/05/usatoday_eu_us-1024x624.jpg 1024w, http://paulcalvano.com/wp-content/uploads/2018/05/usatoday_eu_us-700x426.jpg 700w" sizes="(max-width: 1200px) 100vw, 1200px" />](http://paulcalvano.com/wp-content/uploads/2018/05/usatoday_eu_us.jpg)

Shortly after the 3MB average page weight milestone was reached last summer, I did some [analysis](https://discuss.httparchive.org/t/tracking-page-weight-over-time/1049) to try and understand the sudden jump in page weight. It turns out that the largest 5% of pages were influencing the average, which is a perfect example of averages misleading us.

<img src="/assets/wp-content/uploads/2018/03/ha_pageweight.jpg" alt="" width="690" height="274" class="alignnone size-full wp-image-287" srcset="http://paulcalvano.com/wp-content/uploads/2018/03/ha_pageweight.jpg 690w, http://paulcalvano.com/wp-content/uploads/2018/03/ha_pageweight-300x119.jpg 300w" sizes="(max-width: 690px) 100vw, 690px" /> 

These days we focus more on percentiles, histograms and statistical distributions to represent page weight. For example, in the image below you can see how this is being represented [in the recently redesigned HTTP Archive reports](https://httparchive.org/reports/state-of-the-web#bytesTotal).

[<img src="/assets/wp-content/uploads/2018/05/page-weight-new-http-archive.jpg" alt="" width="1165" height="837" class="alignnone size-full wp-image-463" srcset="http://paulcalvano.com/wp-content/uploads/2018/05/page-weight-new-http-archive.jpg 1165w, http://paulcalvano.com/wp-content/uploads/2018/05/page-weight-new-http-archive-300x216.jpg 300w, http://paulcalvano.com/wp-content/uploads/2018/05/page-weight-new-http-archive-768x552.jpg 768w, http://paulcalvano.com/wp-content/uploads/2018/05/page-weight-new-http-archive-1024x736.jpg 1024w, http://paulcalvano.com/wp-content/uploads/2018/05/page-weight-new-http-archive-700x503.jpg 700w" sizes="(max-width: 1165px) 100vw, 1165px" />](http://paulcalvano.com/wp-content/uploads/2018/05/page-weight-new-http-archive.jpg)

**_Is page weight still something we should care about in 2018?_**

Thanks to the [HTTP Archive](https://httparchive.org/), we have the ability to analyze how the web is built. And by combining it with the [Chrome User Experience Report (CrUX)](https://developers.google.com/web/tools/chrome-user-experience-report/), we can also see how this translates into the actual end user experiences across these sites. This is an extremely powerful combination.

_Note: If you are not familiar with CrUX, it is Real User Measurement data collected by Google from Chrome users who have opted-in to syncing their browsing history and have usage statistic reporting enabled. I [wrote about CruX](https://paulcalvano.com/index.php/2018/04/26/using-googles-crux-to-compare-your-sites-rum-data-w-competitors/) and created some [overview videos](https://paulcalvano.com/index.php/2018/05/06/tutorial-using-bigquery-to-analyze-chrome-user-experience-report-data/) if you are interested in learning more about the data and how to work with it._

By leveraging CrUX and the HTTP Archive together, we can analyze performance across many websites and look for trends. For example, below you can see how often the Alexa top 10 sites are able to load pages in less than 2 seconds, 2-4 seconds, 4-6 seconds and greater than 6 seconds. It&#8217;s easy to glance that this chart and see which sites have a large percentage of slower pages. I wrote a another post about how we can use CrUX data like this to [compare yourself to competitors](https://paulcalvano.com/index.php/2018/04/26/using-googles-crux-to-compare-your-sites-rum-data-w-competitors/).

[<img src="/assets/wp-content/uploads/2018/07/alexa-10.jpg" alt="" width="1111" height="372" class="alignnone size-full wp-image-472" srcset="http://paulcalvano.com/wp-content/uploads/2018/07/alexa-10.jpg 1111w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-10-300x100.jpg 300w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-10-768x257.jpg 768w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-10-1024x343.jpg 1024w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-10-700x234.jpg 700w" sizes="(max-width: 1111px) 100vw, 1111px" />](http://paulcalvano.com/wp-content/uploads/2018/07/alexa-10.jpg)

But what happens if we look at the real user performance for 1,000 popular sites this way? The results are oddly symmetrical, with almost as many fast sites as slow ones. In the graph below I sorted the onLoad metrics from fast (left) to slow (right). There are 1000 tiny bars &#8211; each representing a summary of a single website&#8217;s real user experiences on Chrome browsers. The most consistently fast site in this list is the webcomic XKCD &#8211; with an impressive 93.5% of users loading pages in < 2 seconds. Some other sites in the &#8220;extremely fast&#8221; category are Google, Bing, CraigsList, Gov.uk, etc. Many of the slow sites (far right of this graph) have large page weights, videos, advertisements and numerous 3rd parties. Where do you think your site&#8217;s performance stacks up?

[<img src="/assets/wp-content/uploads/2018/07/alexa-1000-onload_may2018.jpg" alt="" width="1218" height="568" class="alignnone size-full wp-image-490" srcset="http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-onload_may2018.jpg 1218w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-onload_may2018-300x140.jpg 300w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-onload_may2018-768x358.jpg 768w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-onload_may2018-1024x478.jpg 1024w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-onload_may2018-700x326.jpg 700w" sizes="(max-width: 1218px) 100vw, 1218px" />](http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-onload_may2018.jpg)

Since we&#8217;re interested in investigating the relationship of page weight to performance, let&#8217;s look at the top 1000 pages that are less than 1MB and the top 1000 pages that are greater than 3MB. The pattern in load times is quite revealing. A few notable observations:

  * The distribution of fast vs slow seems to be cut down the middle. Sites appear to either be mostly fast or mostly slow
  * There are far more fast <1MB pages compared to slow ones
  * There are far more slow >3MB pages compared to fast ones.
  * The fact that there are still some fast >3MB pages and slow <1MB pages proves that page weight isn&#8217;t everything, and it is possible to optimize rich experiences for performance.

[<img src="/assets/wp-content/uploads/2018/07/alexa-1000-small-vs-large.jpg" alt="" width="1534" height="762" class="alignnone size-full wp-image-478" srcset="http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-small-vs-large.jpg 1534w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-small-vs-large-300x149.jpg 300w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-small-vs-large-768x381.jpg 768w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-small-vs-large-1024x509.jpg 1024w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-small-vs-large-700x348.jpg 700w" sizes="(max-width: 1534px) 100vw, 1534px" />](http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-small-vs-large.jpg)

_Note: I&#8217;ve also applied the same logic to the top 10,000 sites, and the pattern was identical._

**What About the Other Metrics that CrUX Collects?**

Since CrUX contains additional metrics, I also looked at the relationship of page weight to DOM Content Loaded and First Contentful Paint. The set of graphs below compare the fastest range (<2s for onLoad, <1s for FCP and DCL) for the top 1000 sites. Across these three metrics, we see the highest correlation of load times to page weight with the onLoad metric.

[<img src="/assets/wp-content/uploads/2018/07/alexa-1000-onload-fcp-dcl.jpg" alt="" width="1333" height="735" class="alignnone size-full wp-image-480" srcset="http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-onload-fcp-dcl.jpg 1333w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-onload-fcp-dcl-300x165.jpg 300w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-onload-fcp-dcl-768x423.jpg 768w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-onload-fcp-dcl-1024x565.jpg 1024w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-onload-fcp-dcl-700x386.jpg 700w" sizes="(max-width: 1333px) 100vw, 1333px" />](http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-onload-fcp-dcl.jpg) _(Note: the higher %s mean that more pages experienced faster load times. So higher=better in these graphs.)_

**What Aspects of Page Weight Impacts Performance the Most?**

We’ve seen a strong correlation of performance to page weight, and we’ve learned that this is more measurable via onLoad vs First Contentful Paint. But what contributing factors of page weight impact load times the most?

If we examine the Top 1000 sites again and pull some of the page weight statistics from the HTTP Archive, we can once again compare HTTP Archive data w/ CrUX. The graphs below summarize the percentage of pages with onLoad times less than 2 seconds. The Y axis is the median number of bytes, and the X axis represents the percentage of sites with fast page loads.

[<img src="/assets/wp-content/uploads/2018/07/alexa-1000-bloat_images_css_js.jpg" alt="" width="880" height="549" class="alignnone size-full wp-image-481" srcset="http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-bloat_images_css_js.jpg 880w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-bloat_images_css_js-300x187.jpg 300w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-bloat_images_css_js-768x479.jpg 768w, http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-bloat_images_css_js-700x437.jpg 700w" sizes="(max-width: 880px) 100vw, 880px" />](http://paulcalvano.com/wp-content/uploads/2018/07/alexa-1000-bloat_images_css_js.jpg)

In the top left graph, page weight shows a strong correlation, and the sites with less fast pages tended to have larger page weights. The remaining 3 graphs show how JavaScript, CSS and Images contribute to page weight and performance. Based on these graphs, Images and JavaScript are the most significant contributors to the page weights that affect load time. And for some slow sites, the amount of compressed JavaScript actually exceeds the number of image bytes!

**Conclusion**

Page weight is an important metric to track, but we should always consider using appropriate statistical methods when tracking it on the web as a whole. Certainly track it for your sites &#8211; because as you&#8217;ve seen here, the size of content does matter. It&#8217;s not the only thing that matters &#8211; but the correlation is strong enough that a spike in page weight should merit some investigation.

If your site has a page weight problem, there are a few things you do can do:

  * [Akamai&#8217;s Image Manager](https://www.akamai.com/us/en/solutions/why-akamai/image-management.jsp) can help to optimize images with advanced techniques such as perceptual quality compression. This is also a great way to ensure that you don&#8217;t get any surprises when a marketing promo drops an 2MB hero image on your homepage. 
  * Limit the use of large video files, or defer their loading to avoid critical resources competing for bandwidth. Check out Doug Sillars&#8217; [blog post on videos embedded into web pages](https://dougsillars.com/2018/01/04/video-on-the-mobile-web-a-performance-study/).
  * Lazy load images that are not in the viewport of your screen. [Jeremy Wagner wrote a nice guide](https://developers.google.com/web/fundamentals/performance/lazy-loading-guidance/images-and-video/) on this recently. 
  * Ensure that you are compressing text based content. Gzip compression at a minimum should be enabled. [Brotli](https://opensource.googleblog.com/2015/09/introducing-brotli-new-compression.html) compression is [widely supported](https://caniuse.com/#feat=brotli) can help reduce content size further. (Akamai Ion customers can automatically serve Brotli compressed resources via [Resource Optimizer](https://developer.akamai.com/blog/2017/11/17/automatically-optimize-size-critical-page-resources/))
  * Use Lighthouse and Chrome Dev Tools to audit your pages. [Find unused CSS and JS with the Coverage feature](https://developers.google.com/web/updates/2017/04/devtools-release-notes#coverage) and attempt to optimize.
  * Audit your 3rd parties. Many sites do not realize how much content their 3rd parties add to their site and how inconsistent their performance may become as a result. [Harry Roberts wrote a helpful guide here!](https://csswizardry.com/2018/05/identifying-auditing-discussing-third-parties/). Also, Akamai&#8217;s [Script Manager](https://community.akamai.com/customers/s/article/New-Ion-Adaptive-Acceleration-features-Turn-them-on-now?language=en_US) service can help to manage third parties based on performance. 
  * Track your sites page weight over time and alert on increases. If you use [Akamai&#8217;s mPulse](https://www.akamai.com/us/en/products/web-performance/mpulse-real-user-monitoring.jsp) RUM service &#8211; you can do this with resource timing data ([if TAO is permitted](http://www.lognormal.com/blog/2017/07/25/a-study-of-timing-allow-origin/)).

_Thanks to Yoav Weiss and Ilya Grigorik for reviewing this and providing feedback._