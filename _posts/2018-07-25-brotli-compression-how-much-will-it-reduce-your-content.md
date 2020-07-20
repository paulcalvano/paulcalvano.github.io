---
title: 'Brotli Compression: How Much Will It Reduce Your Content?'
date: 2018-07-25T12:42:24+00:00
author: Paul Calvano
layout: post
---
A few years ago [Brotli](https://github.com/google/brotli) compression entered into the webperf spotlight with impressive gains of up to 25% over gzip compression. The algorithm was created by Google, who initially introduced it as a way to compress web fonts via the [woff2 format](https://developers.googleblog.com/2015/02/smaller-fonts-with-woff-20-and-unicode.html). Later in 2015 it was released as a [compression library to optimize the delivery of web content](https://opensource.googleblog.com/2015/09/introducing-brotli-new-compression.html). Despite Brotli being a completely different format from Gzip, it was quickly [supported by most modern web browsers](https://caniuse.com/#feat=brotli).

[<img src="http://paulcalvano.com/wp-content/uploads/2018/07/broti_caniuse.jpg" alt="" width="1272" height="427" class="alignnone size-full wp-image-503" srcset="http://paulcalvano.com/wp-content/uploads/2018/07/broti_caniuse.jpg 1272w, http://paulcalvano.com/wp-content/uploads/2018/07/broti_caniuse-300x101.jpg 300w, http://paulcalvano.com/wp-content/uploads/2018/07/broti_caniuse-768x258.jpg 768w, http://paulcalvano.com/wp-content/uploads/2018/07/broti_caniuse-1024x344.jpg 1024w, http://paulcalvano.com/wp-content/uploads/2018/07/broti_caniuse-700x235.jpg 700w" sizes="(max-width: 1272px) 100vw, 1272px" />](http://paulcalvano.com/wp-content/uploads/2018/07/broti_caniuse.jpg)

<!--more-->

By mid-2016 some websites and CDNs started to support it. However, adoption across the web is still quite low. Based on both Desktop and Mobile data from the HTTP Archive&#8217;s June 2018 dataset, only 71% of text based compressible resources (html, javascript, css, json, xml, svg, etc) are actually being compressed! Of them, 11% are Brotli encoded and 60% are Gzip encoded.

[<img src="http://paulcalvano.com/wp-content/uploads/2018/07/brotli_httparchive.jpg" alt="" width="634" height="331" class="alignnone size-full wp-image-504" srcset="http://paulcalvano.com/wp-content/uploads/2018/07/brotli_httparchive.jpg 634w, http://paulcalvano.com/wp-content/uploads/2018/07/brotli_httparchive-300x157.jpg 300w" sizes="(max-width: 634px) 100vw, 634px" />](http://paulcalvano.com/wp-content/uploads/2018/07/brotli_httparchive.jpg)

If you are not using Brotli today, have you ever wondered how much Brotli compression could reduce your content? Read on to find out!

**Compression Levels**

Both Gzip and Brotli use multiple compression levels to indicate how aggressive the algorithms will work to compress a file. With both Gzip and Brotli, the compression levels start at 1, and increase. The higher the compression level, the smaller the file (and the more computationally expensive the compression is). The below graph, which was generated from [Squash benchmark results](https://quixdb.github.io/squash-benchmark/), makes this very clear.

[<img src="http://paulcalvano.com/wp-content/uploads/2018/07/gzip_brotli_squash_benchmark.jpg" alt="" width="645" height="421" class="alignnone size-full wp-image-507" srcset="http://paulcalvano.com/wp-content/uploads/2018/07/gzip_brotli_squash_benchmark.jpg 645w, http://paulcalvano.com/wp-content/uploads/2018/07/gzip_brotli_squash_benchmark-300x196.jpg 300w" sizes="(max-width: 645px) 100vw, 645px" />](http://paulcalvano.com/wp-content/uploads/2018/07/gzip_brotli_squash_benchmark.jpg)

Many popular web servers default to a mid-range gzip level, because it compresses the file adequately while keeping CPU costs in check. For example: Apache defaults to zlib&#8217;s default, which is [level 6](https://httpd.apache.org/docs/2.4/mod/mod_deflate.html#deflatecompressionlevel), IIS defaults to [level 7](https://docs.microsoft.com/en-us/iis/configuration/system.webServer/httpCompression/scheme), NGINX defaults to [level 1](http://nginx.org/en/docs/http/ngx_http_gzip_module.html#gzip_comp_level). When sites can afford to, compressing to Gzip level 9 will shave off a few more bytes. The same is true for Brotli, although the CPU costs are much higher compared to Gzip. Brotli compression levels 10 and 11 are far more computationally expensive &#8211; but the savings are significant. If you are able to precompress resources, then Brotli level 11 is the way to go. If not, then Brotli level 4 or 5 should provide a smaller payload than the highest gzip compression level, with a reasonable processing time tradeoff.

Note: Akamai is able to compress to Brotli level 11 because of the way Resource Optimizer is architected. Brotli resources are only served from Akamai&#8217;s cache, and cache misses are precompressed prior to being served to an end user. This allows us to provide the most byte savings without the user incurring any processing delays.

**How Much Will My Files Be Compressed With Brotli?**

I&#8217;ve created a new tool which you can access here: . <https://tools.paulcalvano.com/compression.php>

Here&#8217;s how it works:

  1. Find a URL that you want to test the compression levels for. For example, you may want to choose the largest JS or CSS file on a page. 
  2. Paste the URL for the object you want to test in the text box, and click the &#8220;Compression Test&#8221; button. 
  3. On the server side, the file will be downloaded 3 times. Once w/o an Accept-Encoding header, and then with Accept-Encoding:gzip header and finally with an Accept-Encoding:br header. This allows us to log the uncompressed file size, and whether gzip and brotli are currently supported. 
  4. The uncompressed version of the file is then compressed at each level for Gzip and Brotli. Compression ratios are calculated, and the compression levels used by the website are estimated based on file size.

[<img src="http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_tool.jpg" alt="" width="741" height="740" class="alignnone size-full wp-image-508" srcset="http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_tool.jpg 741w, http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_tool-150x150.jpg 150w, http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_tool-300x300.jpg 300w, http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_tool-700x700.jpg 700w" sizes="(max-width: 741px) 100vw, 741px" />](http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_tool.jpg)

**Example:**

Let&#8217;s take a look at a common JavaScript file that many sites are currently using. Below I&#8217;ve tested the minified jQuery UI 1.12 library from https://code.jquery.com/. I can see that this file is 253KB uncompressed, and that gzip compression reduces the size to 83KB. However the Gzip compression level is quite low, and increasing it to Gzip level 9 would shave off 16KB from the download. Furthermore, using Brotli compression drops the file down to 57KB, which is 26KB (32%) smaller than the Gzip compressed version that was served!

[<img src="http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_jquery.jpg" alt="" width="1011" height="785" class="alignnone size-full wp-image-527" srcset="http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_jquery.jpg 1011w, http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_jquery-300x233.jpg 300w, http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_jquery-768x596.jpg 768w, http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_jquery-700x544.jpg 700w" sizes="(max-width: 1011px) 100vw, 1011px" />](http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_jquery.jpg)

Now let&#8217;s examine another site with a very large JavaScript file. I ran a query against the HTTP Archive for the largest JavaScript file used by a site in the Alexa top 1000. I won&#8217;t shame the lucky winner here, but the file size was 4.3MB uncompressed and was gzip compressed down to 1.04 MB. Brotli level 11 would reduce this another 27% to 776KB

[<img src="http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_largejs.jpg" alt="" width="951" height="767" class="alignnone size-full wp-image-513" srcset="http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_largejs.jpg 951w, http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_largejs-300x242.jpg 300w, http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_largejs-768x619.jpg 768w, http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_largejs-700x565.jpg 700w" sizes="(max-width: 951px) 100vw, 951px" />](http://paulcalvano.com/wp-content/uploads/2018/07/compression_estimator_largejs.jpg)

Go ahead and try it on your content! <https://tools.paulcalvano.com/compression.php>

**Implementing Brotli @ Akamai**

At Akamai, we support Brotli in two ways and both implementations are supported via a simple On/Off toggle:

  * Brotli Served from Origin
  * Resource Optimizer 

By default Akamai sends an Accept-Encoding header to origins advertising support for Gzip compression. The Brotli Support feature instructs the edge servers to send an Accept-Encoding header that advertises support for both Brotli and Gzip, and will cache both Brotli and Gzip versions of your assets.

Resource Optimizer is a simple turnkey feature that allows you to serve Gzip compressed resources from your origin while Akamai compresses the resources for you with Brotli. The Brotli compressed resources are precompressed, encoded at level 11 compression and loaded into cache the first time they are requested, so that no user will experience a processing delay in downloading Brotli compressed resources. To turn it on, simply enable the &#8220;Resource Optimizer&#8221; feature of Adaptive Acceleration &#8211;

[<img src="http://paulcalvano.com/wp-content/uploads/2018/07/brotli_support_akamai.jpg" alt="" width="701" height="425" class="alignnone size-full wp-image-524" srcset="http://paulcalvano.com/wp-content/uploads/2018/07/brotli_support_akamai.jpg 701w, http://paulcalvano.com/wp-content/uploads/2018/07/brotli_support_akamai-300x182.jpg 300w, http://paulcalvano.com/wp-content/uploads/2018/07/brotli_support_akamai-700x424.jpg 700w" sizes="(max-width: 701px) 100vw, 701px" />](http://paulcalvano.com/wp-content/uploads/2018/07/brotli_support_akamai.jpg)

**Conclusion**

Many sites are using Gzip compression already, and there are existing tools that will tell you if you are not compressing your content. The tool I created will tell you what Gzip and Brotli compression levels you are most likely using and what is possible for your assets at each compression level. If you are unsure of how much Brotli can benefit your content, then hopefully this helps!