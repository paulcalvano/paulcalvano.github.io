---
layout: post
title: "Identifying Font Subsetting Opportunities with Web Font Analyzer"
date: 2024-02-16 10:00:00 -0400
related_posts:
 
  - _posts/2017-07-25-performance-and-usage-implications-of-custom-fonts.md
---

10 years ago, custom web fonts were a niche feature [used by ~10% of websites](https://almanac.httparchive.org/en/2022/fonts#fig-1). Today they are used by over 83% of websites! Fonts are generally loaded as a [high priority resource](https://docs.google.com/document/d/1bCDuq9H1ih9iNjgzyAL0gpwNFiEP4TZS-YLRp_RuMlc/edit#), and some sites use techniques such as [preload](https://web.dev/articles/codelab-preload-web-fonts) and [early hints](https://developer.chrome.com/docs/web-platform/early-hints) to get them to load as quickly as possible. Custom web fonts are important to many sites, since rendering with a specific typography is often preferred from a design perspective. However, this can easily become a performance issue when a large amount of fonts are loaded. 

![Image of a WebPageTest waterfall where a site is loading a large amount of fonts](/assets/img/blog/identifying-font-subsetting-opportunities/font-loading-example.jpg)

In this article, we’ll explore some potential issues around font loading and the performance benefits of a lesser used feature - font subsetting. We’ll look at [HTTP Archive](https://httparchive.org/) data to understand how prevalent the issue is, and then examine a few case studies. And finally, I created a new tool - [Web Font Analyzer](https://tools.paulcalvano.com/wpt-font-analysis/) - that may help you explore whether font subsetting is something to consider optimizing on your site.

**Background**

There’s been a lot written about web fonts over the years. In fact the [HTTP Archive’s Web Almanac](https://almanac.httparchive.org/) has an [entire chapter](https://almanac.httparchive.org/en/2022/fonts) dedicated to this topic. A few years ago Zach Leatherman wrote a fantastic [checklist](https://www.zachleat.com/web/font-checklist/) on font loading strategies, which included using preload to load fonts earlier. Way back in 2016 the CSS [font-display](https://developer.chrome.com/blog/font-display/) attribute was introduced, and today it is [supported](https://caniuse.com/?search=font-display) on all modern browsers and used by almost a third of websites! [Variable fonts](https://almanac.httparchive.org/en/2022/fonts#variable-fonts) are heavily used by Google Fonts which are widely used across the web. Barry Pollard wrote a great article on [self hosting Google Fonts](https://www.tunetheweb.com/blog/should-you-self-host-google-fonts/). And just last month Stoyan Stefanov wrote an article about [subsetting fonts](https://www.phpied.com/bytes-normal-web-font-study-google-fonts/) as a way to further reduce their size.

The font-display feature was a major step forward in font loading performance, as it gave developers control over whether to prioritize rendering or typography. Using font-display:swap would avoid a rendering delay by painting text using a system font and then swapping the actual font after it’s loaded. 

Font optimization strategies are great, but when combined the results can be confusing. For example, preloading fonts is a great way to get them to load earlier, but using font-display:swap at the same time may result in a less effective use of bandwidth early in the page load. It’s a good idea to understand exactly how your fonts are loading, how they are being used and what they contain.

**Font sizes across popular sites**

Using the HTTP Archive we can explore font usage across millions of websites. For the purpose of this analysis, we’ll focus on the top million sites. As of January 2024, 81.5% of the top million sites are utilizing at least one custom web font. Usage varies widely, with the average site loading 238 KB of fonts.

![Font usage by site popularity](/assets/img/blog/identifying-font-subsetting-opportunities/font-usage-by-site-popularity.jpg){:loading="lazy"}

Depending on the font loading strategy used, fonts may be delivered at different parts of the page loading lifecycle. For the purpose of this analysis, I’m going to break this up into 4 parts:
* Before FCP - Means that the fonts finished downloading before the First Contentful Paint was measured. This could indicate that a font was render blocking, fetched with a high priority or preloaded.
* FCP to LCP - Means that the font was loaded in between the First and Largest Contentful Paint. These fonts were loaded while other resources critical to the user experience were loading
* LCP to onLoad - The fonts were loaded after the Largest Contentful Paint but before the onLoad event.
* After onLoad - This could indicate that the font was either delayed or not used by the DOM until much later on.

Out of the top million sites that are loading fonts, 63.6% are loading at least 1 custom web font prior to FCP. I would expect this to be high, especially considering how often preloading fonts is recommended. 

![Font downloads relative to performance metrics](/assets/img/blog/identifying-font-subsetting-opportunities/font-downloads-relative-to-perf-metrics.jpg){:loading="lazy"}

However, 25% of these sites are loading more than 75 KB of fonts before the FCP, and over 4500 sites are loading more than 500 KB of fonts during this time! Regardless of the benefits of the font loading strategies applied - I’d say that there is likely some waste occurring.

**Glyphs vs Characters on Pages**

A font is essentially a typographical representation of a character. One can display the same text with different fonts and they will appear differently on a web page - however the unicode value for the character will always be the same. For example, the space character is 0032, the values ABC are 0065, 0066 and 0067 respectively across numerous fonts. 

Some font packages are designed to display icons, and others are designed for text. However the one thing that is not always apparent to developers is how many glyphs are included in each font package. For example, a popular Google font called [“Material Icons”](https://fonts.gstatic.com/s/materialicons/v140/flUhRq6tzZclQEJ-Vdg-IuiaDsNcIhQ8tQ.woff2) is used on 114K websites. It contains 2229 glyphs and adds 128 KB to websites using it. It’s very unlikely that sites are making use of all these glyphs.

The most popular Google font is [Open Sans](https://fonts.gstatic.com/s/opensans/v36/memvYaGs126MiZpBA-UvWbX2vVnXBbObj2OVTS-mu0SC55I.woff2), and it’s used by over 1.5 million websites. You can use a tool like [FontDrop](https://fontdrop.info/) to explore the contents of your fonts, and you might be surprised! This font contains 280 glyphs and adds 43 KB to pages using it. Loading a few fonts like this can really add up.

![Open Sans Font Glyphs](/assets/img/blog/identifying-font-subsetting-opportunities/open-sans.jpg){:loading="lazy"}

So how does that compare with the rendered HTML of a page? Using the HTTP Archive, I was able to write a query that extracts and summarizes the visible glyphs in rendered HTML pages for the top 10K sites. We can then compare this to the minimum and maximum number of glyphs in a page’s custom web fonts. The results might surprise you!.

* The median popular site contains 3 fonts, totalling 95 KB. The rendered HTML has 101 glyphs, while the smallest font has 251 glyphs.
* At every percentile, the number of glyphs in the smallest font has 2-3x the number of glyphs compared to the rendered HTML. 

<table>
  <tr style="font-weight:bold">
   <td></td>
   <td colspan="5"><p style="text-align: center">Font Glyphs vs Rendered HTML</p></td>
  </tr>
  <tr style="font-weight:bold">
   <td></td>
   <td>Font Count</td>
   <td>Font Weight</td>
   <td>Visibly Glyphs</td>
   <td>Min Font Glyphs</td>
   <td>Max Font Glyphs</td>
  </tr>
  <tr>
   <td>p50</td>
   <td><p style="text-align: right">3</p></td>
   <td><p style="text-align: right">95 KB</p></td>
   <td><p style="text-align: right">101</p></td>
   <td><p style="text-align: right">248</p></td>
   <td><p style="text-align: right">524</p></td>
  </tr>

  <tr>
   <td>p75</td>
   <td><p style="text-align: right">5</p></td>
   <td><p style="text-align: right">193 KB</p></td>
   <td><p style="text-align: right">124</p></td>
   <td><p style="text-align: right">444</p></td>
   <td><p style="text-align: right">861</p></td>
  </tr>

  <tr>
   <td>p95</td>
   <td><p style="text-align: right">9</p></td>
   <td><p style="text-align: right">542 KB</p></td>
   <td><p style="text-align: right">221</p></td>
   <td><p style="text-align: right">901</p></td>
   <td><p style="text-align: right">2229</p></td>
  </tr>

  <tr>
   <td>p99</td>
   <td><p style="text-align: right">15</p></td>
   <td><p style="text-align: right">1001 KB</p></td>
   <td><p style="text-align: right">631</p></td>
   <td><p style="text-align: right">1530</p></td>
   <td><p style="text-align: right">3248</p></td>
  </tr>      
</table>


**How to Test Your Page**

There’s a few tools in this area that provide some insight into font usage:

* [FontDrop](https://fontdrop.info/) provides a useful way to explore the contents of your fonts. Simply drop the font file onto their UI and it will show you the metadata and all the glyphs contained in it.
* [YellowLabs](https://yellowlab.tools/) has a font audit that will tell you if you have unused glyphs and summarize it by character sets. This can also be useful when deciding to subset fonts.

As part of this research, I created a tool named “Web Font Analyzer” that will leverage results from a WebPageTest measurement to show you a summary of the glyphs used on a page, when they are loaded relative to some performance metrics and how many glyphs are supported by each font. I’ve also attempted to provide some guidance on how you can use this information in the tool. 

You can find the tool at [https://tools.paulcalvano.com/wpt-font-analysis/](https://tools.paulcalvano.com/wpt-font-analysis/). 

Using the HTTP Archive we can identify sites that exhibit some of these issues, and there are quite a few. Let’s look at a few examples using this tool alongside WebPageTest.  

**Example - Mayoclinic**

After running a [WebPageTest](https://www.webpagetest.org/result/240113_BiDcGH_4W6/1/details/) measurement for Mayoclinic’s homepage, I copy and pasted the URL of the test results into the font analyzer tool. The tool will fetch the measurement data and create a summary of the fonts that were loaded.

![Web Font Analyzer](/assets/img/blog/identifying-font-subsetting-opportunities/web-font-analyzer.jpg){:loading="lazy"}

The page summary section tells you how many fonts you are loading, and how large they are. It also provides a summary of the number of visible characters in the rendered HTML. In this example, there were 366 KB of fonts loaded on a page that contained only 85 visible glyphs. You can also click on “show glyphs” to see a summary of the actual glyphs in the rendered HTML. 

![Character Summary - Mayoclinic](/assets/img/blog/identifying-font-subsetting-opportunities/char-summary-mayoclinic-example.jpg){:loading="lazy"}

Next we can see where these fonts are loading. The majority of these fonts are loading prior to the First Contentful Paint.And all of them are loading prior to the Largest Contentful Paint. It’s very likely that these font assets are competing with other resources for bandwidth.

![When are Fonts Loading - Mayoclinic](/assets/img/blog/identifying-font-subsetting-opportunities/when-are-fonts-loading-mayoclinic-example.jpg){:loading="lazy"}

If we examine the summary of fonts used on the page, we can see that many of them contain more than 500 glyphs!

![Font Usage Summary - Mayoclinic](/assets/img/blog/identifying-font-subsetting-opportunities/font-usage-summary-mayoclinic-example.jpg){:loading="lazy"}

In the WebPageTest measurement we can see 8 font files loading immediately after the HTML. 

![Mayoclinic WebPageTest Measurement](/assets/img/blog/identifying-font-subsetting-opportunities/wpt-mayoclinic-example-1.jpg){:loading="lazy"}

In the HTTP response header for the base page, there is a Link header with preloads for 5 font files. Of these preloads, 4 are for fonts hosted on www.mayoclinic.com, and the other is design.mayloclinic.com. While the preloaded fonts are downloading, the HTML initiates 4 preloads for the font files on www.mayoclinic.com (however those are already in flight). Then the fonts.css file loads, which references fonts from design.mayoclinic.com. During this page load, approximately 700ms was spent loading fonts, half of which were unused. Each of these font files were ~42 KB and contained > 500 glyphs. Meanwhile the page contained less than 100 glyphs in the rendered HTML.

Recommendations:
* Update CSS to use the fonts from www.mayoclinc.com.
* Subset the font files to a latin character set to reduce their weight by up to 90%.
* Continue preloading the (much smaller) font files. 

**Example - Kia**

You can see another example on Kia.com US homepage. The page weight is 21.5 MB, and 3 MB of that are font files. There are 85 visible glyphs on the page, and expanding the glyphs we can see that there are two korean glyphs (codepoints 54620 and 44544) that are used to render the “Korean” language switch link in the footer.

![Character Summary - Kia](/assets/img/blog/identifying-font-subsetting-opportunities/char-summary-kia-example.jpg){:loading="lazy"}

Prior to LCP, there was almost 7 MB of content loaded, including all of the fonts. Font weight accounts for 14% of overall page weight, but a whopping 42% of bytes loaded prior to LCP! The fonts are almost certainly competing for bandwidth.

![When are Fonts Loading - Kia](/assets/img/blog/identifying-font-subsetting-opportunities/when-are-fonts-loading-kia-example.jpg){:loading="lazy"}

When looking at the individual font assets loaded, we can see that these are not using font-display:swap, and that 3 of them contain 15,190 glyphs! It also appears that all of the KiaSignature fonts are delivered as WOFF files (converting them to WOFF2 would reduce some of the bytes). Their icons font is also delivered as a TTF file, and not gzip compressed.    

![Font Usage Summary - Kia](/assets/img/blog/identifying-font-subsetting-opportunities/font-usage-summary-kia-example.jpg){:loading="lazy"}

The fonts are referenced in clientlib-base.css and they are not being preloaded. At the start of the waterfall we can see the first-party CSS and JS loading, but then clientlib-base.js gets interrupted by the higher priority KiaSignatureRegular.woff font file. This font is 965 KB, which delays the JavaScript and ultimately the first contentful paint. Additionally, the font is only cached for 1 day - so repeat visitors will need to download the fonts again.

![Kia WebPageTest Example - Part 1](/assets/img/blog/identifying-font-subsetting-opportunities/wpt-kia-example-1.jpg){:loading="lazy"}

Further down the waterfall we can see numerous PNG images. However they are being delayed by 2.1 MB of fonts. At the same time, a 2 MB hero image loaded via Adobe Experience Cloud (Scene7) is fetched and used as the poster image for a hero video.

![Kia WebPageTest Example - Part 2](/assets/img/blog/identifying-font-subsetting-opportunities/wpt-kia-example-2.png){:loading="lazy"}

While fonts are not the only factor affecting the performance of this page, they are clearly holding back the FCP and LCP by competing with other resources for bandwidth. Applying some easy to implement performance optimizations on the images such as lazy loading, using optimal image formats, and better cache directives will help - but ultimately the font loading will still cause delays - so subsetting them would be ideal.

Recommendations:
* Subset the KiaSignature fonts for regional websites so that they are using only the necessary glyphs. For example on the US site, using latin + extended latin + the specific KR characters needed. 
* Convert WOFF files to WOFF2
* Enable gzip compression for kia-icons.ttf. This would reduce the file to 104 KB (45% smaller). Subset the font to reduce the size even further.
* Fonts are only cached on the browser for 1 day. TTL should be increased. 
* Not font related, but consider image and video compression, alternate image formats,and lazy loading images. 

**How to Subset Fonts**

In some of the examples above, font subsetting would be an excellent optimization. However based on the data from the HTTP Archive, it doesn’t seem that this is being used very often. Zach Leatherman wrote a great tool for subsetting fonts, called [Glyphhanger](https://github.com/zachleat/glyphhanger/).  You can also use the [fonttools](https://github.com/fonttools/fonttools) command line library to subset your fonts. 

To use fonttools, you need to install the library. 

```
sudo apt-get install fonttools 
```

Once installed, you’ll have the pyfmtfont application which can be used to subset fonts. You can see some examples of its usage in [this blog post](https://markoskon.com/creating-font-subsets/).

When I was creating the WPT Font Analyzer tool, I started using a [fontawesome font](https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/webfonts/fa-solid-900.woff2) to render a checkbox, an exclamation point and an information circle icon. This resulted in a 124 KB font, which was many times the size of the rest of the tool! I was able to reduce this to a small 1 KB font by running the following command to create a subsetted font with the glyphs I needed. 


```
pyftsubset fa-solid-900.woff2 \
        --unicodes=U+f05a+f058+f071 \
        --flavor=woff2 \
        --output-file=fa-solid-900-subset.woff2
```

Let’s try this against the 3 Kia fonts we saw from the previous example. In this example, I’m subsetting 

* Basic latin characters (02-7E)
* Copyright symbol (A9)
* Registered Trademark symbol (AE)
* Trademark Symbol (2122)
* Double Quotes (201C-201D)
* Korean Hangul syllables (D55D, ADC0)

```
pyftsubset KiaSignatureRegular.woff \
          --flavor=woff2 \
          --unicodes="U+0020-007E,U+00A9,U+00AE,U+2122,U+201C-021D,U+D55D,U+ADC0" \
          --output-file=KiaSignatureRegular_subset.woff2
```

The resulting file is 10 KB for each of the KiaSignature fonts. So in this example 3 MB of font weight could be reduced to 30 KB! You can download this subsetted font [here](/assets/img/blog/identifying-font-subsetting-opportunities/KiaSignatureRegular_subset.woff2) and examine it in [FontDrop](https://fontdrop.info/).

**Conclusion**

Fonts can be challenging to support from a web performance perspective, especially as their placement on modern web pages occurs at the intersection of design and web operations. Over the years there have been some innovative approaches to font loading, designed to limit the performance overhead of them. There’s also been a lot of great research in the web performance industry on this topic and best practices published. It’s always worth evaluating the end user experience to ensure that the tools and optimizations put in place are having the desired impact on user experience. I’m hopeful that the [Web Font Analyzer](https://tools.paulcalvano.com/wpt-font-analysis/) tool I created adds another lens for you to evaluate font loading through.

Many thanks to [Barry Pollard](https://twitter.com/tunetheweb) and [Tim Vereecke](https://twitter.com/TimVereecke) for reviewing this.



**Queries**

Interested in seeing some of the HTTP Archive Queries behind this analysis? Here's a few of the queries I used.  Please be aware that some of these queries  will exceed the free tier quota - so be careful when running them!  (You can read more on how to minimize query costs at [har.fyi](https://har.fyi/guides/minimizing-costs/).)

<details>
  <summary><b>Percent of sites using fonts by rank</b></summary>
  This query will calculate the percentage of sites that are using at least 1 custom web font, and group the results by rank.
  <pre><code>
SELECT 
  rank, 
  IF(SAFE_CAST(JSON_EXTRACT(summary, "$.reqFont") AS INT64) >0,"Fonts", "No Fonts") as f,
  COUNT(*),
  COUNT(0) / SUM(COUNT(0)) OVER (PARTITION BY rank) AS pct

FROM `httparchive.all.pages`
WHERE 
  date = "2024-01-01"
  AND is_root_page = true
  AND client = "mobile"
GROUP BY rank,f
ORDER BY rank ASC
  </code></pre>
</details>

<details>
  <summary><b>Average font weight across top million sites</b></summary>
  This query will calculate the average, median and p75 font weight of sites with a rank <= 1 million, which are using at least 1 custom web font.
  <pre><code>
SELECT 
  COUNT(*) AS freq,
  ROUND(AVG(SAFE_CAST(JSON_EXTRACT(summary, "$.bytesFont") AS INT64)),2) AS avgFontSize,
  ROUND(APPROX_QUANTILES(SAFE_CAST(JSON_EXTRACT(summary, "$.bytesFont") AS INT64), 100)[SAFE_ORDINAL(50)],2) p50FontSIze,
  ROUND(APPROX_QUANTILES(SAFE_CAST(JSON_EXTRACT(summary, "$.bytesFont") AS INT64), 100)[SAFE_ORDINAL(75)],2) p75FontSIze
FROM `httparchive.all.pages`
WHERE 
  date = "2024-01-01"
  AND is_root_page = true
  AND client = "mobile"
  AND rank <= 1000000
  AND SAFE_CAST(JSON_EXTRACT(summary, "$.reqFont") AS INT64) > 0
  </code></pre>
</details>


<details>
  <summary><b>When do fonts start loading?</b></summary>
  This query will run against the top 100K sites to identify the FCP, LCP and onLoad time for each measurement. It JOINs the requests table to search for the earliest start time for a font download.  Then it groups the results by time interval - such as "Before FCP", "Between FCP and LCP", etc. This query processes ~1TB of data.
  <pre><code>
CREATE TEMP FUNCTION GetLcpTime(json_data STRING)
RETURNS INT64
LANGUAGE js AS """
  var data = JSON.parse(json_data);

  if (data && Array.isArray(data)) {
    for (var i = 0; i < data.length; i++) {
      if (data[i] && data[i].name === 'LargestContentfulPaint') {
        return data[i].time;
      }
    }
  }
  
  return null;
""";

SELECT 
CASE
    WHEN FCP IS null THEN 'error - no FCP'
    WHEN LCP IS null THEN 'error - no LCP'
    WHEN onLoad IS null THEN 'error - no onLoad'
    WHEN firstFontStartTime IS null THEN 'error - no firstFontStartTime'
    WHEN firstFontStartTime < FCP THEN "Before FCP"
    WHEN firstFontStartTime BETWEEN FCP AND LCP THEN "Between FCP and LCP"
    WHEN firstFontStartTime BETWEEN LCP AND onLoad THEN "Between LCP and onLoad"
    WHEN firstFontStartTime >  onLoad THEN "After onLoad"
    ELSE 'Unhandled Error'
  END AS test,
  COUNT(*)
FROM (
  SELECT 
    p.page,
    SAFE_CAST(JSON_EXTRACT(p.payload, "$._firstContentfulPaint") AS INT64) AS FCP,
    getLcpTime(JSON_EXTRACT(p.payload, "$._chromeUserTiming")) AS LCP,
    SAFE_CAST(JSON_EXTRACT(p.summary, "$.onLoad") AS INT64) AS onLoad,
    MIN(SAFE_CAST(JSON_EXTRACT(r.payload, "$._all_start") AS INT64)) AS firstFontStartTime,
  FROM `httparchive.all.pages` AS p
  INNER JOIN `httparchive.all.requests` AS r
  ON CAST(JSON_EXTRACT(p.summary, "$.pageid") AS INT64) = CAST(JSON_EXTRACT(r.summary, "$.pageid") AS INT64)
  WHERE 
    p.date = "2024-01-01" AND r.date = "2023-11-01"
    AND p.is_root_page = true AND r.is_root_page = true
    AND p.client = "mobile" AND r.client = "mobile"
    AND rank <= 100000
    AND r.type = "font"
    AND SAFE_CAST(JSON_EXTRACT(p.summary, "$.reqFont") AS INT64) > 0
  GROUP BY 1,2,3,4
)
GROUP BY 1
ORDER BY 2 DESC
  </code></pre>
</details>

<details>
  <summary><b>How many font bytes are downloaded before FCP?</b></summary>
  This query will run against the top 1 million sites to aggregate the number of bytes loaded before FCP. This query processes ~1.3TB of data.
  <pre><code>

SELECT 
  COUNT(*),
  ROUND(APPROX_QUANTILES(fontBytesBeforeFCP, 100)[SAFE_ORDINAL(50)],2) p50FontSize,
  ROUND(APPROX_QUANTILES(fontBytesBeforeFCP, 100)[SAFE_ORDINAL(75)],2) p75FontSize,
  ROUND(APPROX_QUANTILES(fontBytesBeforeFCP, 100)[SAFE_ORDINAL(95)],2) p95FontSize,
  ROUND(APPROX_QUANTILES(fontBytesBeforeFCP, 100)[SAFE_ORDINAL(99)],2) p99FontSize,
FROM (
  SELECT 
    p.page,
    SUM(SAFE_CAST(JSON_EXTRACT(r.summary, "$.respSize") AS INT64)) AS fontBytesBeforeFCP,
  FROM `httparchive.all.pages` AS p
  INNER JOIN `httparchive.all.requests` AS r
  ON CAST(JSON_EXTRACT(p.summary, "$.pageid") AS INT64) = CAST(JSON_EXTRACT(r.summary, "$.pageid") AS INT64)
  WHERE 
    p.date = "2023-11-01" AND r.date = "2023-11-01"
    AND p.is_root_page = true AND r.is_root_page = true
    AND p.client = "mobile" AND r.client = "mobile"
    AND rank <= 1000000
    AND r.type = "font"
    AND SAFE_CAST(JSON_EXTRACT(p.summary, "$.reqFont") AS INT64) > 0
    AND SAFE_CAST(JSON_EXTRACT(r.payload, "$._all_start") AS INT64) < SAFE_CAST(JSON_EXTRACT(p.payload, "$._firstContentfulPaint") AS INT64)
  GROUP BY 1
)
  </code></pre>
</details>


<details>
  <summary><b>Font glyphs vs rendered HTML</b></summary>
  This query provides a list of the top 10K web sites, individual font URLs, the number of glyphs in the font, and the number of visible glyphs in the rendered HTML.  It also provides a link to the WebPageTest results from the HTTP Archive run for further analysis. This query processes almost 4 TB of data. I stored the results of the query in a scratchspace table in the HTTP Archive `httparchive.scratchspace.2024_01_01_font_glyphs_top100k`
  <pre><code>
CREATE TEMPORARY FUNCTION  CountVisibleGlyphs(html STRING)
RETURNS INT64
LANGUAGE js
AS """
  var extractedText = '';
  if (html) {
    // Remove HTML tags and keep only text content
    extractedText = html.replace(/<[^>]+>/g, '');

    // Remove extra spaces and newlines
    extractedText = extractedText.replace(/\\s+/g, ' ');

    // Remove leading and trailing spaces
    extractedText = extractedText.trim();

    // Count unique characters
    var uniqueChars = new Set(extractedText.split('')).size;
    return uniqueChars;
  } else {
    return 0; // Handle cases with empty HTML content
  }
""";

SELECT 
  pages.page, 
  rank,
  fontRequests.url,
  CAST(JSON_EXTRACT_SCALAR(fontRequests.summary, "$.respBodySize") AS INT64) AS font_size,
  CAST(JSON_EXTRACT_SCALAR(fontRequests.payload, "$._font_details.counts.num_glyphs") AS INT64) AS glyphs,
  CountVisibleGlyphs(htmlRequests.response_body) AS visibleGlyphs,
  CONCAT("https://webpagetest.httparchive.org/result/", wptid, "/") AS webpagetest, 

FROM `httparchive.all.pages` AS pages
INNER JOIN `httparchive.all.requests` AS fontRequests
  ON CAST(JSON_EXTRACT(pages.summary, "$.pageid") AS INT64) = CAST(JSON_EXTRACT(fontRequests.summary, "$.pageid") AS INT64)
INNER JOIN `httparchive.all.requests` AS htmlRequests
  ON CAST(JSON_EXTRACT(pages.summary, "$.pageid") AS INT64) = CAST(JSON_EXTRACT(htmlRequests.summary, "$.pageid") AS INT64)

WHERE 
  pages.date = "2024-01-01" 
  AND fontRequests.date = "2024-01-01"
  AND htmlRequests.date = "2024-01-01"
    
  -- mobile
  AND pages.client="mobile" 
  AND fontRequests.client="mobile"
  AND htmlRequests.client="mobile"
  
  -- root pages
  AND pages.is_root_page = true
  AND fontRequests.is_root_page = true
  AND htmlRequests.is_root_page = true

  -- font requests and HTML request
  AND fontRequests.type = "font"
  AND htmlRequests.is_main_document = true

  AND rank <= 10000

  </code></pre>
</details>


<details>
  <summary><b>Font glyphs vs rendered HTML - analysis</b></summary>
  This query uses the results from the previous table to compared the minimum and maximum number of glyphs in a font to the glyphs in the rendered HTML. Since this query uses the saved results from the previous query, it process a very small amount of data (~1 MB)
  <pre><code>
WITH fontStats AS (
  SELECT page, 
  COUNT(*) AS fonts,
  MIN(CAST(font_size AS INT64)) AS minSize, 
  MAX(CAST(font_size AS INT64)) AS maxSize, 
  SUM(CAST(font_size AS INT64)) AS totalSize, 
  min(CAST(glyphs AS INT64)) AS minGlyphs, 
  MAX(CAST(glyphs AS INT64)) AS maxGlyphs, 
  AVG(visibleGlyphs) AS visibleGlyphs 
FROM `httparchive.scratchspace.2024_01_01_font_glyphs_top100k` 
GROUP BY 1
)

SELECT 
  ROUND(APPROX_QUANTILES(fonts, 100)[SAFE_ORDINAL(50)],2) p50FontCount,
  ROUND(APPROX_QUANTILES(fonts, 100)[SAFE_ORDINAL(75)],2) p75FontCount,
  ROUND(APPROX_QUANTILES(fonts, 100)[SAFE_ORDINAL(95)],2) p95FontCount,
  ROUND(APPROX_QUANTILES(fonts, 100)[SAFE_ORDINAL(99)],2) p99FontCount,
  ROUND(APPROX_QUANTILES(totalSize, 100)[SAFE_ORDINAL(50)],2) p50FontWeight,
  ROUND(APPROX_QUANTILES(totalSize, 100)[SAFE_ORDINAL(75)],2) p75FontWeight,
  ROUND(APPROX_QUANTILES(totalSize, 100)[SAFE_ORDINAL(95)],2) p95FontWeight,
  ROUND(APPROX_QUANTILES(totalSize, 100)[SAFE_ORDINAL(99)],2) p99FontWeight,  
  ROUND(APPROX_QUANTILES(visibleGlyphs, 100)[SAFE_ORDINAL(50)],2) p50VisibleGlyphs,
  ROUND(APPROX_QUANTILES(visibleGlyphs, 100)[SAFE_ORDINAL(75)],2) p75VisibleGlyphs,
  ROUND(APPROX_QUANTILES(visibleGlyphs, 100)[SAFE_ORDINAL(95)],2) p95VisibleGlyphs,
  ROUND(APPROX_QUANTILES(visibleGlyphs, 100)[SAFE_ORDINAL(99)],2) p99VisibleGlyphs,
  ROUND(APPROX_QUANTILES(minGlyphs, 100)[SAFE_ORDINAL(50)],2) p50MinGlyphs,
  ROUND(APPROX_QUANTILES(minGlyphs, 100)[SAFE_ORDINAL(75)],2) p75MinGlyphs,
  ROUND(APPROX_QUANTILES(minGlyphs, 100)[SAFE_ORDINAL(95)],2) p95MinGlyphs,
  ROUND(APPROX_QUANTILES(minGlyphs, 100)[SAFE_ORDINAL(99)],2) p99MinGlyphs,  
  ROUND(APPROX_QUANTILES(maxGlyphs, 100)[SAFE_ORDINAL(50)],2) p50MaxGlyphs,
  ROUND(APPROX_QUANTILES(maxGlyphs, 100)[SAFE_ORDINAL(75)],2) p75MaxGlyphs,
  ROUND(APPROX_QUANTILES(maxGlyphs, 100)[SAFE_ORDINAL(95)],2) p95MaxGlyphs,
  ROUND(APPROX_QUANTILES(maxGlyphs, 100)[SAFE_ORDINAL(99)],2) p99MaxGlyphs,   
FROM fontStats
  </code></pre>
</details>
