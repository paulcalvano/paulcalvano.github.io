---
layout: post
title: "Serving web content with Cloud Storage?  Don't forget the CDN!"
date: 2026-02-08 00:00:00 -0400

---

Many websites use cloud storage as part of how they deliver static content to end users. However using these services without a CDN in front of them has the potential to negatively impact performance. You might be surprised by how many sites do just that - about 8.5% of websites that use a CDN for their primary content, based on [HTTP Archive](https://httparchive.org/) data from January 2026!

**Background**

Content Delivery Networks (CDNs) are an essential component of website delivery, especially when it comes to web performance. According to the [2025 Web Almanac](https://almanac.httparchive.org/en/2025/), approximately [70% of popular websites use them](https://almanac.httparchive.org/en/2025/cdn#fig-3). CDNs enable users to connect to servers that are geographically close to them (and at lower latencies), provide distributed caching to offload backend/origin servers and provide other services such as image optimization, edge compute and security. Some CDNs offer cloud storage services, but pretty much all of them can sit in front of another cloud provider’s storage when it is configured for web delivery.

![WebAlmanac CDN Usage](/assets/img/blog/serving-web-content-with-cloud-storage-dont-forget-the-cdn/httparchive-cdn-usage.jpg){:loading="lazy"}

Many websites with a cloud backend/origin host their static content on cloud storage services such as [Google Cloud Storage](https://cloud.google.com/storage?hl=en), [Amazon S3](https://aws.amazon.com/pm/serv-s3/), [Oracle Cloud ObjectStorage](https://www.oracle.com/cloud/storage/object-storage/) and [Azure Blob Storage](https://azure.microsoft.com/en-us/products/storage/blobs). However it’s important to understand that these services operate as backends and do not automatically provide CDN functionality. Put another way: your content is only behind a CDN if you configure it to be!

In numerous web performance audits over the years, I’ve found cloud storage hostnames being used to deliver static content to end users. During my [Performance Mistakes talk](https://paulcalvano.com/speaking/#:~:text=I%E2%80%99m%20presenting%20in.-,Performance%20Mistakes,-(2024)%20%2D%20Performance%20Now) in November 2024, I shared that there were 580K websites serving content directly from Amazon S3. This has the potential to negatively impact performance, since those resources are generally served from the locations they are hosted in and not a CDN.

I queried the HTTP Archive to see how common this still is and found a few surprising statistics - 

* Overall 5.8% of websites serve content directly from cloud storage! 
* 8.53% of websites that use a CDN for delivering their primary content, serve content directly from cloud storage!
* The number of websites serving content directly from Amazon S3 is now 629K - which is an increase of 8.5% since November 2024! 

![Websites Delivering Assets via Cloud Storage](/assets/img/blog/serving-web-content-with-cloud-storage-dont-forget-the-cdn/websites-delivering-assets-via-cloud-storage.jpg){:loading="lazy"}

**Cloud Storage is not distributed like CDNs**

When you configure cloud storage services, you usually have to define which region your content will be hosted in. This is essentially where your static content will live. When you deliver that content directly via a cloud storage solution, then it will fetch the asset the same as it would if it was hosted on a webserver.

For example, below is a request that was delivered from a European news site. I’m browsing it from the northeast US. The hostname `s3.eu-central-1.amazonaws.com` indicates that this content is being delivered from Amazon’s S3 region in Frankfurt, and I can confirm the same via a traceroute.

When I examine this in Chrome DevTools, I can see very high TCP and TLS connection times for this request! 

![Latency Example](/assets/img/blog/serving-web-content-with-cloud-storage-dont-forget-the-cdn/latency-example.jpg){:loading="lazy"}

**What type of content is delivered directly via Cloud Storage**

In this analysis, I’ve searched for 4 different popular cloud storage providers based on their documented URL structures for web delivery. For Amazon S3 I’m looking for hostnames ending in `amazonaws.com` with `s3` optionally within the hostname. For Google Cloud Storage, I’m looking for a subdomain of `storage.googleapis.com`. For Azure, a subdomain of `windows.net` and for Oracle a subdomain of either `oraclecloud.com` or `customer-oci.com`.

The graph below illustrates the amount of requests delivered by these services across all measured sites, and it’s grouped by content type. Amazon S3 is by far the most commonly used cloud storage service when it comes to this type of delivery, followed by Google Cloud Storage.

![Static Asset Types Delivered by Cloud Storage](/assets/img/blog/serving-web-content-with-cloud-storage-dont-forget-the-cdn/static-asset-types-delivered-by-cloud-storage.jpg){:loading="lazy"}

If we look at the same data by the percentage of requests to each cloud storage service, we can see that regardless of the service almost 75% of content delivered from cloud storage are scripts and images. This type of content is best served to users via a CDN rather than a centralized storage solution.

Delivery of JSON content is also common. While JSON can be dynamically generated, if it’s being served from a cloud storage then it is likely cacheable. HTML delivery is less common, except for Oracle Cloud where it represents 11.8% of cloud storage requests!

![Distribution of Static Asset Types Delivered by Cloud Storage](/assets/img/blog/serving-web-content-with-cloud-storage-dont-forget-the-cdn/distribution-of-static-asset-types-delivered-by-cloud-storage.jpg){:loading="lazy"}


**Caching and Compression**

One thing that may not be apparent when configuring cloud storage for delivery of web content is that compression and downstream caching are often not enabled by default. 

Based on the HTTP Archive, a majority of compressible content types are not being served compressed when delivered directly from Cloud Storage. This is a critical performance mistake, and is often overlooked because most web servers and CDNs do this by default!

![Static Asset Compression When Delivered via Cloud Storage](/assets/img/blog/serving-web-content-with-cloud-storage-dont-forget-the-cdn/static-asset-compression-when-delivered-via-cloud-storage.jpg){:loading="lazy"}

Most content being delivered from cloud storage should be cacheable unless personalized. However the percentage of cloud storage services including Cache-Control headers is incredibly low (with the exception of Google Cloud Storage). That means that clients will have to frequently make requests to these cloud storage services.

<table>
  <tr>
   <td style="text-align: center;"><strong>Service</strong></td>
   <td style="text-align: center;"><strong>Cacheable Requests</strong></td>
   <td style="text-align: center;"><strong>Non-Cacheable Requests</strong></td>
   <td style="text-align: center;"><strong>% Cacheable</strong></td>
   <td style="text-align: center;"><strong>% Non Cacheable</strong></td>
  </tr>
  <tr>
   <td style="text-align: right;">Amazon S3</td>
   <td style="text-align: right;">736,950</td>
   <td style="background-color: #ffe5e5; text-align: right;">2,587,320</td>
   <td style="text-align: right;">22.2%</td>
   <td style="background-color: #ffe5e5; text-align: right;">77.8%</td>
  </tr>
  <tr>
   <td style="text-align: right;">Google Cloud Storage</td>
   <td style="text-align: right;">969,660</td>
   <td style="background-color: #ffe5e5; text-align: right;">259,629</td>
   <td style="text-align: right;">78.9%</td>
   <td style="background-color: #fff4e0; text-align: right;">21.1%</td>
  </tr>
  <tr>
   <td style="text-align: right;">Azure Blob Storage</td>
   <td style="text-align: right;">102,857</td>
   <td style="background-color: #ffe5e5; text-align: right;">376,291</td>
   <td style="text-align: right;">21.5%</td>
   <td style="background-color: #ffe5e5; text-align: right;">78.5%</td>
  </tr>
  <tr>
   <td style="text-align: right;">Oracle Cloud Object Storage</td>
   <td style="text-align: right;">6,763</td>
   <td style="background-color: #ffe5e5; text-align: right;">27,877</td>
   <td style="text-align: right;">19.5%</td>
   <td style="background-color: #ffe5e5; text-align: right;">80.5%</td>
  </tr>
</table>


Digging a bit deeper we can see that images, JS, JSON and CSS account for most of the non-cacheable content delivered via Amazon S3. These asset types are often delivered with cache-control headers allowing caching from Google Cloud Storage.

<table>
  <tr>
   <td style="text-align: center;"></td>
   <td style="text-align: center;" colspan="2">Amazon S3</td>
   <td style="text-align: center;" colspan="2">Google Cloud Storage</td>
   <td style="text-align: center;" colspan="2">Azure Blob Storage</td>
   <td style="text-align: center;" colspan="2">Oracle Cloud Object Storage</td>
  </tr>
  <tr>
   <td style="text-align: center;"><em>type</em></td>
   <td style="text-align: center;">cacheable</td>
   <td style="text-align: center;">not-cacheable</td>
   <td style="text-align: center;">cacheable</td>
   <td style="text-align: center;">not-cacheable</td>
   <td style="text-align: center;">cacheable</td>
   <td style="text-align: center;">not-cacheable</td>
   <td style="text-align: center;">cacheable</td>
   <td style="text-align: center;">not-cacheable</td>
  </tr>
  <tr>
   <td style="text-align: center;">image</td>
   <td style="text-align: right;">561,592</td>
   <td style="background-color: #ffe5e5; text-align: right;">1,743,876</td>
   <td style="text-align: right;">489,387</td>
   <td style="background-color: #ffe5e5; text-align: right;">131,988</td>
   <td style="text-align: right;">84,912</td>
   <td style="background-color: #ffe5e5; text-align: right;">284,536</td>
   <td style="text-align: right;">4,668</td>
   <td style="background-color: #ffe5e5; text-align: right;">18,132</td>
  </tr>
  <tr>
   <td style="text-align: center;">script</td>
   <td style="text-align: right;">91,890</td>
   <td style="background-color: #ffe5e5; text-align: right;">272,434</td>
   <td style="text-align: right;">370,127</td>
   <td style="background-color: #ffe5e5; text-align: right;">45,621</td>
   <td style="text-align: right;">5,378</td>
   <td style="background-color: #ffe5e5; text-align: right;">26,061</td>
   <td style="text-align: right;">1,658</td>
   <td style="background-color: #ffe5e5; text-align: right;">2,599</td>
  </tr>
  <tr>
   <td style="text-align: center;">json</td>
   <td style="text-align: right;">41,046</td>
   <td style="background-color: #ffe5e5; text-align: right;">201,543</td>
   <td style="text-align: right;">48,534</td>
   <td style="background-color: #ffe5e5; text-align: right;">27,917</td>
   <td style="text-align: right;">303</td>
   <td style="background-color: #ffe5e5; text-align: right;">6,083</td>
   <td style="text-align: right;">263</td>
   <td style="background-color: #ffe5e5; text-align: right;">1,126</td>
  </tr>
  <tr>
   <td style="text-align: center;">css</td>
   <td style="text-align: right;">26,926</td>
   <td style="background-color: #ffe5e5; text-align: right;">103,007</td>
   <td style="text-align: right;">30,893</td>
   <td style="background-color: #ffe5e5; text-align: right;">1,340</td>
   <td style="text-align: right;">2,919</td>
   <td style="background-color: #ffe5e5; text-align: right;">18,666</td>
   <td style="text-align: right;">141</td>
   <td style="background-color: #ffe5e5; text-align: right;">863</td>
  </tr>
  <tr>
   <td style="text-align: center;">xml</td>
   <td style="text-align: right;">40</td>
   <td style="background-color: #ffe5e5; text-align: right;">85,778</td>
   <td style="text-align: right;">193</td>
   <td style="background-color: #ffe5e5; text-align: right;">11,909</td>
   <td style="text-align: right;">9</td>
   <td style="background-color: #ffe5e5; text-align: right;">9,186</td>
   <td style="text-align: right;"></td>
   <td style="background-color: #ffe5e5; text-align: right;">33</td>
  </tr>
  <tr>
   <td style="text-align: center;">other</td>
   <td style="text-align: right;">2,111</td>
   <td style="background-color: #ffe5e5; text-align: right;">82,043</td>
   <td style="text-align: right;">4,623</td>
   <td style="background-color: #ffe5e5; text-align: right;">2,170</td>
   <td style="text-align: right;">408</td>
   <td style="background-color: #ffe5e5; text-align: right;">14,539</td>
   <td style="text-align: right;"></td>
   <td style="background-color: #ffe5e5; text-align: right;">903</td>
  </tr>
  <tr>
   <td style="text-align: center;">font</td>
   <td style="text-align: right;">13,002</td>
   <td style="background-color: #ffe5e5; text-align: right;">33,652</td>
   <td style="text-align: right;">24,619</td>
   <td style="background-color: #ffe5e5; text-align: right;">1,344</td>
   <td style="text-align: right;">8,242</td>
   <td style="background-color: #ffe5e5; text-align: right;">8,511</td>
   <td style="text-align: right;">30</td>
   <td style="background-color: #ffe5e5; text-align: right;">31</td>
  </tr>
  <tr>
   <td style="text-align: center;">video</td>
   <td style="text-align: right;">191</td>
   <td style="background-color: #ffe5e5; text-align: right;">37,699</td>
   <td style="text-align: right;">441</td>
   <td style="background-color: #ffe5e5; text-align: right;">19,435</td>
   <td style="text-align: right;">680</td>
   <td style="background-color: #ffe5e5; text-align: right;">5,183</td>
   <td style="text-align: right;"></td>
   <td style="background-color: #ffe5e5; text-align: right;">79</td>
  </tr>
  <tr>
   <td style="text-align: center;">text</td>
   <td style="text-align: right;">148</td>
   <td style="background-color: #ffe5e5; text-align: right;">16,395</td>
   <td style="text-align: right;">386</td>
   <td style="background-color: #ffe5e5; text-align: right;">4,245</td>
   <td style="text-align: right;">2</td>
   <td style="background-color: #ffe5e5; text-align: right;">2,840</td>
   <td style="text-align: right;"></td>
   <td style="background-color: #ffe5e5; text-align: right;">37</td>
  </tr>
  <tr>
   <td style="text-align: center;">html</td>
   <td style="text-align: right;"></td>
   <td style="background-color: #ffe5e5; text-align: right;">4,350</td>
   <td style="text-align: right;">20</td>
   <td style="background-color: #ffe5e5; text-align: right;">10,382</td>
   <td style="text-align: right;"></td>
   <td style="background-color: #ffe5e5; text-align: right;">541</td>
   <td style="text-align: right;"></td>
   <td style="background-color: #ffe5e5; text-align: right;">4,071</td>
  </tr>
  <tr>
   <td style="text-align: center;">audio</td>
   <td style="text-align: right;">3</td>
   <td style="background-color: #ffe5e5; text-align: right;">6,514</td>
   <td style="text-align: right;">431</td>
   <td style="background-color: #ffe5e5; text-align: right;">3,278</td>
   <td style="text-align: right;">4</td>
   <td style="background-color: #ffe5e5; text-align: right;">138</td>
   <td style="text-align: right;">3</td>
   <td style="background-color: #ffe5e5; text-align: right;">3</td>
  </tr>
  <tr>
   <td style="text-align: center;">wasm</td>
   <td style="text-align: right;">1</td>
   <td style="background-color: #ffe5e5; text-align: right;">29</td>
   <td style="text-align: right;">6</td>
   <td style="text-align: right;"></td>
   <td style="text-align: right;"></td>
   <td style="background-color: #ffe5e5; text-align: right;">7</td>
   <td style="text-align: right;"></td>
   <td style="text-align: right;"></td>
  </tr>
</table>


Here’s an example from a popular movie theater chain in the US. This content appears to be loaded by a 3rd party (Unbounce), and that third party is configured to load images directly from S3. There are no cache headers present, which means that the browser will [heuristically cache](https://paulcalvano.com/2018-03-14-http-heuristic-caching-missing-cache-control-and-expires-headers-explained/) the resources. On this particular page, this third party’s S3 content accounted for over 9MB of content - none of them containing a `cache-control` header! Beyond that there are a few opportunities for image optimization that could be applied.

![Caching Example](/assets/img/blog/serving-web-content-with-cloud-storage-dont-forget-the-cdn/caching-example.jpg){:loading="lazy"}

In an example from another movie theater’s website, we can see render-blocking CSS and JS loaded from Amazon S3, with no `cache-control` header to indicate caching, and no compression. This is perhaps the worst case scenario - where you have content critical to the rendering of your website that is loaded slowly from a centralized location, with unnecessary large payloads and then unpredictable caching (or no caching) due to a missing `cache-control` header.

![Caching Compression Example](/assets/img/blog/serving-web-content-with-cloud-storage-dont-forget-the-cdn/caching-compression-example.jpg){:loading="lazy"}

**Cloud Storage vs CDN Delivery Costs**

It’s also worth evaluating how much delivering traffic directly from these cloud storage solutions is costing. Depending on your contracts with the providers, you may find that delivering directly via cloud storage is more expensive compared to CDN delivery - especially if you are not caching or compressing the content! If that is the case, you can get a double-win by saving money while improving performance!

**Conclusion**

It's incredible that 8.5% of websites that utilize a CDN are delivering content to users directly from cloud storage services. Discovering and fixing these could provide a quick performance boost. When you audit your website’s performance, if you notice cloud storage hostnames then you should definitely investigate how they got there, and move that content behind your CDNs.

**HTTP Archive queries**

This section provides some details on how this analysis was performed, including SQL queries. Please be warned that some of the SQL queries process a significant amount of bytes - which can be very expensive to run.

<details>
  <summary><b>What Percentage of CDN delivered sites have request for Cloud Storage hosted assets</b></summary>
   This query counts the number of sites that contain at least 1 request for an asset delivered directly via a Cloud Storage service (without a CDN).
  <pre><code>
 SELECT
    IF(JSON_VALUE(p.summary.cdn) IS NOT NULL AND NOT JSON_VALUE(p.summary.cdn) = "",true, false) AS usesCDN,
    CASE
       WHEN NET.HOST(url) LIKE "%.amazonaws.com" THEN "Amazon S3"
       WHEN NET.HOST(url) LIKE "%storage.googleapis.com" THEN "Google Cloud Storage"
       WHEN NET.HOST(url) LIKE "%.windows.net" THEN "Azure Blob Storage"
       WHEN NET.HOST(url) LIKE "%.oraclecloud.com" THEN "Oracle Cloud Object Storage"
       WHEN NET.HOST(url) LIKE "%.customer-oci.com" THEN "Oracle Cloud Object Storage"
       ELSE "Unknown"
    END AS CloudStorage,
    COUNT(DISTINCT p.page) AS sites
FROM `httparchive.crawl.requests` AS r
INNER JOIN `httparchive.crawl.pages` AS p
ON r.page = p.page
WHERE
   p.date = "2026-01-01" AND r.date = "2026-01-01"
   AND p.client = "mobile" AND r.client = "mobile"
   AND p.is_root_page = TRUE AND r.is_root_page = TRUE
   AND (
     NET.HOST(url) LIKE "%.amazonaws.com"
     OR NET.HOST(url) LIKE "%storage.googleapis.com"
     OR NET.HOST(url) LIKE "%.windows.net"
     OR NET.HOST(url) LIKE "%.oraclecloud.com"
     OR NET.HOST(url) LIKE "%.customer-oci.com"
   )
GROUP BY 1,2
</code></pre>
</details>

<details>
  <summary><b>Caching and Compression of Cloud Storage Delivered Assets</b></summary>
   This query counts the number of requests being delivered cached and compressed by content type. 
  <pre><code>
 
SELECT
  IF(JSON_VALUE(p.summary.cdn) IS NOT NULL AND NOT JSON_VALUE(p.summary.cdn) = "",true, false) AS usesCDN,
  CASE
    WHEN NET.HOST(url) LIKE "%.amazonaws.com" THEN "Amazon S3"
    WHEN NET.HOST(url) LIKE "%storage.googleapis.com" THEN "Google Cloud Storage"
    WHEN NET.HOST(url) LIKE "%.windows.net" THEN "Azure Blob Storage"
    WHEN NET.HOST(url) LIKE "%.oraclecloud.com" THEN "Oracle Cloud Object Storage"
    WHEN NET.HOST(url) LIKE "%.customer-oci.com" THEN "Oracle Cloud Object Storage"
    ELSE "Unknown"
  END AS CloudStorage,
  JSON_VALUE(r.summary.type) AS type,
  CASE JSON_VALUE(r.payload._contentEncoding)
    WHEN 'gzip' THEN 'Gzip'
    WHEN 'br' THEN 'Brotli'
    WHEN 'zstd' THEN 'zStandard'
    ELSE 'No compression'
  END AS compression_type,
  IF(SAFE_CAST(JSON_VALUE(r.payload._cache_time) AS INT64) > 0, "cacheable", "not-cacheable") AS cacheable,
  COUNT(DISTINCT p.page) AS sites,
  COUNT(*) AS requests
FROM `httparchive.crawl.requests` AS r
INNER JOIN `httparchive.crawl.pages` AS p
ON r.page = p.page
WHERE
  p.date = "2026-01-01" AND r.date = "2026-01-01"
  AND p.client = "mobile" AND r.client = "mobile"
  AND p.is_root_page = TRUE AND r.is_root_page = TRUE
  AND (
      NET.HOST(url) LIKE "%.amazonaws.com"
      OR NET.HOST(url) LIKE "%storage.googleapis.com"
      OR NET.HOST(url) LIKE "%.windows.net"
      OR NET.HOST(url) LIKE "%.oraclecloud.com"
      OR NET.HOST(url) LIKE "%.customer-oci.com"
  )
GROUP BY 1,2,3,4,5
</code></pre>
</details>

<details>
  <summary><b>Examples of Sites that load content from cloud storage services</b></summary>
   Detailed examples of sites that are loading content from cloud storage services. 
  <pre><code>
 SELECT
   p.rank,
   r.page AS page,
   JSON_VALUE(p.summary.cdn) AS pageCDN,
   NET.HOST(r.url) AS hostname,
   COUNT(*) AS requests,
   SUM(CAST(JSON_VALUE(r.summary.respBodySize) AS INT64)/1024/1024) AS responseMB,
   STRING_AGG(DISTINCT JSON_VALUE(r.summary.type)) AS types,
   FROM `httparchive.crawl.requests` AS r
INNER JOIN `httparchive.crawl.pages` AS p
ON r.page = p.page
WHERE
   p.date = "2026-01-01" AND r.date = "2026-01-01"
   AND p.client = "mobile" AND r.client = "mobile"
   AND p.is_root_page = TRUE AND r.is_root_page = TRUE
   AND p.rank <= 1000 AND r.rank <= 1000
   AND (
     NET.HOST(url) LIKE "%.amazonaws.com"
     OR NET.HOST(url) LIKE "%storage.googleapis.com"
     OR NET.HOST(url) LIKE "%.windows.net"
     OR NET.HOST(url) LIKE "%.oraclecloud.com"
     OR NET.HOST(url) LIKE "%.customer-oci.com"
   )
GROUP BY 1,2,3,4
ORDER BY 6 DESC

  </code></pre>
</details>

