---
title: HTTP Heuristic Caching (Missing Cache-Control and Expires Headers) Explained
date: 2018-03-14T13:00:51+00:00
author: Paul Calvano
layout: post
---
Have you ever wondered why WebPageTest can sometimes show that a repeat view loaded with less bytes downloaded, while also triggering warnings related to browser caching? It can seem like the test is reporting an issue that does not exist, but in fact it&#8217;s often a sign of a more serious issue that should be investigated. _Often the issue is not the lack of caching, but rather lack of control over how your content is cached._

If you have not run into this issue before, then examine the screenshot below to see an example:

<img src="/assets/wp-content/uploads/2018/03/webpagettest_missing_cc_expires.jpg" alt="" width="1753" height="970" class="alignnone size-full wp-image-315"  /> 

<!--more-->

The repeat view shows that we loaded resources from the browser cache, and that seems like a desirable outcome. However the results are flagging a cache issue. And the site received a poor PageSpeed score for &#8220;Caching Static Content&#8221;.

Before we dig into the reasons behind this, let&#8217;s step back and review some fundamentals&#8230;

**HTTP Caching Simplified**

For an HTTP client to cache a resource, it needs to understand 2 pieces of information:

  * &#8220;How long am I allowed to cache this for?&#8221;
  * &#8220;How do I validate that the content is still fresh?&#8221;

