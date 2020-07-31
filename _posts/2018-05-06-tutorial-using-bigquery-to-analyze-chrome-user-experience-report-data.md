---
title: 'Tutorial: Using BigQuery to Analyze Chrome User Experience Report Data'
date: 2018-05-06T03:25:24+00:00
author: Paul Calvano
layout: post
related_posts:
  - _posts/2019-01-11-correlating-performance-metrics-to-page-characteristics.md
  - _posts/2018-05-15-analyzing-3rd-party-performance-via-http-archive-crux.md
  - _posts/2018-04-26-using-googles-crux-to-compare-your-sites-rum-data-w-competitors.md

---
Last week I wrote a [blog post](https://paulcalvano.com/index.php/2018/04/26/using-googles-crux-to-compare-your-sites-rum-data-w-competitors/) showing some examples of how you can use the Chrome User Experience report to compare your site&#8217;s RUM data to competitors. In this post I&#8217;d like to share some brief videos to help you quickly get started exploring the data via Google BigQuery.

<!--more-->

**Accessing the Chrome User Experience Report (CrUX) Data**

The [Chrome User Experience Report](https://developers.google.com/web/tools/chrome-user-experience-report/) data is available on Google BigQuery, which is part of the Google Cloud Platform. To get started, [log into Google Cloud](https://console.cloud.google.com), create a project for your CrUX work, and then navigate to the [BigQuery console](https://bigquery.cloud.google.com). Then add the `chrome-ux-report` dataset and explore the way the tables are structured. Here&#8217;s a short video that walks you through this process.

<iframe width="560" height="315" src="https://www.youtube.com/embed/LK0o8gPBfFk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Links:

  * Google Cloud Console: <https://console.cloud.google.com> 

**Brief CrUX Overview**

Now that we&#8217;ve accessed the CrUX data, let&#8217;s explore the table structure and where you can find some resources for additional help:

<iframe width="560" height="315" src="https://www.youtube.com/embed/0MARQKhfniU" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Links:

  * CrUX Overview: <https://developers.google.com/web/tools/chrome-user-experience-report/>
  * Rick Viscomi&#8217;s CrUX Cookbook: <https://github.com/rviscomi/crux-cookbook>

**Comparing Form Factor and Connection Type Distributions For Different Sites**

Next let&#8217;s explore what we can do with the form factor and effective connection type dimensions. In this next video we&#8217;ll explore the % of these dimensions for two sites.

<iframe width="560" height="315" src="https://www.youtube.com/embed/ojFMCD9JY9c" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Links:

  * [CrUX Query Example: Device Histogram](https://bigquery.cloud.google.com/savedquery/513101547739:8691c7f4a6c64867bb4877e7569fddea)

**Competitive Histograms**

In this next video we&#8217;ll take a look at how to analyze the performance of a single metric across multiple sites in BigQuery. We&#8217;ll export the results and graph them &#8211;

<iframe width="560" height="315" src="https://www.youtube.com/embed/F9r8iU1tOY8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Links:

  * [CrUX Query Example: Competitive Histogram](https://bigquery.cloud.google.com/savedquery/513101547739:36c3b67a72554f74838118ccfd1e902f)
  * [HTTP Archive CrUX Histograms](https://httparchive.org/reports/chrome-ux-report) 

**Graphing all the Metrics**

Finally, let&#8217;s UNION together a bunch of queries and examine the performance of a single site by creating histograms for first paint, first contentful paint, DOM Content Loaded and onLoad.

<iframe width="560" height="315" src="https://www.youtube.com/embed/z-N5QE9SUB8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Links:

  * [CrUX Query Example: Histogram With Multiple Metrics](https://bigquery.cloud.google.com/savedquery/513101547739:bd47760e32b742139352a66361d2d91b)

**Conclusion**

I hope this helps you get started digging into the CrUX data. As I mentioned in my earlier blog post on CrUX, Akamai is working on building this functionality into [mPulse](https://www.akamai.com/us/en/products/web-performance/mpulse-real-user-monitoring.jsp) so that it will be easy for our customers to quickly analyze this data in the future. But for now these videos should give you a starting point to explore the data via BigQuery.