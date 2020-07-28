---
title: "San Certificates: How Many Alt-Names Are Too Many?"
date: 2020-02-17T18:59:25+00:00
author: Paul Calvano
layout: post
---

According to the HTTP Archive, 84% of HTTPS certificates are using the Subject Alternate Name (SAN) extension, which allows multiple hostnames to be protected by a single certificate. The largest certificate I found in the HTTP Archive contained a whopping 1275 alt-names! During this post we’ll explore why this is a web performance problem, and how you can determine what a reasonable limit would be for your certificates.

**Overview**

The Subject Alternate Name extension is detailed in [RFC5280 Section 4.2.1.6](https://tools.ietf.org/html/rfc5280#section-4.2.1.6) and the range of alt-names is defined as 1..MAX. While the value of MAX is not defined, certificate authorities can and often do impose limits on the number of alt-names in a certificate. For example, [LetsEncrypt](https://letsencrypt.org/docs/rate-limits/), [DigiCert](https://www.websecurity.symantec.com/security-topics/san-ssl-certificates) and [GoDaddy](https://www.godaddy.com/web-security/multi-domain-san-ssl-certificate) all limit SAN certificates to 100 hostnames. [Comodo's](https://comodosslstore.com/comodo-mdc-ssl.aspx) limit is 2000.

However these stated limits don’t always match what we can measure in the wild...

**Distribution of Alt-Name Counts**

The majority of certificates have just 2 alt-names. This is usually because they contain both a top level domain (ie, example.com) and either a wildcard (*.example.com) or a fully qualified domain (www.example.com). However, 408,433 domains have at least 20 alt-names, and there are 47,171 domains with over 100 alt-names! That’s a lot of alt-names!

![](/assets/img/blog/san-certificates-how-many-alt-names-are-too-many/1.jpg) 

**Do Clients Limit Certificate Sizes?**

The appropriately named website, [badssl.com](https://badssl.com/), contains links to numerous poorly configured certificates for testing purposes. They even have a [certificate with 10,000 alt-names](https://10000-sans.badssl.com/)!

Attempting to load their 10k SAN cert in Chrome fails with an SSL Protocol Error, likely due to the excessive size.

![438x200](/assets/img/blog/san-certificates-how-many-alt-names-are-too-many/2.png)

When attempting to load the same certificate from curl, we can see that the request fails due to the message size. The [certificate with 1000 alt-names](https://1000-sans.badssl.com/) is able to load successfully though. So what is the hard limit?

```
curl https://10000-sans.badssl.com/
curl: (35) error:14006098:SSL routines:CONNECT_CR_CERT:excessive message size
```

I always thought that certificate sizes would be limited to 16KB, since [the maximum TLS record size is 16KB](https://hpbn.co/transport-layer-security-tls/). However the largest certificates are more than 16KB, and multiple TLS records are used for them. This got me curious about the maximum certificate size allowed. After some digging around in the Chromium source, I found that Chrome will [reject a certificate larger than 64KB](https://cs.chromium.org/chromium/src/net/cert/internal/cert_issuer_source_aia.cc?l=20). But please, for the sake of your web performance, don’t do that!

**Certificate Size and Web Performance**

To understand why certificate size can be a performance concern, let's revisit what happens when a client attempts to establish a TLS connection.

After a TCP connection is established, the client sends a ClientHello packet to initiate the TLS connection. The server then responds with a ServerHello, sends the certificate and the list of alt-names. If the certificate is too large, then you have to span multiple packets. The exact amount will vary based on the congestion window size, but in general new connections start with the initial congestion window. @igrigorik explains this in detail in the [High Performance Browser Networking chapter on TLS](https://hpbn.co/transport-layer-security-tls/):

![624x460](/assets/img/blog/san-certificates-how-many-alt-names-are-too-many/3.png)

A large certificate can be problematic for a few reasons:

* The TLS negotiation will need to span numerous packets, and multiple round trips
* All packets need to be received and reassembled before any HTTP request can be made, so this is head of line blocking
* Packet loss and latency can add additional delays

Optimizing your alt-name list is just one of many things you can do to tweak TLS performance, and you can find more details in the [TLS Chapter of High Performance Browser Networking](https://hpbn.co/transport-layer-security-tls/), as well as the ["Is TLS Fast" website](https://istlsfastyet.com/).

The graph below was taken from a WebPageTest measurement for a site that contained 1000 alt-names. In this example, the TLS time actually took longer than the download time for the entire base HTML.

![624x239](/assets/img/blog/san-certificates-how-many-alt-names-are-too-many/4.png)

To get a better idea of what is going on behind the scenes, let's examine a Wireshark capture for this measurement. Packets 13, 17 and 19 show the 114ms TCP connection. Packets 20 - 68 is the TLS negotiation, with the ServerHello taking up the largest amount of it. We’re still limited by a 16KB TLS record size, so there are 2 TLS records that need to be assembled to complete the TLS negotiation. A total of 11 packets!

![624x268](/assets/img/blog/san-certificates-how-many-alt-names-are-too-many/5.jpg )

**Examining Certificate Sizes**

In Chrome DevTools you can examine the certificate details in the Security tab. For example, we can see that ugos.ugm.ac.id contains 1342 alt-names. There were 1275 hostnames during the last HTTP Archive run, so 67 alt-names were added since then!

![624x452](/assets/img/blog/san-certificates-how-many-alt-names-are-too-many/6.png)

You can use the openssl command to request a certificate and count the number of bytes in the certificate chain. In the example below, you can see that 46KB of data was transferred just to establish a secure connection!

```
openssl s_client -showcerts -connect gizikesehatan.ugm.ac.id:443 2>&1 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' | wc -c

47051
```

You can also use the following openssl command to count the number of bytes in the alt-name list. In this example, 36KB of this certificate was just alt-names.

```
openssl s_client -showcerts -connect gizikesehatan.ugm.ac.id:443 2>&1 | openssl x509 -noout -text | grep DNS | wc -c

37297
```

**Alt-Name Distribution of Popular Certificate Authorities**

In the [2019 Web Almanac](https://almanac.httparchive.org/en/2019/), certificate size was mentioned in [the chapter on CDNs](https://almanac.httparchive.org/en/2019/cdn#rtt-and-tls-management). The reason for this is because some content delivery networks utilize large shared certificates.  

> TLS handshake performance is impacted by a number of factors. These include RTT, TLS record size, and TLS certificate size. While RTT has the biggest impact on the TLS handshake, the second largest driver for TLS performance is the TLS certificate size.
> ...
> 
> Many CDNs, however, depend on shared TLS certificates and will list many customers in the SAN of a certificate. This is often necessary because of the scarcity of IPv4 addresses.
> ...
> 
> Most CDNs balance the need for shared certificates and performance. Most cap the number of SANs between 100 and 150. This limit often derives from the certificate providers.


I found the earlier example particularly intriguing since 1342 alt-names is much larger than what DigiCert’s website lists as their maximum. The graph below illustrates the maximum alt-name sizes observed for the top 15 certificate authorities. Sectigo and cPanel certificates are signed by Comodo, which explains the 1000 maximum. However Digicert appears to serve the largest of the large certificates.

![624x309](/assets/img/blog/san-certificates-how-many-alt-names-are-too-many/7.png)

The average certificate sizes are significantly smaller, which is in line with the earlier observation that the majority of certificates have 2 alt-names.

![624x304](/assets/img/blog/san-certificates-how-many-alt-names-are-too-many/8.png)

If we examine the distribution of certificate sizes, it seems that the certificate authorities used for the larger alt-name lists tend to be GTS, Comodo and DigiCert.

![624x324](/assets/img/blog/san-certificates-how-many-alt-names-are-too-many/9.png)

**So, What Is a Reasonable Alt-Name Size?**

While it’s easy to think of limits as the number of hosts, the reality is that the bytes matter more than the number of hosts. In High Performance Browser Networking,

> Assuming a common 1,500-byte starting MTU, this leaves 1,420 bytes for a TLS record delivered over IPv4, and 1,400 bytes for IPv6. To be future-proof, use the IPv6 size, which leaves us with 1,400 bytes for each TLS record, and adjust as needed if your MTU is lower.

In the Wireshark example I shared earlier, the ServerHello packet was 1488 bytes. After the rest of the certificate details were sent, only 10 of the 1000 alt-names fit into the first packet. The second packet had a larger length and sent an additional 110 alt-names. Given that the other aspects of the certificate size will vary, it’s difficult to provide guidance on exactly how many alt-names should be included in your certificate.

I’d recommend capturing a packet capture of the negotiation and examining it as I did above. Beyond that, keeping the number of alt-names to a minimum will reduce the number of round trips and result in a faster TLS negotiation.

**HTTP Archive Queries**

------

I used the following query to extract some of the certificate details and count the alt-names. Please note that this query processes 2.84 TB of data. I’ve saved the results in `httparchive.scratchspace.certificateAltNames_2020_01_01_mobile` if you would like to examine the results.

```
CREATE TEMPORARY FUNCTION countAltNames(payload STRING)
RETURNS INT64
LANGUAGE js AS """
var altNames = JSON.parse(payload);
var altNameCount = 0
if (altNames) {
    var altNameCount = altNames.length;
}

try {
    return altNameCount;
} catch (e) {
    return 0;
}

""";

SELECT
    DISTINCT page,
    NET.HOST(url) AS hostname,
    countAltNames(JSON_EXTRACT(payload, '$._securityDetails.sanList')) AS altNameCount,
    JSON_EXTRACT_SCALAR(payload, "$._ip_addr") AS ip_address,
    JSON_EXTRACT_SCALAR(payload, "$._securityDetails.subjectName") AS subjectName,
    JSON_EXTRACT_SCALAR(payload, "$._securityDetails.protocol") AS protocol,
    JSON_EXTRACT_SCALAR(payload, "$._securityDetails.cipher") AS cipher,
    JSON_EXTRACT_SCALAR(payload, "$._securityDetails.issuer") AS issuer,
    JSON_EXTRACT(payload, '$._securityDetails.sanList') AS altNameList
FROM 
    `httparchive.requests.2020_01_01_mobile`
WHERE
    url LIKE "https://%"
    AND JSON_EXTRACT_SCALAR(payload, "$._securityDetails.subjectName") IS NOT NULL
```

The following query summarizes the alt-name sizes per hostname -

```
SELECT
    altNameCount,
    COUNT(DISTINCT hostname) AS freq
FROM 
    `httparchive.scratchspace.certificateAltNames_2020_01_01_mobile`
GROUP BY altNameCount
ORDER BY altNameCount ASC
```

And this query summarizes the alt-name sizes per certificate authority:

```
SELECT
    issuer,
    COUNT(DISTINCT hostname) AS hostnames,
    MIN(altNameCount) AS minAltNameCount,
    ROUND(AVG(altNameCount)) AS avgAltNameCount,
    MAX(altNameCount) AS maxAltNameCount,
    SUM(IF(altNameCount <= 10,1,0)) AS LessThanOrEqualTo10,
    SUM(IF(altNameCount > 10 AND altNameCount <=50,1,0)) AS Between11_and_50,
    SUM(IF(altNameCount > 50 AND altNameCount <=100,1,0)) AS Between51_and_100,
    SUM(IF(altNameCount > 100,1,0)) AS GreaterThan100
FROM 
    `httparchive.scratchspace.certificateAltNames_2020_01_01_mobile`
GROUP BY issuer
ORDER BY hostnames DESC
```

_Originally published at <https://discuss.httparchive.org/t/san-certificates-how-many-alt-names-are-too-many/1867/>_