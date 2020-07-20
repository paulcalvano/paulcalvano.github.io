---
id: 90
title: Performance Debugging Leads to Discovery of a Bizarre Browser Bug
date: 2017-06-27T05:39:21+00:00
author: Paul Calvano
layout: post
guid: http://54.146.223.226/?p=90
permalink: /index.php/2017/06/27/performance-debugging-leads-to-discovery-of-a-bizarre-browser-bug/
categories:
  - Uncategorized
---
I worked on an interesting performance investigation recently, and I wanted to share some of the techniques I used to dig in and isolate the problem.

It all started when a customer informed me that they rolled back an HTTP/2 configuration push because it caused significant slowdowns on their site. And by significant they meant a greater than one minute delay in render time!

As with many performance troubleshooting efforts, I started with WebPageTest to get an idea of what was causing the delays. I then dug deeper with a variety of additional tools and techniques. This particular issue turned out to be caused by a really interesting bug in the Chrome browser, which was recently patched.  
<!--more-->

  
The bug affected only a handful of sites on Chrome browsers older than v59. When the specific conditions that triggered this behavior occurred, the waterfall graph looked like the below image (we blurred the details to protect the client). Fortunately this only happened to a few sites, but the ones impacted had to disable HTTP/2 until Chrome 59 was released on June 6th, 2017.  
<img src="http://paulcalvano.com/wp-content/uploads/2017/06/waterfall.png" alt="" width="816" height="600" class="alignnone size-full wp-image-185" srcset="http://paulcalvano.com/wp-content/uploads/2017/06/waterfall.png 816w, http://paulcalvano.com/wp-content/uploads/2017/06/waterfall-300x221.png 300w, http://paulcalvano.com/wp-content/uploads/2017/06/waterfall-768x565.png 768w, http://paulcalvano.com/wp-content/uploads/2017/06/waterfall-700x515.png 700w" sizes="(max-width: 816px) 100vw, 816px" /> 

**Collecting Diagnostic Data**  
When we first talked with the customer about this, they explained the issue they saw but none of us at Akamai could reproduce it. I started asking whether there were proxies in their network that could be impacting the traffic. However the issue was occurring both inside and outside their network – and only with browsers that had no existing cache.

