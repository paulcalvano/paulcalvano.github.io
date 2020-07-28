---
title: "Exploring usage of the Console API"
date: 2019-05-22T00:00:00+00:00
author: Paul Calvano
layout: post
---

The JavaScript [Console API](https://developer.mozilla.org/en-US/docs/Web/API/console) is an incredibly useful feature that provides access to the browsers debug console from a web page. It can be accessed from any global object, and it's common to call methods such as `console.log()` to print messages for debugging purposes. For  example:

```
console.log("This is an example!")
```

Logging to the console seems to be the most frequent use of this API, but have you ever wondered how frequently each of the logging methods are used?  And what about other methods provided by the API? I was intrigued about this when seeing @NicJ's plea to developers, asking not to use the `console.clear()` method. The reason for this is pretty straightforward - the console is a shared resource and clearing it can interfere with debugging. 

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Don&#39;t do this <a href="https://t.co/4VCwJKYtDo">pic.twitter.com/4VCwJKYtDo</a></p>&mdash; nicj (@nicj) <a href="https://twitter.com/nicj/status/1129434565905387520?ref_src=twsrc%5Etfw">May 17, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

If you are interested in learning more about the console API, the [MDN Reference](https://developer.mozilla.org/en-US/docs/Web/API/console) and [Google Developer docs](https://developers.google.com/web/tools/chrome-devtools/console/api) are great references.

In the HTTP Archive, we can query the response bodies to answer questions such as:
- Which console object APis are being used the most?
- Are console method calls being triggered more by 1st or 3rd party content?
- How frequently is console.clear() an issue - and who is doing it?

**Identifying Console API  Methods in Response Bodies**
Let's extract some useful data from the response_bodies tables. Considering it's size, the first step in digging in will be to extract as much detail as we may need from the response bodies table.  Saving that to a temporary table allows us to do further analysis with a smaller table. I'm using [REGEXP_EXTRACT_ALL](https://cloud.google.com/bigquery/docs/reference/standard-sql/string_functions#regexp_extract_all) to extract occurrences of these method calls into arrays, and theb [UNNEST](https://cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax#unnest) to output a row for each occurrence in the output.  I'm also grouping by the page, URL and whether the URL is a first or third party (using @igrigorik's classification logic here - https://discuss.httparchive.org/t/what-is-the-distribution-of-1st-party-vs-3rd-party-resources/100/10). 

Note: this query processes 6.2TB of data, well above the free tier limit of 1TB.


```
SELECT page, 
       url, 
       IF (STRPOS(NET.HOST(url),REGEXP_EXTRACT(NET.REG_DOMAIN(page), r'([\w-]+)'))>0, 1, 3) AS party,
       console
FROM (
   SELECT page, url, REGEXP_EXTRACT_ALL(LOWER(body), r'(console.assert|console.clear|console.count|console.countreset|console.debug|console.dir|console.dirxml|console.error|console.exception|console.group|console.groupcollapsed|console.groupend|console.info|console.log|console.profile|console.profileend|console.table|console.time|console.timeEnd|console.timeLog|console.timestamp|console.trace|console.warn)') console
   FROM `httparchive.response_bodies.2019_04_01_desktop` 
),
UNNEST(console) AS console
```

The output from the above query was stored in httparchive.scratchspace.response_bodies_console_api_usage_201904_desktop. This table is 18GB and contains 147 million occurrences of the console API. Now that we have our data - let's dig in!


**Console Method Usage**

The query below summarizes the console API method usage

```
SELECT console, 
       count(*) consoleEvents, 
       count(distinct page) pageCount,
       ROUND(count(distinct page) /  (SELECT count(*) FROM `httparchive.summary_pages.2019_04_01_desktop`),2) percentPages
FROM httparchive.scratchspace.response_bodies_console_api_usage_201904_desktop
GROUP BY console
ORDER BY consoleEvents DESC
```
![444x422](/assets/img/blog/exploring-usage-of-the-console-api/1.jpg) 

Out of the 3,946,357 sites in the April 2019 desktop dataset, 83% of sites included console.log(). Additionally, 68% of sites used console.warn() and 66% of sites used console.error().  The rest of the methods are used less frequently - although even low percentages account for a significant number of web sites. For example, console.clear() was used by 1% of sites - but that is still 27,844 web sites. Graphically, this data looks like:

![690x414](/assets/img/blog/exploring-usage-of-the-console-api/2.jpg) 

Another interesting observation is invalid references to console methods are on up to 2% of sites.  For example, thousands of websites contained the following instead of console.log()
* console_log
* console log
* console-log
* console,log
* console_log


**Usage by First Party vs Third Party**
By adding another dimension to this query, we can also get insight into whether these console API methods are being included by 1st party javascript, 3rd parties or a combination of both. In the query below, I simply added `party` to the query

```
SELECT party,
       console, 
       count(*) consoleEvents, 
       count(distinct page) pageCount,
       ROUND(count(distinct page) /  (SELECT count(*) FROM `httparchive.summary_pages.2019_04_01_desktop`),2) percentPages
FROM httparchive.scratchspace.response_bodies_console_api_usage_201904_desktop
GROUP BY party, console
ORDER BY consoleEvents DESC
```

Graphically, we can see that the popular methods - log, error and warn are nearly split between 1st and 3rd party.  Some other methods that are more popular with third parties, such as console.table(), console.exceptions() and some of the possible typos mentioned above - console,log, console_error, and console-debug.

![690x484](/assets/img/blog/exploring-usage-of-the-console-api/3.jpg) 

**console.clear()**

Now onto the mystery that led me down this path!  In an earlier query we saw that 27,844 web sites included console.clear(). In the above data, we can also see that console.clear() is split between 1st and 3rd party resources. Let's explore this a bit deeper - 

The query below uses a regular expression to extract the filename from a URL.  It's extracting a string after the last `/` character, and before a `#` or `?` character.
```
SELECT party,
       REGEXP_EXTRACT(url, r'(?:.+\/)([^#?]+)') file,
       count(*) consoleEvents, 
       count(distinct url) urlCount
FROM httparchive.scratchspace.response_bodies_console_api_usage_201904_desktop
WHERE console = "console.clear"
GROUP BY party, file
ORDER BY urlCount DESC
```

The results show a few scripts that account for at least 24% of all console.clear() uses on the web.   
![427x500](/assets/img/blog/exploring-usage-of-the-console-api/4.jpg) 

The largest source of console.clear() events comes from the tagdiv wordpress theme, although inspecting the source it seems that these are commented out - 
  ![690x181](/assets/img/blog/exploring-usage-of-the-console-api/5.jpg) 

semantic.js seems to be overriding this function to avoid allowing other scripts to clear the console - 
![690x80](/assets/img/blog/exploring-usage-of-the-console-api/6.png) 

It's usage on invoke.js appears to be a bit more dubious, as they are clearing the console when DevTools is open. There was some discussion about whether DevTools detection should be allowed back in [2017 in a chromium bug report](https://bugs.chromium.org/p/chromium/issues/detail?id=672625). 

![690x204](/assets/img/blog/exploring-usage-of-the-console-api/7.jpg) 

However the vast majority of uses appear to be attempts at debugging that may have been left in place in production, or attempts at clearing out previous entries for readability. For example, the following script cleared the console on an ecommerce site as soon as an item is added to the cart!

![690x115](/assets/img/blog/exploring-usage-of-the-console-api/8.png) 

**Conclusion**

The console API is widely used by both 1st and 3rd party scripts, and 83% of sites are currently using it to facilitate logging informational, warning and error messages to the console. Other features of this API are used sparingly by less then 3% of sites. The output from these messages are valuable for developers both 1st and 3rd party, and thus clearing the console with the console.clear() method should generally be avoided.

_Originally published at <https://discuss.httparchive.org/t/exploring-usage-of-the-console-api/1656>_