---
title: Which 3rd Party Content Loads Before Render Start?
date: 2017-09-17T14:46:52+00:00
author: Paul Calvano
layout: post
---
Since the HTTP Archive is capturing the timing information on each request, I thought it would be interesting to correlate request timings (ie, when an object was loaded) with page timings. The idea is that we can categorize resources that were loaded before or after and event.

**Content Type Loaded Before/After Render Start** It’s generally well known that third party content impacts performance. We see this with both resource loading, and JavaScript execution blocking the browser from loading other content. While we don’t have the data to evaluate script execution timings per resource captured here, we can definitely look at when resources were loaded with respect to certain timings and get an idea of what is being loaded before a page starts rendering. <!--more-->

For example, here’s a query to categorize objects that are loaded before/after RenderStart by Content-Type

    SELECT IF(STRPOS(resp_content_type, ";")>0,SUBSTR(resp_content_type, 0, STRPOS(resp_content_type, ";")-1), resp_content_type) as ContentType,
           COUNT(*) num_requests,
           SUM(IF(req.startedDateTime < (pages.startedDateTime + (renderStart/1000) ),1,0)) BeforeRenderStart,
           SUM(IF(req.startedDateTime < (pages.startedDateTime + (renderStart/1000) ),0,1)) AfterRenderStart 
    FROM httparchive.runs.2017_09_01_requests req 
    JOIN ( 
           SELECT rank, NET.HOST(url) hostname, url, pageid, startedDateTime, renderStart 
           FROM httparchive.runs.2017_09_01_pages 
         ) pages ON pages.pageid = req.pageid 
    WHERE NET.HOST(req.url) != pages.hostname 
         and rank > 0 and rank < 100000 
    GROUP BY ContentType 
    HAVING num_requests > 1000
    ORDER BY BeforeRenderStart desc
    

The query may look complicated, but I’ll break it down before we continue:

  * The calculation for the ContentType column is truncating everything to the right of a semicolon in the Content-Type header. For example, text/javascript; charset=UTF-8 becomes text/javascript. `IF(STRPOS(resp_content_type, ";")>0,SUBSTR(resp_content_type, 0, STRPOS(resp_content_type, ";")-1), resp_content_type) as ContentType`

  * The BeforeRenderStart and AfterRenderStart columns are calculated by looking at the difference of when the request for the 3rd party content was made, and the the time when the page started to render. `SUM(IF(req.startedDateTime < (pages.startedDateTime + (renderStart/1000) ),1,0)) BeforeRenderStart,<br />
SUM(IF(req.startedDateTime < (pages.startedDateTime + (renderStart/1000) ),0,1)) AfterRenderStart`

  * The WHERE clause is comparing the hostname of the request to the  
    hostname of the page. I’m considering a hostname to be a third party if it doesn’t match the hostname of the page AND if there are at  
    least 1000 requests for the host in the HTTP Archive data. This logic should avoid content domains for individual sites from being counted as thirdparties.
    
    `WHERE NET.HOST(req.url) != pages.hostname HAVING  num_requests > 1000`

  * I’m looking at rank > 0 AND rank < 100000 to start with, but will use this to look at other Alexa rankings later on…

**So, What Content Types Load Before/After Render Start?** I was a bit surprised to see that the most frequent 3rd party content type was images and not JavaScript.<img src="/assets/wp-content/uploads/2017/09/fdce5c823be98d9f15b9aee6bb54bed990491b55.png" alt="" width="543" height="293" class="alignnone size-full wp-image-199" srcset="http://paulcalvano.com/wp-content/uploads/2017/09/fdce5c823be98d9f15b9aee6bb54bed990491b55.png 543w, http://paulcalvano.com/wp-content/uploads/2017/09/fdce5c823be98d9f15b9aee6bb54bed990491b55-300x162.png 300w" sizes="(max-width: 543px) 100vw, 543px" /> 

As with all things HTTP Archive, when you find something unexpected &#8211; you keep digging deeper! I wound up tweaking the WHERE clause to look at different ranges of Alexa rankings, and found that in the top 100 sites, most 3rd party content was images. In the top 1000 sites,it was images, javascript and css. And then the top 10000+ sites fonts get added to that list. In the table below, the % columns indicate the percentage of requests for a specific content type that was loaded before RenderStart.<img src="/assets/wp-content/uploads/2017/09/6a28a7c092bc8adfbf9d73f55edf0640ea382635.png" alt="" width="642" height="421" class="alignnone size-full wp-image-195" srcset="http://paulcalvano.com/wp-content/uploads/2017/09/6a28a7c092bc8adfbf9d73f55edf0640ea382635.png 642w, http://paulcalvano.com/wp-content/uploads/2017/09/6a28a7c092bc8adfbf9d73f55edf0640ea382635-300x197.png 300w" sizes="(max-width: 642px) 100vw, 642px" /> 

