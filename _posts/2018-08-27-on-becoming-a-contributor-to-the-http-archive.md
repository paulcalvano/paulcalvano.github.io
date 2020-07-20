---
title: On Becoming a Contributor to the HTTP Archive
date: 2018-08-27T02:40:16+00:00
author: Paul Calvano
layout: post

---
The [HTTP Archive](https://httparchive.org/) is an open source project that tracks how the web is built. Twice a month it crawls 1.3 million web pages on desktop and emulated mobile devices, and collects technical information about each of the web pages. That information is then aggregated and made available in [curated reports](https://httparchive.org/reports). The raw data is also made [available via Google BigQuery](https://github.com/HTTPArchive/legacy.httparchive.org/blob/master/docs/bigquery-gettingstarted.md), which makes answering interesting questions about the web accessible to anyone with some knowledge of SQL as well as the curiosity to dig in.

[<img src="http://paulcalvano.com/wp-content/uploads/2018/08/httparchive.jpg" alt="" width="397" height="215" class="aligncenter size-full wp-image-537" srcset="http://paulcalvano.com/wp-content/uploads/2018/08/httparchive.jpg 397w, http://paulcalvano.com/wp-content/uploads/2018/08/httparchive-300x162.jpg 300w" sizes="(max-width: 397px) 100vw, 397px" />](https://httparchive.org)

<!--more-->

When [Steve Souders](https://twitter.com/Souders) created the project back in 2010, it included far less pages &#8211; but it was immensely valuable to the community. As [sponsorship](https://httparchive.org/about#sponsors) increased so did the infrastructure and the ability to do more with it. Over time more and more information was added to the archive &#8211; including HAR files, Lighthouse reports and even response bodies.

[<img src="http://paulcalvano.com/wp-content/uploads/2018/08/httparchive_sponsors.jpg" alt="" width="1157" height="365" class="aligncenter size-full wp-image-538" srcset="http://paulcalvano.com/wp-content/uploads/2018/08/httparchive_sponsors.jpg 1157w, http://paulcalvano.com/wp-content/uploads/2018/08/httparchive_sponsors-300x95.jpg 300w, http://paulcalvano.com/wp-content/uploads/2018/08/httparchive_sponsors-768x242.jpg 768w, http://paulcalvano.com/wp-content/uploads/2018/08/httparchive_sponsors-1024x323.jpg 1024w, http://paulcalvano.com/wp-content/uploads/2018/08/httparchive_sponsors-700x221.jpg 700w" sizes="(max-width: 1157px) 100vw, 1157px" />](https://httparchive.org/about#sponsors)

In 2017 [Ilya Grigorik](https://twitter.com/igrigorik), [Patrick Meenan](https://twitter.com/patmeenan) and [Rick Viscomi](https://twitter.com/rick_viscomi) started maintaining the project. They have done some amazing work overhauling the new website, creating new and useful reports and continuing to push the envelope on what the HTTP Archive is capable of providing to the web community. As of last week I&#8217;ve joined Ilya, Pat and Rick as a co-maintainer of the HTTP Archive, and I couldn&#8217;t be more excited!

[<img src="http://paulcalvano.com/wp-content/uploads/2018/08/paul_httparchive.jpg" alt="" width="542" height="279" class="aligncenter size-full wp-image-546" srcset="http://paulcalvano.com/wp-content/uploads/2018/08/paul_httparchive.jpg 542w, http://paulcalvano.com/wp-content/uploads/2018/08/paul_httparchive-300x154.jpg 300w" sizes="(max-width: 542px) 100vw, 542px" />](https://twitter.com/HTTPArchive/status/1032407393320218624)

**So how have I been using the HTTP Archive?**

Rarely does a week go by where someone doesn&#8217;t ask a question or share a news article that doesn&#8217;t provoke a question that can be answered with the archive. I love diving deep into questions about the web, and many of my colleagues joke about &#8220;nerd sniping Paul&#8221;. Fortunately no Paul&#8217;s have been injured using BigQuery :).

[<img src="http://paulcalvano.com/wp-content/uploads/2018/08/nerdsniping.jpg" alt="" width="753" height="456" class="aligncenter size-full wp-image-539" srcset="http://paulcalvano.com/wp-content/uploads/2018/08/nerdsniping.jpg 753w, http://paulcalvano.com/wp-content/uploads/2018/08/nerdsniping-300x182.jpg 300w, http://paulcalvano.com/wp-content/uploads/2018/08/nerdsniping-700x424.jpg 700w" sizes="(max-width: 753px) 100vw, 753px" />](https://xkcd.com/356/) <small><em>Source: <a href="https://xkcd.com/356/">https://xkcd.com/356/</a></em></small>

Over the past few months, I&#8217;ve been sharing some of my research on the [HTTP Archive Discussion forums](https://discuss.httparchive.org/). An example of a recent post was just a few days ago when the [Blink-Dev team announced that the Application Cache was being deprecated in Chrome](https://groups.google.com/a/chromium.org/forum/#!msg/blink-dev/FvM-qo7BfkI/0daqyD8kCQAJ). It only took a few minutes to write a SQL query to identify sites that are still using this feature. After doing some analysis on the data, I wound up sharing my research with the blink team so that they could track this. Since I work at Akamai, I&#8217;m also planning to give a proactive heads up to the customers whose sites will be affected. Being able to quickly notify numerous websites of an important change that might impact their business is truly a priceless use case.

[<img src="http://paulcalvano.com/wp-content/uploads/2018/08/appcache.jpg" alt="" width="529" height="290" class="aligncenter size-full wp-image-540" srcset="http://paulcalvano.com/wp-content/uploads/2018/08/appcache.jpg 529w, http://paulcalvano.com/wp-content/uploads/2018/08/appcache-300x164.jpg 300w" sizes="(max-width: 529px) 100vw, 529px" />](https://twitter.com/paulcalvano/status/1032862176762048512)

At the [2018 Fluent Conference in San Jose, CA](https://conferences.oreilly.com/fluent/fl-ca) this past June, I&#8217;ve shared a few additional examples of how I&#8217;ve used the HTTP Archive at Akamai. You can see the slides [here](https://www.slideshare.net/PaulCalvano/fluent-2018-tracking-performance-of-the-web-with-http-archive-102455732), where I talk about how I used the archive to help improve configuration defaults, assist in product research and even security notifications.

[<img src="http://paulcalvano.com/wp-content/uploads/2018/08/fluent_talk_slide.jpg" alt="" width="634" height="348" class="aligncenter size-full wp-image-536" srcset="http://paulcalvano.com/wp-content/uploads/2018/08/fluent_talk_slide.jpg 634w, http://paulcalvano.com/wp-content/uploads/2018/08/fluent_talk_slide-300x165.jpg 300w" sizes="(max-width: 634px) 100vw, 634px" />](https://www.slideshare.net/PaulCalvano/fluent-2018-tracking-performance-of-the-web-with-http-archive-102455732)

I&#8217;m truly grateful that Akamai is both sponsoring the HTTP Archive, as well as allowing me to spend some of my time supporting it. The project provides a significant benefit for the web community and it’s just so much fun to work with. I&#8217;m really looking forward to working with Ilya, Pat and Rick on this &#8211; and can&#8217;t wait to see what comes next!