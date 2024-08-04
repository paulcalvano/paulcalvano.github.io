---
layout: post
title: "Third Parties and Certificate Revocations"
date: 2024-08-04 00:00:00 -0400
related_posts:
 
  - _posts/2020-02-20-certificate-validity-dates.markdown
  - _posts/2020-02-17-san-certificates-how-many-alt-names-are-too-many.md

---

On Monday June 29th, DigiCert [announced](https://www.digicert.com/support/certificate-revocation-incident) the need to revoke a large number of certificates due to a mistake in domain validation. The CA/B Forum [requirements](https://cabforum.org/working-groups/server/baseline-requirements/documents/) to revoke certificates within 24 hours are pretty strict, and suffice to say Monday night and Tuesday were pretty busy days for a lot of folks. What remained a mystery was how many sites and third parties would be affected, how many would be prepared in time and what the impact of a mass revocation might look like across the web. In this blog post we’ll use the [HTTP Archive](https://httparchive.org/) to explore the impact.

**Which hostnames were affected?**

In a [bugzilla](https://bugzilla.mozilla.org/show_bug.cgi?id=1910322) post, DigiCert shared a list of 86k serial numbers for certificates that were affected and needed to be revoked.  The list didn’t include hostnames, so it wasn’t easy to see which domains would be affected from the files alone. The [HTTP Archive](https://httparchive.org/) contains details for every certificate it encounters, so I was able write a query to correlate the list of serial numbers with certificate used across 16 million websites.  <i>If you are interested in the queries used to do this analysis, you'll find them at the end of this post.</i>

I found 13,823 of the certificate serial numbers on publicly available web pages from last month's HTTP Archive crawl. Many of these were first party resources, but a few hundred belonged to popular third parties. Overall I found that 1,241,943 websites would have been affected by this revocation in some way, meaning they either made a first or third party request for a resource that used at least one of the affected certificates! 

The original deadline for the certificate revocations came and passed, but DigiCert was granted an extension due to extenuating circumstances. Given the amount of websites serving content on the affected certificates, that 24 hour notice and subsequent extension likely saved the web from a significant amount of disruption! 

Here’s a list of some of the more popular domains that were affected. The list contains an apex domain, the number of sites requesting resources from it, and the number of subdomains domains that contained certificates needing to be revoked.

<table>
  <tr>
   <td colspan="3" align="center"><b>Third Party with Certificates Affected by DigiCert Recovations</b></td>
  </tr>
  <tr>
   <td>Domain</td>
   <td>Number of Sites</td>
   <td>Number of Hostnames</td>
  </tr>
  <tr>
   <td>yahoo.com</td>
   <td><p style="text-align: right">467,827</p></td>
   <td><p style="text-align: right">49</p></td>
  </tr>
  <tr>
   <td>rubiconproject.com</td>
   <td><p style="text-align: right">387,241</p></td>
   <td><p style="text-align: right">20</p></td>
  </tr>
  <tr>
   <td>fontawesome.com</td>
   <td><p style="text-align: right">299,959</p></td>
   <td><p style="text-align: right">10</p></td>
  </tr>
  <tr>
   <td>pinterest.com</td>
   <td><p style="text-align: right">281,145</p></td>
   <td><p style="text-align: right">57</p></td>
  </tr>
  <tr>
   <td>taboola.com</td>
   <td><p style="text-align: right">133,309</p></td>
   <td><p style="text-align: right">43</p></td>
  </tr>
  <tr>
   <td>pinimg.com</td>
   <td><p style="text-align: right">91,573</p></td>
   <td><p style="text-align: right">14</p></td>
  </tr>
  <tr>
   <td>ib-ibi.com</td>
   <td><p style="text-align: right">60,557</p></td>
   <td><p style="text-align: right">1</p></td>
  </tr>
  <tr>
   <td>snapchat.com</td>
   <td><p style="text-align: right">49,390</p></td>
   <td><p style="text-align: right">15</p></td>
  </tr>
  <tr>
   <td>advertising.com</td>
   <td><p style="text-align: right">41,815</p></td>
   <td><p style="text-align: right">1</p></td>
  </tr>
  <tr>
   <td>datadoghq-browser-agent.com</td>
   <td><p style="text-align: right">35,404</p></td>
   <td><p style="text-align: right">1</p></td>
  </tr>
  <tr>
   <td>sift.com</td>
   <td><p style="text-align: right">13,962</p></td>
   <td><p style="text-align: right">1</p></td>
  </tr>
  <tr>
   <td>scdn.co</td>
   <td><p style="text-align: right">10,949</p></td>
   <td><p style="text-align: right">1</p></td>
  </tr>
  <tr>
   <td>usonar.jp</td>
   <td><p style="text-align: right">7,402</p></td>
   <td><p style="text-align: right">4</p></td>
  </tr>
  <tr>
   <td>sojern.com</td>
   <td><p style="text-align: right">7,371</p></td>
   <td><p style="text-align: right">4</p></td>
  </tr>
  <tr>
   <td>olark.com</td>
   <td><p style="text-align: right">6,966</p></td>
   <td><p style="text-align: right">1</p></td>
  </tr>
</table>
<p>&nbsp;</p>
Looking at a few of the popular third party domains, you can see that many of them updated their certificates on July 30th.  The initial deadline to reissue certificates was July 30th 19:30 UTC. 

<table>
  <tr>
   <td colspan="3" align="center"><b>Third Party Hostnames Affected by DigiCert Revocation</b></td>
  </tr>
  <tr>
   <td><strong>Host</strong></td>
   <td><strong>ValidFrom</strong></td>
   <td><strong>ValidTo</strong></td>
  </tr>
  <tr>
   <td>pixel.rubiconproject.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Apr 3 23:59:59 2025 GMT</td>
  </tr>
  <tr>
   <td>pr-bh.ybp.yahoo.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Jan 22 23:59:59 2025 GMT</td>
  </tr>
  <tr>
   <td>ups.analytics.yahoo.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Jan 22 23:59:59 2025 GMT</td>
  </tr>
  <tr>
   <td>kit.fontawesome.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Jan 27 23:59:59 2025 GMT</td>
  </tr>
  <tr>
   <td>token.rubiconproject.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Apr 3 23:59:59 2025 GMT</td>
  </tr>
  <tr>
   <td>eus.rubiconproject.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Apr 3 23:59:59 2025 GMT</td>
  </tr>
  <tr>
   <td>pixel-us-east.rubiconproject.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Apr 3 23:59:59 2025 GMT</td>
  </tr>
  <tr>
   <td>secure-assets.rubiconproject.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Apr 3 23:59:59 2025 GMT</td>
  </tr>
  <tr>
   <td>ct.pinterest.com</td>
   <td>Aug 2 00:00:00 2024 GMT</td>
   <td>Aug 7 23:59:59 2025 GMT</td>
  </tr>
  <tr>
   <td>cms.analytics.yahoo.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Jan 22 23:59:59 2025 GMT</td>
  </tr>
  <tr>
   <td>fastlane.rubiconproject.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Apr 3 23:59:59 2025 GMT</td>
  </tr>
  <tr>
   <td>ka-p.fontawesome.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Jan 27 23:59:59 2025 GMT</td>
  </tr>
  <tr>
   <td>log.pinterest.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Aug 7 23:59:59 2024 GMT</td>
  </tr>
  <tr>
   <td>s.pinimg.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Aug 7 23:59:59 2024 GMT</td>
  </tr>
  <tr>
   <td>assets.pinterest.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Aug 7 23:59:59 2024 GMT</td>
  </tr>
  <tr>
   <td>sync.taboola.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Dec 31 23:59:59 2024 GMT</td>
  </tr>
  <tr>
   <td>prebid-server.rubiconproject.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Apr 3 23:59:59 2025 GMT</td>
  </tr>
  <tr>
   <td>global.ib-ibi.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Apr 2 23:59:59 2025 GMT</td>
  </tr>
  <tr>
   <td>cdn.taboola.com</td>
   <td>Jul 30 00:00:00 2024 GMT</td>
   <td>Dec 31 23:59:59 2024 GMT</td>
  </tr>
</table>

<p>&nbsp;</p>
After the final deadline of August 3rd, 2024 19:30 UTC had passed, I ran an test against a list of the 13,823 publicly availble certificates. I found that 78.5% of them were updated prior to the initial deadline or switched to a using a different certificate. Another 5.3% of the certificates were updated during the extension. However 9.3% of certificates - 1,291 - had failed to get reissued and were revoked.

<table>
  <tr>
   <td colspan="3" align="center"><b>Validity Start Dates for Affected Certificates</b></td>
  </tr>
  <tr>
   <td><strong>Certificate Validity Date</strong></td>
   <td><strong>Certificates</strong></td>
   <td><strong>Percent of Certificates</strong></td>
  </tr>
  <tr>
   <td><p style="text-align: right">Jul 29 2024</p></td>
   <td><p style="text-align: right">120</p></td>
   <td><p style="text-align: right">0.87%</p></td>
  </tr>
  <tr>
   <td><p style="text-align: right">Jul 30 2024</p></td>
   <td><p style="text-align: right">9,670</p></td>
   <td><p style="text-align: right">70.17%</p></td>
  </tr>
  <tr>
   <td><p style="text-align: right">Jul 31 2024</p></td>
   <td><p style="text-align: right">978</p></td>
   <td><p style="text-align: right">7.10%</p></td>
  </tr>
  <tr>
   <td><p style="text-align: right">Aug 1 2024</p></td>
   <td><p style="text-align: right">426</p></td>
   <td><p style="text-align: right">3.09%</p></td>
  </tr>
  <tr>
   <td><p style="text-align: right">Aug 2 2024</p></td>
   <td><p style="text-align: right">254</p></td>
   <td><p style="text-align: right">1.84%</p></td>
  </tr>
  <tr>
   <td><p style="text-align: right">Aug 3 2024</p></td>
   <td><p style="text-align: right">50</p></td>
   <td><p style="text-align: right">0.36%</p></td>
  </tr>
  <tr>
   <td><p style="text-align: right">Using Different Cert</p></td>
   <td><p style="text-align: right">1,034</p></td>
   <td><p style="text-align: right">7.50%</p></td>
  </tr>
  <tr>
   <td><p style="text-align: right">Not Updated</p></td>
   <td><p style="text-align: right">1,291</p></td>
   <td><p style="text-align: right">9.37%</p></td>
  </tr>
</table>
<p>&nbsp;</p>

 A few of these certificates are used by third parties on some popular websites. For example one security service, used across approximately 14k websites, was impacted. I attempted to give them a heads up when I noticed this, but was unsuccessful. Another chat service used on a few popular ecommerce sites was impacted. Fortunately none of these appear to be impacting the loading of these sites - but they may impact some of their functionality once browsers pick up the updated CRL. 

**Monitoring for third party failures**

One of the reasons I went down the path of looking for third parties that might be impacted is because the timing of the event was known. Often we don’t have the liberty of advance notice of impending failures - such as the recent Crowdstrike outages, CDN failures, and other major internet platform incidents. In this case, we had at least 24 hours notice that a massive certificate revocation event would occur (and then a few additional days after the extension).

Using the HTTP Archive data I could see that none of the third parties used at my employer were impacted. However to be absolutely certain I configured a Catchpoint dashboard to monitor for third party availability issues. This dashboard displays the % availability for each third party host, the number of failures for each third party, and some load time metrics. The idea was that if a particular third party we use experienced an issue, we’d be able to identify it quickly.

The dashboard was created by using a line chart broken down by host. Enabling “host data” allowed me to chart some host metrics such as availability and number of failures, as well as exclude first party content. You can see some more details on how to do this in this [blog post from Catchpoint](https://www.catchpoint.com/blog/how-to-filter-out-the-noise-with-zones-and-hosts-a-catchpoint-differentiator). 

<table>
  <tr>
   <td><img src="/assets/img/blog/third-parties-and-certificate-revocations/catchpoint-1.jpg" width="200" alt="Catchpoint dashboard configuration screenshot" loading="lazy"></td>
   <td><img src="/assets/img/blog/third-parties-and-certificate-revocations/catchpoint-2.jpg" width="200" alt="Catchpoint dashboard configuration screenshot"  loading="lazy"></td>
  </tr>
</table>

You may ask why not use real user monitoring (RUM) data for this? RUM can give you timing information on third party requests and additional metrics if the third party sets the `Timing-Allow-Origin` header. It’s great for detecting performance issues related to their party content. However, detecting failures in loading third party resources is not as easy since a failure simply won’t show up in resource timing data.

**Preparing for Third Party Failures**

When a popular third party fails or degrades, sometimes you'll read about it in the news. Far too often organizations handle third party performance or failure risks reactively as well. There’s a few things you can do to prepare ahead of time though.

* Identify third party SPOFs.
    * [WebPageTest’s SPOF feature](https://product.webpagetest.org/tutorials/how-to-simulate-a-single-point-of-failure-spof-using-webpagetest) is great for this!
* Identify third party performance risks. 
    * Test to see what happens when you block or remove their resources. ([WebPageTest](https://andydavies.me/blog/2018/02/19/using-webpagetest-to-measure-the-impact-of-3rd-party-tags/) or [Chrome](https://developer.chrome.com/docs/devtools/network-request-blocking))
* Identify third parties that might impact functionality on your site. 
    * Test to see what happens when you block or remove their resources.
* Monitor third party performance and availability.
    * A combination of RUM for performance and Synthetic measurements for availability can be helpful here.

I’ve been working on a tool that will help identify potential third parties that are worth investigating for performance or single point of failure risks. Hoping to share that with you all very soon!

**Mistakes happen, and we can all learn from them**

On the CA/B [bugzilla](https://bugzilla.mozilla.org/show_bug.cgi?id=1910322) post, DigiCert’s CISO was very communicative about the issues and some of the exceptional circumstances their customers were dealing with. In my opinion it really showed how much the company was doing behind the scenes to balance urgency with risk to the rest of the web. They recently provided a root cause analysis in the comments of the bugzilla thread. It was very informative, but I was disappointed to read that their CISO chose to resign over this incident. 

Incidents are a tremendous opportunity for learning and growth. Resigning over an incident (whether forced or an individual’s decision) sends a very clear message about accountability, blame and punishment. In my opinion, it’s the wrong message to send to the employees of DigiCert and the community at large. When a massive failure or mistake happens within a complex system, it shouldn’t fall on an individual to take responsibility. The fault lies within a system that could have prevented it - and learning from those mistakes is key to not repeating them.

I first learned about the concept of blameless post mortems back in 2011 at a Velocity conference, when Etsy’s former CTO [John Allspaw gave a talk](https://www.oreilly.com/library/view/velocity-conference-2011/9781449311773/oreillyvideos851513.html) about it. He also wrote a fantastic [blog post](https://www.etsy.com/codeascraft/blameless-postmortems) about it on Etsy’s codeascraft blog. A few years later Etsy became one of my clients. And one unfortunate Friday afternoon a change that I was responsible for broke their website. I felt terrible. Etsy's response was to invite me to come present at one of their post mortems - after which I learned that I was the first vendor to ever present at one in person. It was such a fantastic experience, we all learned a lot, it enhanced our working relationship and I wound up continuing to support them for many years. Eventually I decided to join Etsy in 2022. Suffice to say, that post mortem experience had a lasting impact and was a factor in my decision to join the company.

**Conclusion**

While 86k certificates may not sound like a huge amount compared to the scale of the web, the way those certificates were used could have caused a significant amount of disruption to the web. There's no doubt it was frustrating for those that had to rush to update certificates on short notice, but it could have been much worse. There's been a lot of negativity about how DigiCert handled this, but I have a huge amount of empathy for what they have been dealing with. This could have been incredibly disruptive to the web, but it wasn't. Nicely done DigiCert team!
Lastly, I really hope that their post mortems follow a blameless model. Mistakes happen. We can all learn from them and do better.

**HTTP Archive queries**

This section provides some details on how this analysis was performed, including SQL queries and commands for testing the certificates.  Please be warned that some of the SQL queries process a signicant amount of bytes - which can be very expensive to run (particularly the first one).

<details>
  <summary><b>Extract certificate serial numbers from HTTP Archive</b></summary>
   The list of affected certificates that DigiCert provided included the serial numbers, but not a hostname.  In order to identify the hostnames, this query was written to collect serial numbers from all DigiCert certificates found in the HTTP Archive requests table. The approach involved base64 decoding the certificate, converting it to bytes and extracting the substring where the serial number exists. This is a bit of a hack, but it worked!
   <p>&nbsp;</p>
   <b>Warning</b>: this query processes 13 TB of data, which is much higher than the  1 TB free tier.  The results for it have been saving in another BigQuery table for analysis: httparchive.scratchspace.2024_07_01_cert_serials.
  <pre><code>

CREATE TEMPORARY FUNCTION extractCertHex(cert_block STRING)
RETURNS STRING
LANGUAGE js AS '''
   // Extract the Base64 content between the certificate tags
   const base64Match = cert_block.match(/-----BEGIN CERTIFICATE-----\\s*([A-Za-z0-9+/=\\s]+)\\s*-----END CERTIFICATE-----/);
   if (!base64Match) {
       return 'Invalid Certificate Block';
   }
   const base64Content = base64Match[1].replace(/\\s+/g, '');

   // Base64 decode
   function base64ToBytes(base64) {
       const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';
       let bytes = [];
       let buffer = 0, bits = 0;

       for (let i = 0; i < base64.length; i++) {
           if (base64[i] === '=') break;
           const val = chars.indexOf(base64[i]);
           buffer = (buffer << 6) | val;
           bits += 6;
           if (bits >= 8) {
               bits -= 8;
               bytes.push((buffer >> bits) & 0xFF);
           }
       }

       return bytes;
   }

   // Decode the Base64 content
   const cert_der = base64ToBytes(base64Content);

   // Convert DER to hexadecimal
   let cert_hex = '';
   for (let i = 0; i < cert_der.length; i++) {
       const hex = cert_der[i].toString(16).padStart(2, '0');
       cert_hex += hex;
   }

   return cert_hex;
''';

SELECT
 DISTINCT NET.HOST(url) AS host,
 SUBSTR(extractCertHex(JSON_EXTRACT_SCALAR(payload, "$._certificates[0]")),31,32) AS serial_num
FROM 
   `httparchive.requests.2024_07_01_mobile`
WHERE
 JSON_EXTRACT_SCALAR(payload, "$._securityDetails.issuer") LIKE "%DigiCert%"
 AND JSON_EXTRACT_SCALAR(payload, "$._certificates[0]") IS NOT NULL

  </code></pre>
</details>

<details>
  <summary><b>Identify hostnames subject to revocation</b></summary>
   In order to identify the hostnames subject to revocation, I uploaded a copy of the revocation serial numbers to a table: httparchive.scratchspace.digicert_revocation_20240730.  Then I performed a simple INNER JOIN on the output from the previous query to identify hostnames that had a serial number in the revocation list.
  <pre><code>
SELECT DISTINCT 
   host, 
   d.serial AS serial
FROM 
   `httparchive.scratchspace.2024_07_01_cert_serials` ha
INNER JOIN 
   `httparchive.scratchspace.digicert_revocation_20240730` as d
ON ha.serial_num = d.serial
WHERE 
   d.serial IS NOT NULL  
  </code></pre>
</details>

<details>
  <summary><b>Summarize popular third party domains that had hostnames impacted by the revocation</b></summary>
   This query summarized domain names from the requests table by the number of sites loading a resource from it, and the number of hostnames that appeared in DigiCert's list of revoked hostnames.  The previous query is used in the IN() clause of this query.
  <pre><code>
SELECT 
   NET.REG_DOMAIN(url) domain, 
   COUNT(DISTINCT page) sites, 
   COUNT(DISTINCT NET.HOST(url)) hostnames
FROM 
   `httparchive.all.requests` AS r
WHERE 
  date = "2024-07-01"
  AND client = "mobile"
  AND is_root_page = true
  AND NET.HOST(url) IN (
    SELECT DISTINCT host
    FROM `httparchive.scratchspace.2024_07_01_cert_serials` ha
    INNER JOIN `httparchive.scratchspace.digicert_revocation_20240730` as d 
    ON ha.serial_num = d.serial
    WHERE d.serial IS NOT NULL    
  )
GROUP BY 1
ORDER BY 2 DESC

  </code></pre>
</details>

<details>
  <summary><b>Summarize popular third party hostnames that had hostnames impacted by the revocation</b></summary>
   This query is similar to the previous one, but summarizes by hostname instead of domain name.
  <pre><code>
SELECT 
   NET.HOST(url) hostname, 
   COUNT(DISTINCT page) sites
FROM 
   `httparchive.all.requests` AS r
WHERE 
  date = "2024-07-01"
  AND client = "mobile"
  AND is_root_page = true
  AND NET.HOST(url) IN (
    SELECT DISTINCT host
    FROM `httparchive.scratchspace.2024_07_01_cert_serials` ha
    INNER JOIN `httparchive.scratchspace.digicert_revocation_20240730` as d 
    ON ha.serial_num = d.serial
    WHERE d.serial IS NOT NULL    
  )
GROUP BY 1
ORDER BY 2 DESC
  </code></pre>
</details>

<details>
  <summary><b>Bash Script to test certificates for affected hosts</b></summary>
   This script will loop through a list of hostnames and extract the validity dates and serial numbers for each certificate, timing out after 3 seconds if the host is unresponsive.
  <pre><code>
for i in $(cat all_hosts.txt); do 
  timeout 3 echo | openssl s_client -connect "$i:443" 2>/dev/null | 
  openssl x509 -noout -startdate -enddate -serial 2>/dev/null | 
  awk -F= -v host="$i" '
    /^notBefore/ { start = $2 } 
    /^notAfter/  { end = $2 } 
    /^serial/    { serial = $2 } 
    END { 
      print host "," start "," end "," serial 
    }'
done > all_hosts_checked_20240803_1930UTC.csv
  </code></pre>
</details>