We attempted to troubleshoot this in real time and collect diagnostic data. We had scheduled a one hour time where we could enable H2 in production, collect diagnostic data, and quickly roll back the configuration once we had enough information to analyze this further. We collected the following:

  * Akamai logs from the edge servers
  * CatchPoint test results from multiple locations
  * WebPageTest results with packet captures from multiple locations
  * SiteSpeed.io tests run by the customer
  * HAR files exported from browsers that were able to reproduce the issue
  * HTTP/2 Debug Log (from chrome://net-internals/#http2) from browsers that were able to reproduce the issue

**Checking the Logs**  
Despite all the monitoring, we were only able to reproduce the issue on a handful of machines: the customer’s laptop, a few CatchPoint agents, and a few WebPageTest agents. It only happened on Chrome browsers, too.

The first thing I did was attempt to pull logs and see if we could identify the issue there. [Akamai Pragma headers](https://community.akamai.com/community/web-performance/blog/2015/03/31/using-akamai-pragma-headers-to-investigate-or-troubleshoot-akamai-content-delivery) were used to give us the necessary information to track down the logs, and I was able to quickly pull up logs from an impacted client’s requests to Akamai. I saw lots of requests that were responded to in under one second – but nothing that looked anywhere close to 70 seconds. So this told us that either the issue was not being logged, or that it was occurring before an HTTP request was made to Akamai.

**Analyzing the Chrome HTTP/2 Debug Trace**  
Next we examined the HTTP/2 Debug log. These log files can be a bit difficult to follow, so I used [a tool that I wrote for parsing data out of these files](https://github.com/paulcalvano/http2_debug_log_parser) and created a CSV extract. I filtered on the event name HTTP2\_SESSION\_SEND_HEADERS and could see multiple HTTP/2 requests for a coalesced domain.  
<img src="http://paulcalvano.com/wp-content/uploads/2017/06/domain.png" alt="" width="1586" height="473" class="alignnone size-full wp-image-186" srcset="http://paulcalvano.com/wp-content/uploads/2017/06/domain.png 1586w, http://paulcalvano.com/wp-content/uploads/2017/06/domain-300x89.png 300w, http://paulcalvano.com/wp-content/uploads/2017/06/domain-768x229.png 768w, http://paulcalvano.com/wp-content/uploads/2017/06/domain-1024x305.png 1024w, http://paulcalvano.com/wp-content/uploads/2017/06/domain-700x209.png 700w" sizes="(max-width: 1586px) 100vw, 1586px" /> 

However when I looked at the stream_id column, I was able to correlate each request with a “fin” data frame . That confirmed that every HTTP2 request that was made, was responded to. Since we know that the HTTP/2 requests were served quickly at the edge (from the Akamai logs we examined), and there weren’t any requests that weren’t responded to – that means that the browser never actually made the request.  
<img src="http://paulcalvano.com/wp-content/uploads/2017/06/request.png" alt="" width="771" height="493" class="alignnone size-full wp-image-187" srcset="http://paulcalvano.com/wp-content/uploads/2017/06/request.png 771w, http://paulcalvano.com/wp-content/uploads/2017/06/request-300x192.png 300w, http://paulcalvano.com/wp-content/uploads/2017/06/request-768x491.png 768w, http://paulcalvano.com/wp-content/uploads/2017/06/request-700x448.png 700w" sizes="(max-width: 771px) 100vw, 771px" /> 

Looking at the end of the events in the HTTP/2 debug log I can see stream-id 41 complete and then the connection closed 70 seconds later:  
<img src="http://paulcalvano.com/wp-content/uploads/2017/06/streamcomplete.png" alt="" width="827" height="263" class="alignnone size-full wp-image-188" srcset="http://paulcalvano.com/wp-content/uploads/2017/06/streamcomplete.png 827w, http://paulcalvano.com/wp-content/uploads/2017/06/streamcomplete-300x95.png 300w, http://paulcalvano.com/wp-content/uploads/2017/06/streamcomplete-768x244.png 768w, http://paulcalvano.com/wp-content/uploads/2017/06/streamcomplete-700x223.png 700w" sizes="(max-width: 827px) 100vw, 827px" /> 

And the debug log indicated that the error was because the connection was closed:

    t=37328 [st= 382] HTTP2_STREAM_UPDATE_RECV_WINDOW
        --> delta = 914
        --> window_size = 15728640
    t=107321 [st=70375] HTTP2_SESSION_CLOSE
        --> description = "Connection closed"
        --> net_error = -100 (ERR_CONNECTION_CLOSED)
    t=107321 [st=70375] HTTP2_SESSION_POOL_REMOVE_SESSION
        --> source_dependency = 1465 (HTTP2_SESSION)
    t=107321 [st=70375] -HTTP2_SESSION
    

**Analyzing a Packet Capture**  
Next I decided to analyze the TCP packet capture from a test run where the issue occurred. On the WebPageTest instance where I was able to reproduce the issue, I ran a test with the advanced setting: “capture network packet trace”. Once the test was run, I downloaded the tcpdump capture.  
<img src="http://paulcalvano.com/wp-content/uploads/2017/06/packetcapture.png" alt="" width="487" height="227" class="alignnone size-full wp-image-189" srcset="http://paulcalvano.com/wp-content/uploads/2017/06/packetcapture.png 487w, http://paulcalvano.com/wp-content/uploads/2017/06/packetcapture-300x140.png 300w" sizes="(max-width: 487px) 100vw, 487px" /> 

The packet capture clearly shows that the site negotiated a H2 connection via ALPN…  
<img src="http://paulcalvano.com/wp-content/uploads/2017/06/alpn.png" alt="" width="1071" height="526" class="alignnone size-full wp-image-190" srcset="http://paulcalvano.com/wp-content/uploads/2017/06/alpn.png 1071w, http://paulcalvano.com/wp-content/uploads/2017/06/alpn-300x147.png 300w, http://paulcalvano.com/wp-content/uploads/2017/06/alpn-768x377.png 768w, http://paulcalvano.com/wp-content/uploads/2017/06/alpn-1024x503.png 1024w, http://paulcalvano.com/wp-content/uploads/2017/06/alpn-700x344.png 700w" sizes="(max-width: 1071px) 100vw, 1071px" /> 

…and that one of the content domains resolved to the same IP address.

Instead of opening a new TCP connection, this second domain coalesced to the existing H2 connection, which was expected.  
<img src="http://paulcalvano.com/wp-content/uploads/2017/06/dnslookup.png" alt="" width="1065" height="362" class="alignnone size-full wp-image-191" srcset="http://paulcalvano.com/wp-content/uploads/2017/06/dnslookup.png 1065w, http://paulcalvano.com/wp-content/uploads/2017/06/dnslookup-300x102.png 300w, http://paulcalvano.com/wp-content/uploads/2017/06/dnslookup-768x261.png 768w, http://paulcalvano.com/wp-content/uploads/2017/06/dnslookup-1024x348.png 1024w, http://paulcalvano.com/wp-content/uploads/2017/06/dnslookup-700x238.png 700w" sizes="(max-width: 1065px) 100vw, 1065px" /> 

After 1.6 seconds we see a drop in network activity. Then there is no activity for a while, until some TCP Keep-Alive ACKs at ~46s. At around 71s, the edge server sent a FIN packet and closes the connection.

At this time the browser attempted to perform a DNS lookup, and resolved to a different IP address. When it attempted to establish a connection, ALPN indicated that it would only support HTTP/1.1 (H2 was not enabled on the content domain at the time of testing). At that point the client was able to request and receive responses via HTTP/1.1, but after a very severely impacted experience.  
<img src="http://paulcalvano.com/wp-content/uploads/2017/06/70seconds.png" alt="" width="1241" height="595" class="alignnone size-full wp-image-192" srcset="http://paulcalvano.com/wp-content/uploads/2017/06/70seconds.png 1241w, http://paulcalvano.com/wp-content/uploads/2017/06/70seconds-300x144.png 300w, http://paulcalvano.com/wp-content/uploads/2017/06/70seconds-768x368.png 768w, http://paulcalvano.com/wp-content/uploads/2017/06/70seconds-1024x491.png 1024w, http://paulcalvano.com/wp-content/uploads/2017/06/70seconds-700x336.png 700w" sizes="(max-width: 1241px) 100vw, 1241px" /> 

We also saw the same issue occur in CatchPoint, although the test was terminated after 30 seconds. This helped us confirm that there wasn’t an issue with WebPageTest – since it occurred across multiple measurement tools.  
<img src="http://paulcalvano.com/wp-content/uploads/2017/06/measure.png" alt="" width="1851" height="369" class="alignnone size-full wp-image-193" srcset="http://paulcalvano.com/wp-content/uploads/2017/06/measure.png 1851w, http://paulcalvano.com/wp-content/uploads/2017/06/measure-300x60.png 300w, http://paulcalvano.com/wp-content/uploads/2017/06/measure-768x153.png 768w, http://paulcalvano.com/wp-content/uploads/2017/06/measure-1024x204.png 1024w, http://paulcalvano.com/wp-content/uploads/2017/06/measure-700x140.png 700w" sizes="(max-width: 1851px) 100vw, 1851px" />  
At this point, we knew that Akamai wasn’t seeing the requests, the browser wasn’t issuing the requests, and this was occurring across multiple Chrome browsers and measurement tools. It was really looking like a bug in the Chrome browser. However I wasn’t really sure how to prove that.

**Digging Into Chromium**  
The next step in troubleshooting was to work with someone that knew both Akamai’s HTTP/2 implementation as well as Chrome’s HTTP/2 implementation. Fortunately I knew the right person within Akamai to ask: Yoav Weiss, a Principal Architect on Akamai’s Web Experience Engineering team. In addition to helping architect many of the latest and greatest Ion features, he is also a developer on the Google Chromium project and helps contribute important performance features to the browser.

When I asked Yoav about this, he started looking at deep diagnostic data within the Chrome browser and was able to theorize what might be occurring. After some more troubleshooting, he was able modify a development build of Chrome to further analyze and try to isolate where the issue was occurring. He also found another non-production Akamai site that was [affected by the same issue](https://www.webpagetest.org/result/170518_56_91eb51f21d974b4adafc3a395bdd3de6/1/details/#waterfall_view_step1), and was able to analyze that one as well.

After an extremely thorough debugging session, Yoav discovered that under some circumstances, outgoing requests tried to open new sockets before reusing the existing HTTP/2 connection. Once the 6 sockets limit is reached they were added to the “pending requests” container, which wasn’t properly drained and left a few of them in there. In a sense, Chrome’s network stack “forgot” about those requests until the HTTP/2 connection timed out.

**The Chromium Bug Report**  
While we were conducting our analysis another Akamai customer had reached out Google to report the same behavior. And then a WebPageTest user had reported a [similar issue on www.bosslaser.com](https://github.com/WPO-Foundation/webpagetest/issues/877), which is not using a CDN. Google’s Patrick Meenan (the author of WebPageTest as well as a Chrome developer) filed a Chrome bug report as well, and you can see the [bug report and discussion here](https://bugs.chromium.org/p/chromium/issues/detail?id=723748).

This additional information proved both that the issue was more widespread, although still very uncommon. It also confirmed that this was not specifically an Akamai issue since it was occurring on non-Akamaized sites as well.

The bug report and analysis validated much of the analysis we had done up until then, and more importantly one of the Chromium engineers on Google’s Net team, Matt Menke, was able to isolate the cause of the issue and fix it within a day of reporting. His work also confirmed that this is a relatively obscure issue within the socket pool code in Chrome, and that it would have rarely been triggered.

**Conclusion**  
It turned out that the issue we were investigating was not due to a customer’s implementation, but due to a pretty serious browser bug was identified and patched very quickly. Since it was browser bug, we had to wait for a browser release before proceeding with this customer’s HTTP/2 work.

The issue seemed to occur rarely, and we’ve only seen it on a handful of sites. We believe that the issue was also limited to cases where the HTTP/2 connections are coalesced. Chrome v59 was released on June 6th, 2017 and according to <https://bugs.chromium.org/p/chromium/issues/detail?id=723748#c36> the fix was confirmed to be included in Chrome v59.

It’s always fascinating to see how even the most obscure technical issues are discovered and corrected in real time by the open source community, making software better for everyone.

Special thanks to Yoav Weiss (Akamai), Patrick Meenan (Google) and Matt Menke (Google) for their amazing work on digging into this issue and resolving it so quickly.

_Originally published at <https://developer.akamai.com/blog/2017/06/27/performance-debugging-leads-discovery-bizarre-browser-bug/>_