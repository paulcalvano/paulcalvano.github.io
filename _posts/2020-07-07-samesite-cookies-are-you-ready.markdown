---
layout: post
title: "SameSite Cookies - Are You Ready?"
date: 2020-07-07 00:00:00 -0500
tags: 
---
Last year Google[ announced](https://blog.chromium.org/2019/05/improving-privacy-and-security-on-web.html) updates to Chrome that provide a way for developers to control how cross site cookies should work on their sites. This is a good change - as it ultimately improves end user security and privacy by limiting which third parties can read cookies that were set while visiting a different site. It also defeats [cross site request forgery attacks](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)). The implementation is fairly simple, and only requires developers to add the SameSite attribute to their cookies. 

The SameSite attribute is [supported by all modern browsers](https://caniuse.com/#feat=same-site-cookie-attribute), and most have historically defaulted to a permissive use of cookies if the attribute isn’t present. 

![SameSite Cookie Browser Support](/assets/img/blog/samesite-cookies-are-you-ready/gflnfrrax9xc44z8rxgb.png)

Google changed the default behavior of SameSite attribute to secure cookies by default when Chrome 80 was released in February 2020. However it was[ rolled back in April 2020](https://blog.chromium.org/2020/05/resuming-samesite-cookie-changes-in-july.html)<span style="text-decoration:underline;"> </span>to ensure stability during the initial stage of the COVID-19 response. Now they are planning to[ resume SameSite cookie enforcement](https://blog.chromium.org/2020/05/resuming-samesite-cookie-changes-in-july.html) with Chrome 84, which will be released on July 14th. 

Despite almost a year of notice and warnings in the browser console, this seemed to catch many by surprise in February.  How ready are third parties for this now?

 

**What is a Cross Site Third Party Cookie?**

If a request on www.example.com sets the following cookie from its domain, then the browser will store this cookie and send it back on subsequent requests to the same domain. This is an example of a first party cookie. Essentially a cookie whose domain matches the domain that appears in the address bar...

`Set-Cookie: session=abc; path=/; Secure; HttpOnly;`

Now let’s assume that www.example.com includes a third party analytics provider, metrics.analyticsexample.com. When the third party request is made, that third party can also set a cookie in the end users browser. And that third party will be able to read the cookie. This is an example of a third party cookie.

If that same user then navigated to www.example2.com, which uses the same third party analytics provider, then their third party cookies would be readable by them across both sites. The third party is then able to track the user across multiple websites.

**SameSite Cookies**

The SameSite cookie attribute was introduced in a [2016 IETF draft](https://tools.ietf.org/html/draft-west-first-party-cookies-06), but had not been widely adopted initially.  This attribute provided developers with the ability to control when a browser would send a cookie to a third party. Using them is simply a matter of adding the SameSite attribute to a cookie declaration, with one of the three supported values: “None”, “Lax”, and “Strict”.

This provides the following controls:



*   SameSite=None
    *   The browser will send cookies with both cross-site requests and same-site requests.
*   SameSite=Lax
    *   Same-site cookies are withheld on cross-site sub-requests, such as calls to load images or frames, but will be sent when a user navigates to the URL from an external site; for example, by following a link.
*   SameSite=Strict
    *   The browser will only send cookies for same-site requests (requests originating from the site that set the cookie). If the request originated from a different URL than the URL of the current location, none of the cookies tagged with the Strict attribute will be included.

An example of how this is configured is:

`Set-Cookie: key=value; SameSite=Strict`

If the SameSite attribute is not included, then most browsers have historically defaulted to the most permissive behavior: SameSite=None.

**Google Chrome’s Update**

Google has been planning to update the behavior of SameSite within the Chrome browser to default to the more secure SameSite=Lax. Additionally, if a SameSite=None attribute is present, then they would require that the cookie have the “Secure” attribute. There was some concern that this change would cause breakage for some third parties, so a warning message was included in Chrome since version 77 (September 2019).


>A cookie associated with a cross-site resource at <thirdparty domain> was set without the `SameSite` attribute. A future release of Chrome will only deliver cookies with cross-site requests if they are set with `SameSite=None` and `Secure`. You can review cookies in developer tools under Application>Storage>Cookies and see more details at https://www.chromestatus.com/feature/5088147346030592 and[ https://www.chromestatus.com/feature/5633521622188032](https://www.chromestatus.com/feature/5633521622188032).

 

**SameSite Usage Across the Web**

The [HTTP Archive](https://httparchive.org/) stores a tremendous amount of detail for every HTTP request and response for approximately 5.8 million homepages. In the June 2020 data, there were approximately 108 million third party cookies set across 3.79 million homepages. Of these cookies 35,721,768 (32.9%) included the SameSite attribute.  Comparatively in August 2019, 21.4% of cookies had the SameSite attribute.

![SameSite usage for third party cookies](/assets/img/blog/samesite-cookies-are-you-ready/a1lrpyd2wp7irwpfhbjs.png)

_Note: Due to a collection issue described[ here](https://discuss.httparchive.org/t/does-bigquery-contain-har-archive-or-cookies-of-crawled-webpages/1968/8), ~18.6% of third party cookies were unreadable in the June 2020 HTTP Archive data. The remainder of this analysis is on the cookies we could read._

The “Secure” flag of a cookie ensures that the browser only sends the cookie over HTTPS. Chrome made this a requirement to use SameSite=None. Out of the 35 million cookies, nearly 75% of them use the Secure flag. 

![Table Breakdown of Secure vs Non-Secure Cookies and SameSite attribuets](/assets/img/blog/samesite-cookies-are-you-ready/00imlj5llluyriyrzz2g.png)

When we look at this graphically, there are a few interesting observations we can make:

*   SameSite=Lax, which will be the new default, is in use by only 10.82% of secure cookies, but 97% of insecure cookies.
*   SameSite=None is present on 89.10% of Secure cookies.
*   2.65% (238,810) insecure cookies are set with SameSite=None, but not including the Secure flag.  These will default to SameSite=Lax.
*   Only 0.06% (16,000) of secure cookies are using SameSite=Strict!
*   There are less than 1000 SameSite attributes set to an erroneous value (ie, not Lax, Strict or None).

![SameSite Usage for Third Party Cookies, Secure and non-Secure](/assets/img/blog/samesite-cookies-are-you-ready/gvta0gbt9uemri927dff.png)

**Who is using SameSite=None incorrectly?**

There were 238,810 third party cookies set with SameSite=None, but missing the Secure flag. These will default to SameSite=Lax in Chrome 84 unless the Secure flag is added. Overall, there were 1749 third party domains that made this error. The top 5 account for 48% of the erroneous SameSite cookies. This includes Spotxchange, ETargetNet, SmartAdServer, BazaarVoice and EntityTag. 
![Cookies with SameSite=None, but not secure](/assets/img/blog/samesite-cookies-are-you-ready/3ehsetnuxb2l828noj1a.png)

Even more concerning than the number of cookies, is the number of websites that are affected. For example spotxchange.com is setting SameSite=None with insecure cookies on 26,174 websites. EntityTag.co.uk is doing the same for 14,358 websites.

**Who is using SameSite Strict?**

I thought it was odd that SameSite=Strict was used so infrequently. The table below shows some of the third parties that are using it. The top 10 account for 67% of all SameSite=Strict usage.

![Cookies with SameSite=Strict](/assets/img/blog/samesite-cookies-are-you-ready/p96dsxgsqdo99n11ylwq.png)

**What Erroneous SameSite Attribute Values are in Use?**

I was surprised to see that there was such a low percentage of erroneous usage of the SameSite attribute. The top 10 errors listed below account for 80% of the erroneous uses.  Most were SameSite=Secure, SameSite with no value, and SameSite: Lax. 

![Erroneous SameSite Usage](/assets/img/blog/samesite-cookies-are-you-ready/6i8n6hty9254t2hf8lh7.png)

Many of these errors were from a small number of third parties. For example:  

*   SameSite=secure was set by wildbeardbg.com:
    *   dg_perm_sessid=95951591788161; secure; SameSite=Secure
*   SameSite: Lax was set by nytimes.com.
    *   nyt-purr=cfshcfhssc; Expires=Fri, 18 Jun 2021 01:02:51 GMT; Path=/; Domain=.nytimes.com;SameSite: Lax;Secure
*   SameSite=; was set by cloudengage.com
    *   CEUID=E80y%2BgcVMl0o6Skdol5X9FRCrdrbqBNssQeOYrNUnQxC42U4gpGNV6VX; expires=Wed, 01-Jul-2020 16:16:13 GMT; Max-Age=2592000; path=/; samesite=; domain=.cloudengage.com; secure; HttpOnly  

**SameSite Usage Across Popular Third Parties**

So far we’ve looked at how SameSite is being used across third parties. But which third parties are not setting SameSite attributes? By not setting it, they will default to SameSite=Lax. Whether or not this is intentional or will cause breakage will depend on each third party's usage of the cookies. But not setting an explicit SameSite attribute could be indicative of whether a third party has prepared for this.

The graph below shows SameSite usage across some of the most popular third parties. Very few sites third parties are setting SameSite=Lax, which is about to become the new default.  Some of these third parties are used by hundreds of thousands of sites. 

![SameSite Usage Across Popular Third Parties](/assets/img/blog/samesite-cookies-are-you-ready/zofi46l9kj16ojfc8g3n.png)

**Conclusion**

SameSite cookies are a huge win for privacy and security, but there is a risk that Chrome’s new default settings will cause problems. For many cases, this will likely render some cross site tracking techniques ineffective with little change to end user experience. However, with any changes like this there are risks of breakage and serious site issues. With SameSite adoption at less than 33% of third party cookies, this raises the question of how prepared third parties are for this.

It’s important to note that the absence of a SameSite attribute does not necessarily mean that there will be breakage. However depending on how the cookie is used, it has the potential of becoming problematic. It may be worth checking the JS console in Chrome DevTools to see if you see the SameSite warning for any of your third parties. You can also set a flag in Chrome to test how this will affect your site ahead of the Chrome 84 release. The Chrome team has published a [useful guide for debugging SameSite cookies](https://www.chromium.org/updates/same-site/test-debug).

If you are interested in seeing some of the SQL queries and raw data used in this analysis, I’ve created a post with all the details in the [HTTP Archive discussion forums](https://discuss.httparchive.org/t/samesite-cookies-analysis/1988). You can also see all the data used for these graphs in [this Google Sheet](https://docs.google.com/spreadsheets/d/1-zx1AmcvDDSKjOLsM3-AbV9qVXV-Of6dmps8LG-oMSk/edit?usp=sharing). 
