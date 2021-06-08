---
layout: post
title: "What can the HTTP Archive tell us about Largest Contentful Paint?"
date: 2021-06-07 12:30:00 -0400
related_posts:
 
  - _posts/2017-08-31-exploring-relationships-between-performance-metrics-in-http-archive-data.md
  - _posts/2018-07-02-impact-of-page-weight-on-load-time.md
  - _posts/2019-01-11-correlating-performance-metrics-to-page-characteristics.md

---


[Largest Contentful Paint (LCP)](https://web.dev/lcp/) is an important metric that measures when the largest element in the browser’s viewport becomes visible. This could be an image, a background image, a poster image for a video, or even a block of text. The metric is measured with the [Largest Contentful Paint API](https://wicg.github.io/largest-contentful-paint/), which is [supported](https://caniuse.com/?search=largestcontentfulpaint) in Chromium browsers. Optimizing for this metric is critical to end user experience, since it affects their ability to visualize your content.

Google has promoted this metric as one of the three ["Core Web Vitals"](https://web.dev/vitals/) that affect user experience on the web. It is also slated to become a [search ranking signal over the next few weeks](https://developers.google.com/search/blog/2021/04/more-details-page-experience), which has created a lot of awareness about it. The suggested target for a good Largest Contentful Paint is less than 2.5 seconds for at least 75% of page loads.

![Largest Contentful Paint Overview](/assets/img/blog/lcp-httparchive/image9.jpg ){:loading="lazy"}

<small>Source: <a href="https://web.dev/lcp/">https://web.dev/lcp/</a></small>

Some of the recent posts on [WPOStats](https://wpostats.com/tags/core%20web%20vitals/) feature interesting case studies about this metric.  For example, 


*   Google's [research](https://blog.chromium.org/2020/05/the-science-behind-web-vitals.html) found that when Core Web Vitals are met, users are 24% less likely to abandon a page before it finishes loading.
*   Vodafone improved LCP by 31% and saw an 8% increase in sales.
*   NDTV improved their LCP by 55% and saw a 50% reduction in bounce rate.
*   Tokopedia improved their LCP by 55% and saw a 23% increase in session duration.

**Identifying the Largest Contentful Paint Element**

The name of this metric implies that size is used as a proxy for importance. Because of this, you may be wondering specifically which image or text triggered it as well as the percentage of the viewport it consumed. There are a few ways to examine this:

One way to visualize the Largest Contentful Paint is to look at a [WebPageTest](https://webpagetest.org/) filmstrip. You’ll be able to see when visual changes occurred (yellow outline) as well as when the Largest Contentful Paint event occurred (red outline).

![WebPageTest Filmstrip showing LCP Element](/assets/img/blog/lcp-httparchive/image7.jpg ){:loading="lazy"}

In Chrome DevTools, you can also click on the LCP indicator in the “Performance” tab to examine the Largest Contentful Paint element in your browser. Using this method you can see and inspect the exact element (image, text, etc) that triggered it.

![Chrome DevTools Performance Tab](/assets/img/blog/lcp-httparchive/image11.gif ){:loading="lazy"}

Lighthouse also has an audit that identifies the Largest Contentful Paint element. If you examine the screenshot below you’ll notice that there is a yellow box around the largest element, as well as an HTML snippet.

![Lighthouse LCP Element](/assets/img/blog/lcp-httparchive/image5.jpg ){:loading="lazy"}

**How Large is the Largest Contentful Paint?**

The [HTTP Archive](https://httparchive.org/) runs Lighthouse audits for approximately 7.2 million websites every month. In the May 2021 dataset, Lighthouse was able to identify an LCP element in 97.35% of the tests. Since we have the ability to query all of these Lighthouse test results, we can analyze the result of the LCP audits and get more insight into what drives this metric across the web. 

Using the same boundaries that Lighthouse uses to draw the rectangle around the LCP element, it’s possible to calculate the area of it. In the above example, the product of the LCP image’s height (191) and width (340) was 64,940 pixels. Since the Lighthouse test was run with an emulated [Moto G4 user agent](https://almanac.httparchive.org/en/2020/methodology#webpagetest) with a screen size of 640x360, we can also calculate that this particular LCP image took up 28% of the viewport.

The graph below shows the cumulative distribution of the LCP element as a percentage of screen size. The median LCP element takes up 31% of the screen size! At the 75th percentile the LCP element is nearly twice as large, taking up 59% of the screen size. Additionally 10.6% of sites actually had an LCP element that exceeded the viewport (which is why the y axis doesn’t reach 100%).

![Distribution of LCP Element Size as a Percent of Screen Size](/assets/img/blog/lcp-httparchive/image10.jpg ){:loading="lazy"}

The graph below illustrates the same data in a histogram. From this we can see that 4.03% of sites (285,751) had a LCP element that took up 0 pixels. Upon further inspection, the 0 pixel elements appear to have been used in carousels, so by the time the audit completed the LCP element slid out of the viewport.

![Histogram of LCP Element Size as a Percent of Screen Size](/assets/img/blog/lcp-httparchive/image3.jpg ){:loading="lazy"}

**Node Paths of LCP Elements**

Another interesting aspect of the Largest Contentful Paint audit is the nodePath of the element, which shows you where in the DOM this element was. In the example we looked at earlier, the nodePath was: ```
```
1,HTML,1,BODY,8,DIV,2,SECTION,1,DIV,0,DIV,0,DIV,0,UL,0,LI,0,ARTICLE,1,DIV,0,DIV,0,A,0,IMG
```

If we look at the last element in the node path, we can get some insight into the type of element that triggered the Largest Contentful Paint. The most common node that triggered the Largest Contentful Paint was &lt;IMG>, which accounted for 42% of all sites.   Next was &lt;DIV> at 27% (which could include text or images). The &lt;H1> through &lt;H5> header elements accounted for 7.18% of all Largest Contentful Paints.  


|---                            |---            |---              |
|LCP Node (last element in path)|Number of Sites|Percent of Sites |
|IMG                            |3,067,354      |42.12%           |
|DIV                            |1,981,416      |27.21%           |
|P                              |766,977        |10.53%           |
|H1                             |291,091        |4.00%            |
|                               |192,498        |2.64%            |
|SECTION                        |182,267        |2.50%            |
|H2                             |144,534        |1.98%            |
|A                              |107,501        |1.48%            |
|SPAN                           |85,245         |1.17%            |
|HEADER                         |67,762         |0.93%            |
|LI                             |64,212         |0.88%            |
|H3                             |60,679         |0.83%            |
|RS-SBG                         |51,623         |0.71%            |
|TD                             |48,470         |0.67%            |
|H4                             |19,039         |0.26%            |
|VIDEO                          |15,649         |0.21%            |
|ARTICLE                        |12,860         |0.18%            |
|FIGURE                         |9,208          |0.13%            |
|BODY                           |8,859          |0.12%            |
|image                          |8,077          |0.11%            |
|CENTER                         |7,960          |0.11%            |

The &lt;VIDEO> element only accounted for 0.21% of sites. According to the Web Almanac, [the &lt;video> element was used on 0.49% of mobile websites](https://almanac.httparchive.org/en/2020/media#videos) - so from this we can estimate that half of sites loading videos are triggering LCP with video poster images.

**Image Weight for the LCP**

One of the Lighthouse audits looks for opportunities to preload the Largest Contentful Paint element, and estimates the potential savings in performance. This audit also identifies the URL for the LCP element - which can give us some insights into what type of images are being loaded as a LCP element. In the HTTP Archive data, only 67% of the Lighthouse tests were able to identify a URL for an LCP element. Based on this, we can infer that text nodes are used for the LCP on approximately 33% of sites.

![Lighthouse Preload LCP Element Recommendation](/assets/img/blog/lcp-httparchive/image8.jpg ){:loading="lazy"}

The graph below shows the distribution of sizes for the image element that was associated with the Largest Contentful Paint. The median LCP element size was 80KB. At the 90th percentile, the LCP element size was 512KB.   If you have a large LCP image then you should consider optimizing it before you attempt to follow the Lighthouse preload recommendation.

![Distribution of LCP Element Size](/assets/img/blog/lcp-httparchive/image12.jpg ){:loading="lazy"}

Additionally, 70% of the LCP element images were JPEG and 25% were PNG.  Only 3% of sites served a webp as their LCP element.


|---    |---        |---        |
|format |sites      |% of Sites |
|jpg    |3,161,991  |69.37%     |
|png    |1,122,585  |24.63%     |
|webp   |141,441    |3.10%      |
|gif    |84,829     |1.86%      |
|svg    |34,123     |0.75%      |
|Other  |13,272     |0.29%      |



When we look at the LCP element as a percentage of page weight, we can see that the median LCP element is 4.17% of the total page weight. At the higher percentiles, the LCP elements are larger and also a larger percentage of page weight.

![LCP Element as a Percent of Page Weight](/assets/img/blog/lcp-httparchive/image1.jpg ){:loading="lazy"}

|---       |---           |---      |---      |---                      |
|percentile|ImageRequests |ImageKB  |TotalKB  |LCP as a % of Page Weight|
|p25       |15            |422      |1,138    |3.01%                    |
|p50       |26            |1,142    |2,185    |4.17%                    | 
|p75       |45            |2,692    |4,108    |5.58%                    |
|p95       |103           |8,008    |10,036   |8.42%                    |


Since images account for 52% of the median page weight (for the sites that have a LCP image element), we can infer that at the median 8% of page weight is used to render content to 31% of the screen. 

**How does this change based on Site Popularity?**

The HTTP Archive now contains rank groupings, obtained from the Chrome User Experience Report.   This can enable us to segment this analysis based on the popularity of sites.  The rank grouping indicator buckets sites into the top 1K, 10K, 100K, 1 million and 10 million. 

When we look at the Largest Contentful Paint image size based on popularity, it’s interesting to note that the most popular sites tend to be serving smaller images for the LCP element. While there may be numerous reasons for this, I suspect that the more popular sites are investing in image optimization solutions.

![LCP Image Size by Site Popularity](/assets/img/blog/lcp-httparchive/image2.jpg ){:loading="lazy"}

Page weight follows the same pattern, with the least popular websites having some of the largest page weights. If we look at the LCP element based on the percentage of page weight, you can see that within the top 100K sites the ratios are very close. In the less popular sites, the LCP element tends to be a much greater percentage of page weight.


|---              |---    |---    |---    |---  |
|rank             |p25    |p50    |p75    |p95  |
|Top 1k           |1.61%  |2.12%  |2.85%  |5.67%|
|Top 10k          |1.76%  |2.27%  |3.00%  |4.96%|
|Top 100k         |2.07%  |2.87%  |3.77%  |5.78%|
|Top 1 million    |2.53%  |3.49%  |4.60%  |6.95%|
|Top 10 million   |3.11%  |4.30%  |5.75%  |8.65%|

We can also make some interesting observations about how popular sites are optimizing their LCP assets. Looking at the various image formats, JPG images are the most common LCP element. Some other formats such as PNG, WebP, GIF and SVG are used more frequently in the more popular sites. 

![Largest Contentful Paint Element Format by Rank](/assets/img/blog/lcp-httparchive/image6.jpg ){:loading="lazy"}

**Conclusion**

Largest Contentful Paint is an important metric that helps illustrate when a page’s most significant content is rendered to the screen. In reviewing the HTTP Archive data, we can see that this area represents between 30% and 60% of a mobile viewport for a majority of sites.  

There are a shocking number of sites that have a LCP element that consumes a large percentage of the viewport and are delivered as large unoptimized images. Site owners should evaluate both what is triggering the Largest Contentful Paint as well as how it is loaded. Optimizing for the Largest Contentful Paint will ensure that the browser has the opportunity to load and render this content as quickly as possible.

If you are interested in seeing some of the SQL queries and raw data used in this analysis, I’ve created a post with all the details in the [HTTP Archive discussion forums](https://discuss.httparchive.org/t/analyzing-largest-contentful-paint-stats-via-lighthouse-audits/2166). You can also see all the data used for these graphs in this [Google Sheet](https://docs.google.com/spreadsheets/d/1fI_16nby3Yn1LHxWVd4QRyOuPqqMLmBvBU31l5kGF-8/edit?usp=sharing).