RFC 7234 covers this in section [4&#46;2 (Freshness)](https://tools.ietf.org/html/rfc7234#section-4.2) and [4&#46;3 (Validation)](https://tools.ietf.org/html/rfc7234#section-4.3).

The HTTP Response headers that are typically used for **conveying** freshness lifetime are :

  * Cache-Control (max-age provides a cache lifetime duration) 
  * Expires (provides an expiration date, Cache-Control max-age takes priority if both are present)

The HTTP response headers for **validating** the responses stored within the cache, i.e. giving conditional requests something to compare to on the server side, are:

  * Last-Modified (contains the date-time since the object was last modified) 
  * Etag (provides a unique identifier for the content)

The [Cache-Control](https://tools.ietf.org/html/rfc7234#section-5.2) max-age and [Expires](https://tools.ietf.org/html/rfc7234#section-5.3) headers satisfy the job of setting freshness lifetimes (ie, TTLs) as well as provide a time based validation. You can think of them as saying &#8220;this resource is valid for x seconds&#8221; or &#8220;cache this resource until <date/time>&#8221;. When both are present in a response, the browser will [prioritize the Cache-Control over the Expires header.](https://tools.ietf.org/html/rfc7234#section-4.2.1) When the expiration passes, then the browser can send a [conditional request using an If-Modified-Since header](https://tools.ietf.org/html/rfc7234#section-4.3.1) to ask if the object has been modified (since it&#8217;s Last-Modified-Date or from the last resource version the browser received). If neither a Cache-Control or Expires header is present, then the specification allows the browser to [heuristically assign it&#8217;s own freshness time.](https://tools.ietf.org/html/rfc7234#section-4.2.2) It suggests that this heuristic can be based on the Last-Modified-Date.

Etags are useful for providing unique identifiers to a cached item, but they do not specify cache lifetimes. They simply provide a way of [validating the resource via a If-None-Match conditional request.](https://tools.ietf.org/html/rfc7234#section-4.3.1)

**Example**

If we examine a request where this is happening you can see that the site is sending a Last-Modified header, but there are no Cache-Control or Expires headers. The absence of these two headers is why WebPageTest has flagged this as an issue with caching static content.

<img src="/assets/wp-content/uploads/2018/03/webpagettest_missing_cc_expires_example.jpg" alt="" width="955" height="443" class="alignnone size-full wp-image-319"  /> 

This object was last modified on October 27th, 2017 but The browser does not know how long to cache it for, which puts us in the unpredictable realm of &#8220;Heuristic Freshness&#8221;. This can be simplified as _&#8220;So this is cacheable, but you won&#8217;t tell me for how long? I&#8217;ll have to guess a TTL and use that&#8221;_.

This is not to say that there is anything wrong with heuristic freshness. It was originally defined in the HTTP specification so that caches could still add some value in situations where the origin did not set a cache policy, while still being safe. However, it is a loss of control over browser cacheability. The HTTP specification encourages caches to use a heuristic expiration value that is no more than some fraction of the interval since the Last-Modified time. A typical setting of this fraction might be 10%.

**How Long Will Browsers Actually Heuristically Cache Content For?**

So we know that there are problems with this request, and we also know that the cache TTLs may be assigned heuristically based on the Last-Modified date &#8211; but why does Firefox&#8217;s about:cache show that this resource is cacheable for 1 week instead of 10% of 136 days?

<img src="/assets/wp-content/uploads/2018/03/firefox_cache_entry.jpg" alt="" width="597" height="219" class="alignnone size-full wp-image-322" /> 

Chrome&#8217;s cache internals do not show the expiration for a cached resource, so it&#8217;s expiration is not easy to determine. Is it 1 week as well? Or 10% of 136 days? Or something else?

Since Firefox, Chrome and WebKit are open source, we can actually dig into the code and see how the spec is implemented. In the [Chromium source](https://cs.chromium.org/chromium/src/net/http/http_response_headers.cc?sq=package:chromium&dr=C&l=1009) as well as the [Webkit source](https://opensource.apple.com/source/WebCore/WebCore-7604.1.38.1.6/platform/network/CacheValidation.cpp.auto.html), we can see that the lifetimes.freshness variable is set to 10% of the time since the object was last modified (provided that a Last-Modified header was returned). The 10% since last modified heuristic is the same as the suggestion from the RFC. When we look at the [Firefox source code](https://dxr.mozilla.org/mozilla-central/source/netwerk/protocol/http/nsHttpResponseHead.cpp#743), we can see that the same calculation is used, however Firefox uses the `std:min()` C++ function to select the less of the 10% calculation or 1 week. This explains why we only saw the resource cached for 7 days in Firefox. It would likely have been cached for 13 or 14 days in Chrome.

<img src="/assets/wp-content/uploads/2018/03/heuristic_browser_code_chrome_webkit_firefox.jpg" alt="" width="1920" height="440" class="alignnone size-full wp-image-335"  /> 

With Internet Explorer 11, we can look at the Temporary Internet Files to see how the browser handles heuristic freshness. This is similar to the about:cache investigation we did for Firefox, but it&#8217;s the only visibility we have to work with. To do this, clear your browser cache, then open a page where objects are being heuristically cached. Then view the files in `C:\Users\<username>\AppData\Local\Microsoft\Windows\Temporary Internet Files`. The screenshot below shows that there is no expiration for the resources cached, although it&#8217;s not clear whether a heuristic is used.

<img src="/assets/wp-content/uploads/2018/03/heuristic_browser_caching_ie11.jpg" alt="" width="920" height="219" class="alignnone size-full wp-image-327"  /> 

This appears to have been configurable in [earlier versions of IE](https://blogs.msdn.microsoft.com/ie/2010/07/14/caching-improvements-in-internet-explorer-9/). I&#8217;m not sure how heuristic caching is implemented in Edge yet &#8211; but will update this when I find out.

**Summary**

Many content owners incorrectly believe that omitting Cache-Control and Expires headers will prevent downstream caching. In fact, the opposite is true. A worst case scenario of this type of issue would be to combine heuristic caching with infrequent updates to non-versioned CSS/JS objects. Such a scenario would result in page breakage for some clients, while being extremely difficult to reproduce and troubleshoot across different browsers. Simply setting the proper cache headers easily avoids this &#8211; and it&#8217;s simple to fix, either on your web servers or within your CDN configuration.

If you want to ensure that freshness lifetimes are explicitly stated in your HTTP responses, the best way to do this would be to always include a `Cache-Control` header. At a minimum, you should be including `no-store` to prevent caching or `max-age=<seconds>` to indicate how long content should be considered fresh.

If you are an Akamai customer, it&#8217;s easy to create a downstream caching behavior that will automatically update your Cache-Control and Expires headers based on either a static time value or the remaining freshness of the object on the CDN.<img src="/assets/wp-content/uploads/2018/03/downstream_cacheability_akamai.jpg" alt="" width="571" height="234" class="alignnone size-full wp-image-330" /> 

Many thanks to [Yoav Weiss](https://twitter.com/yoavweiss) and [Mark Nottingham](https://twitter.com/mnot) for reviewing this and providing feedback.