It’s interesting to note that many of the top ranked sites appear to be self hosting their own fonts, while the less popular sites are using fonts hosted by a 3rd party. Note &#8211; I wrote a blog post last month about custom web font performance implications here2. Bram Stein recently published an excellent book on this topic as well, which you can find here.

**Examining The Results Per Third Party Domain and Content Type** To dig a bit deeper, I modified the above query to include the third party domain in the SELECT clause and reduce the limit in the HAVING clause. This allowed me to look at each content type and domain. For example, I ran the following to analyze 3rd party domains for the top 100K sites:

    SELECT NET.HOST(req.url) thirdparty, 
           IF(STRPOS(resp_content_type, ";")>0,SUBSTR(resp_content_type, 0, STRPOS(resp_content_type, ";")-1), resp_content_type) as ContentType, 
           COUNT(*) num_requests, 
           SUM(IF(req.startedDateTime < (pages.startedDateTime + (renderStart/1000) ),1,0)) BeforeRenderStart, 
           SUM(IF(req.startedDateTime < (pages.startedDateTime + (renderStart/1000) ),0,1)) AfterRenderStart 
    FROM httparchive.runs.2017_09_01_requests req 
    JOIN ( 
           SELECT rank, NET.HOST(url) hostname, url, pageid, startedDateTime, renderStart 
           FROM httparchive.runs.2017_09_01_pages 
         ) pages ON pages.pageid = req.pageid 
    WHERE NET.HOST(req.url) != pages.hostname 
          and rank > 0 and rank < 100000 
    GROUP BY thirdparty, ContentType 
    HAVING num_requests > 100 
    ORDER BY BeforeRenderStart desc 
    

<img src="/assets/wp-content/uploads/2017/09/e45f2891d78c89c28e9096ecccf32992ab42a80c_1_690x274.png" alt="" width="690" height="274" class="alignnone size-full wp-image-196" srcset="http://paulcalvano.com/wp-content/uploads/2017/09/e45f2891d78c89c28e9096ecccf32992ab42a80c_1_690x274.png 690w, http://paulcalvano.com/wp-content/uploads/2017/09/e45f2891d78c89c28e9096ecccf32992ab42a80c_1_690x274-300x119.png 300w" sizes="(max-width: 690px) 100vw, 690px" /> 

Google and Facebook content shows up prominently at the top of these results due to their frequent usage. However don’t be misled by assuming that this is anything other than the tip of the iceberg. Scrolling through the data there are many other 3rd parties represented here. Here’s a zoomed out view. The darker the cell, the higher the % of resources loading before RenderStart. And the list goes on much longer than this too…<img src="/assets/wp-content/uploads/2017/09/0965faacea5cd2eb3843f5e16a909f3a7f7a3c96_1_556x500.png" alt="" width="556" height="500" class="alignnone size-full wp-image-197" srcset="http://paulcalvano.com/wp-content/uploads/2017/09/0965faacea5cd2eb3843f5e16a909f3a7f7a3c96_1_556x500.png 556w, http://paulcalvano.com/wp-content/uploads/2017/09/0965faacea5cd2eb3843f5e16a909f3a7f7a3c96_1_556x500-300x270.png 300w" sizes="(max-width: 556px) 100vw, 556px" /> 

Focusing on some of the third parties with a higher % of requests loaded prior to RenderStart, we can see some familiar names, and the content types being served from them &#8211;<img src="/assets/wp-content/uploads/2017/09/1d3af0a6d4ea02f906c472485de5d3112ac5d465_1_682x500.png" alt="" width="682" height="500" class="alignnone size-full wp-image-198" srcset="http://paulcalvano.com/wp-content/uploads/2017/09/1d3af0a6d4ea02f906c472485de5d3112ac5d465_1_682x500.png 682w, http://paulcalvano.com/wp-content/uploads/2017/09/1d3af0a6d4ea02f906c472485de5d3112ac5d465_1_682x500-300x220.png 300w" sizes="(max-width: 682px) 100vw, 682px" /> 

While it’s interesting to explore the data like this, another practical use case for this type of analysis would be to research how to optimize for a specific 3rd party.For example, you can modify some of the queries above to output a list of websites that utilize a specific 3rd party, and whether it is loading before/after RenderStart. If you see something interests you, then dig in deeper and possibly learn a new optimization trick! The thing I like about this approach is it crosses industries &#8211; so an ecommerce site might be able to learn something from analyzing an airline site, a media site, a news site, etc!

_Originally published at <https://discuss.httparchive.org/t/which-3rd-party-content-loads-before-render-start/1084>_