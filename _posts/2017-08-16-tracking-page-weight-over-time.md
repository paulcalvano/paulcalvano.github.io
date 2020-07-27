---
title: Tracking Page Weight Over Time
date: 2017-08-16T01:35:41+00:00
author: Paul Calvano
layout: post
---
As of July 2017, the “average” page weight is 3MB. [@Tammy](https://twitter.com/tameverts) wrote an [excellent blog post about HTTP Archive page stats and trends](https://speedcurve.com/blog/web-performance-page-bloat/). Last year [@igrigorik](https://twitter.com/igrigorik/) published [an analysis on page weight using CDF plots](https://www.igvita.com/2016/01/12/the-average-page-is-a-myth/). And of course, we can view the trends over time on the [HTTP Archive trends page](http://httparchive.org/trends.php). Since this is all based on HTTP Archive data, I thought I’d start a thread here to continue the discussion on how to gauge the increase in page weight over time.

<!--more-->

To avoid falling into the trap of averages, I decided to run a query that will show us the trend for not just the average, but also the median and various percentiles. I created a Standard SQL query to extract this data.

But first a few notes about this query:

  * I used a wildcard table name. However since BigQuery doesn’t support wildcard of the pattern `httparchive.runs.20*_pages` I decided to use `httparchive.runs.20*` with a `WHERE` clause on `_TABLE_SUFFIX LIKE<br />
'%_pages'` to limit the amount of data processed.
  * I created a user defined JavaScript function that converts the `_TABLE_SUFFIX` variable to a `mm/dd/yyyy` format. This made it easier to chart the results after exporting the data without having to spend time manipulating the `label` field later. 
  * I’m specifically querying for results where the bytesTotal is > 0 bytes. Since March 2017 there have been ~3000 sites per month that are logged with 0 byte responses and I didn’t want that to influence the trend.

Here’s the query :

    CREATE TEMPORARY FUNCTION tableid_to_date(tableid STRING)
    RETURNS STRING
    LANGUAGE js AS """
      try {
        var parts = tableid.split('_');    
        date_string = parts[1] + '/' + parts[2] + '/20' + parts[0];
        return date_string;
      } catch (e) {
        return '';
      }
    """;
    
    SELECT tableid_to_date(_TABLE_SUFFIX) as Date, count(*) as Sites,
           APPROX_QUANTILES(bytesTotal/1024, 100)[SAFE_ORDINAL(25)] AS `Pct25th`,     
           APPROX_QUANTILES(bytesTotal/1024, 100)[SAFE_ORDINAL(50)] AS `Median`,      
           APPROX_QUANTILES(bytesTotal/1024, 100)[SAFE_ORDINAL(75)] AS `Pct75th`,
           APPROX_QUANTILES(bytesTotal/1024, 100)[SAFE_ORDINAL(85)] AS `Pct85th`,
           APPROX_QUANTILES(bytesTotal/1024, 100)[SAFE_ORDINAL(95)] AS `Pct95th`,
           AVG(bytesTotal/1024) as Average
    FROM `httparchive.runs.20*`
    WHERE  _TABLE_SUFFIX LIKE '%_pages' AND bytesTotal > 0
    GROUP BY Date
    

When viewed on a line chart, it’s interesting to note that on May 1, 2017 the average page weight increased considerably, while the median, 75th percentile and even 85th percentile dropped. The 95th percentile continued to rise, indicating that the recent surge over 3MB was largely a result of 5% of the largest pages.

<img src="/assets/wp-content/uploads/2018/03/ha_pageweight.jpg" alt="" width="690" height="274" class="alignnone size-full wp-image-287"  /> 

To explore this deeper I decided to graph the rate of change for each of these metrics from 2016 until now. While change in page weight over time is very gradual, there have been a few noticeable dates where the average page weight increased. For example, in May 2016 there was an interesting jump in page weight for ~ half of the pages. The jump on May 1st, 2017 continues to be of interest because it seems to have been triggered by an increased in the largest pages.

<img src="/assets/wp-content/uploads/2018/03/ha_pageweight_rate_of_change.jpg" alt="" width="690" height="323" class="alignnone size-full wp-image-286" /> 

If we step back and look at this as a yearly trend, it seems that since 2016 the page weight growth across some websites has increased at a much slower rate. In fact 50% of sites may have actually managed to reduce their page weight slightly. This is fantastic for those sites &#8211; and further investigation can be done to see how they are slowing growth (hint: let’s discuss below!). Another ~35% of sites are still showing a slowdown in the rate that their page weights increase. And then there’s the 15%&#8217;ers…

<img src="/assets/wp-content/uploads/2018/03/ha_pageweight_yearlytrend.jpg" alt="" width="690" height="104" class="alignnone size-full wp-image-285" /> 

There’s so much more that can be done to analyze this data. Let’s continue the discussion in the HTTP Archive discussion forum linked below. We can go beyond the average page weight and extract some deeper insight from this data!

_Originally published at <https://discuss.httparchive.org/t/tracking-page-weight-over-time/1049>_