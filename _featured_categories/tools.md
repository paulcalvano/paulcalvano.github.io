---
# Featured tags need to have either the `list` or `grid` layout (PRO only).
layout: about

# The title of the tag's page.
title: Tools

# The name of the tag, used in a post's front matter (e.g. tags: [<slug>]).
slug: 

# (Optional) Write a short (~150 characters) description of this featured tag.
description: >
  

# (Optional) You can disable grouping posts by date.
# no_groups: true
---

**Compression Tester**

Most web servers and CDNs deliver gzip compressed payloads by default, but many also support Brotli as well. And as of March 2024 Chrome now supports zStandard compression. This tool helps you understand how each of these compression methods can compress your content. It works by downloading compressed payloads from your server, and then attempting to compress it at multiple gzip, brotli and zStandard compression levels.  THe results include both the payload size as well as the time it took to compress at each compression level

[![Compression Tester](/assets/img/blog/choosing-between-gzip-brotli-and-zstandard-compression/sandals_homepage_compression.jpg)](https://tools.paulcalvano.com/compression-tester/)

You can find this tool [here](https://tools.paulcalvano.com/compression-tester/).


**Web Font Analyzer** 

Do you know how many web fonts are loaded on your site? How about their size and the number of glyphs contained in them? There are quite a few sites that load large font files - some containing hundreds or thousands of glyphs. This can be problematic for web performance, since fonts have a high loading priority. I wrote a blog post about this [here](/2024-02-16-identifying-font-subsetting-opportunities).

The Web Font Analyzer tool will take a WebPageTest measurement that you provide and show you some information related to the fonts.  When are they loading? How many glyphs do they contain?  How many visible glyphs are on the page. This data might prove helpful when deciding on whether to subset your fonts.

[![Web Font Analyzer](/assets/img/blog/identifying-font-subsetting-opportunities/web-font-analyzer.jpg)](https://tools.paulcalvano.com/wpt-font-analysis/)

You can find this tool [here](https://tools.paulcalvano.com/wpt-font-analysis/).

**WebPageTest Cookies Script**

When running a [WebPageTest](https://webpagetest.org)  measurement as a return visitor, sometimes the page might be rendered differently based on the cookies that the end user has recieved.  Sometimes the sheer size of the cookies can also [impact TTFB](/2020-07-13-an-analysis-of-cookie-sizes-on-the-web/).  And sometimes you just want to test a website behind a login. In order to test all of these scenarios, you would need to include cookies on your initial request. Fortunately, this is something that you can easily do with the WebPageTest [script](https://sites.google.com/a/webpagetest.org/docs/using-webpagetest/scripting#TOC-setCookie) `setCookie`.

While adding this script step is fairly simple, sometimes the page you are trying to test has a large number of cookies. This can become cumbersome. This  utility parses HTTP request headers to generate a WebPageTest script that includes all of the requested cookies. It can parse both HTTP/1.1 and HTTP/2 request headers (the main differences are case sensitivity in header names and use of `:authority` and `:path` for HTTP/2 instead of `Host` and the the path from the `GET`line in HTTP/1.1)

[![](/assets/img/blog/tools/wpt-cookies.jpg)](http://htmlpreview.github.io/?https://github.com/paulcalvano/requestHeaders-to-WPT-script/blob/master/request-headers-to-wpt-script.html)

You can find the tool [here](http://htmlpreview.github.io/?https://github.com/paulcalvano/requestHeaders-to-WPT-script/blob/master/request-headers-to-wpt-script.html)


**Gzip/Brotli Compression Estimator** 

Ever wonder how effective your web server or CDN is at compressing your text based assets?  While it's easy to confirm that a response was compressed (by looking at 	`Content-Encoding` headers, the compression level that was used is not indicated anywhere in the response. This tool attempts to download both gzip and brotli compressed resources from your server, and estimates their compression levels.  It also compresses the file at each level of gzip and brotli compression so that you can see what the potential savings are. I wrote a blog post about this [here](/2018-07-25-brotli-compression-how-much-will-it-reduce-your-content/)
[![](/assets/wp-content/uploads/2018/07/compression_estimator_jquery.jpg)](https://tools.paulcalvano.com/compression.php)

You can find this tool [here](https://tools.paulcalvano.com/compression.php).

**webfont usage analyzer**

Custom web font usage has grown considerably over the past few years, and now almost every site you visit has them. But how often are they used on a page?  This tool will help you visualize where a font stack is used on a page. This can help technical teams and designers collaborate on the balance between typography and web performance. I wrote more about custom web font usage, and this tool [here](/2017-07-25-performance-and-usage-implications-of-custom-fonts/).

[![](/assets/wp-content/uploads/2017/07/developer_akamai_example.jpg)](https://github.com/paulcalvano/webfont-usage-analyzer)

You can find this tool [here](https://github.com/paulcalvano/webfont-usage-analyzer).


