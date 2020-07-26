---
title: On Becoming a Contributor to the HTTP Archive
date: 2018-08-27T02:40:16+00:00
author: Paul Calvano
layout: post

---
The [HTTP Archive](https://httparchive.org/) is an open source project that tracks how the web is built. Twice a month it crawls 1.3 million web pages on desktop and emulated mobile devices, and collects technical information about each of the web pages. That information is then aggregated and made available in [curated reports](https://httparchive.org/reports). The raw data is also made [available via Google BigQuery](https://github.com/HTTPArchive/legacy.httparchive.org/blob/master/docs/bigquery-gettingstarted.md), which makes answering interesting questions about the web accessible to anyone with some knowledge of SQL as well as the curiosity to dig in.

[<img src="/assets/wp-content/uploads/2018/08/httparchive.jpg" alt="" width="397" height="215" class="aligncenter size-full wp-image-537" />](https://httparchive.org)

<!--more-->

When [Steve Souders](https://twitter.com/Souders) created the project back in 2010, it included far less pages &#8211; but it was immensely valuable to the community. As [sponsorship](https://httparchive.org/about#sponsors) increased so did the infrastructure and the ability to do more with it. Over time more and more information was added to the archive &#8211; including HAR files, Lighthouse reports and even response bodies.

[<img src="/assets/wp-content/uploads/2018/08/httparchive_sponsors.jpg" alt="" width="1157" height="365" class="aligncenter size-full wp-image-538" />

In 2017 [Ilya Grigorik](https://twitter.com/igrigorik), [Patrick Meenan](https://twitter.com/patmeenan) and [Rick Viscomi](https://twitter.com/rick_viscomi) started maintaining the project. They have done some amazing work overhauling the new website, creating new and useful reports and continuing to push the envelope on what the HTTP Archive is capable of providing to the web community. As of last week I&#8217;ve joined Ilya, Pat and Rick as a co-maintainer of the HTTP Archive, and I couldn&#8217;t be more excited!

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Please help us welcome <a href="https://twitter.com/paulcalvano?ref_src=twsrc%5Etfw">@paulcalvano</a> to the HTTP Archive team! 👏👏👏<br><br>Paul is a long time power user, contributor, and advocate and we&#39;re excited for him to join the admin team! 🎉</p>&mdash; 💾 HTTP Archive (@HTTPArchive) <a href="https://twitter.com/HTTPArchive/status/1032407393320218624?ref_src=twsrc%5Etfw">August 22, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


**So how have I been using the HTTP Archive?**

Rarely does a week go by where someone doesn&#8217;t ask a question or share a news article that doesn&#8217;t provoke a question that can be answered with the archive. I love diving deep into questions about the web, and many of my colleagues joke about &#8220;nerd sniping Paul&#8221;. Fortunately no Paul&#8217;s have been injured using BigQuery :).

[<img src="/assets/wp-content/uploads/2018/08/nerdsniping.jpg" alt="" width="753" height="456" class="aligncenter size-full wp-image-539" />](https://xkcd.com/356/) <small><em>Source: <a href="https://xkcd.com/356/">https://xkcd.com/356/</a></em></small>

Over the past few months, I&#8217;ve been sharing some of my research on the [HTTP Archive Discussion forums](https://discuss.httparchive.org/). An example of a recent post was just a few days ago when the [Blink-Dev team announced that the Application Cache was being deprecated in Chrome](https://groups.google.com/a/chromium.org/forum/#!msg/blink-dev/FvM-qo7BfkI/0daqyD8kCQAJ). It only took a few minutes to write a SQL query to identify sites that are still using this feature. After doing some analysis on the data, I wound up sharing my research with the blink team so that they could track this. Since I work at Akamai, I&#8217;m also planning to give a proactive heads up to the customers whose sites will be affected. Being able to quickly notify numerous websites of an important change that might impact their business is truly a priceless use case.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Blink is deprecating AppCache. I decided to take a look at how much AppCache is used in the <a href="https://twitter.com/HTTPArchive?ref_src=twsrc%5Etfw">@HTTPArchive</a> - <a href="https://t.co/8g654KcDz6">https://t.co/8g654KcDz6</a> <a href="https://t.co/EvOPkYPnCT">https://t.co/EvOPkYPnCT</a></p>&mdash; Paul Calvano (@paulcalvano) <a href="https://twitter.com/paulcalvano/status/1032862176762048512?ref_src=twsrc%5Etfw">August 24, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

At the [2018 Fluent Conference in San Jose, CA](https://conferences.oreilly.com/fluent/fl-ca) this past June, I&#8217;ve shared a few additional examples of how I&#8217;ve used the HTTP Archive at Akamai. You can see the slides [here](https://www.slideshare.net/PaulCalvano/fluent-2018-tracking-performance-of-the-web-with-http-archive-102455732), where I talk about how I used the archive to help improve configuration defaults, assist in product research and even security notifications.

[<img src="/assets/wp-content/uploads/2018/08/fluent_talk_slide.jpg" alt="" width="634" height="348" class="aligncenter size-full wp-image-536"  />](https://www.slideshare.net/PaulCalvano/fluent-2018-tracking-performance-of-the-web-with-http-archive-102455732)

I&#8217;m truly grateful that Akamai is both sponsoring the HTTP Archive, as well as allowing me to spend some of my time supporting it. The project provides a significant benefit for the web community and it’s just so much fun to work with. I&#8217;m really looking forward to working with Ilya, Pat and Rick on this &#8211; and can&#8217;t wait to see what comes next!