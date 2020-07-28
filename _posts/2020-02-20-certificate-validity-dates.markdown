---
layout: post
title: "Certificate Validity Dates"
date: 2020-02-20 00:00:00 -0500
img: 
---
Back in 2017 the maximum validity lifetime for an HTTPS certificate was [set to 825 days](https://cabforum.org/2017/03/17/ballot-193-825-day-certificate-lifetimes/), a decision that was widely supported by both browsers and certificate authorities. However, since then there have been multiple unsuccessful attempts at reducing the maximum lifetime to one year. [Scott Helme](https://twitter.com/Scott_Helme) has [written about this previously](https://scotthelme.co.uk/ballot-sc22-reduce-certificate-lifetimes/), and his blog post noted that browser vendors unanimously supported this while some certificate authorities objected to it.

Information is still trickling in, but it seems that Safari is planning to enforce a max validity lifetime of 398 days effective September 1st, 2020.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Today&#39;s big news: One year max public TLS certs are coming, starting 1 Sept 2020, if you want to be trusted in Safari.</p>&mdash; Dean Coclin (@chosensecurity) <a href="https://twitter.com/chosensecurity/status/1230253348236013570?ref_src=twsrc%5Etfw">February 19, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Let’s take a look at the state of certificate validity ranges today, so we can track how this evolves over the next few months. The data for this comes from the [HTTP Archive](https://httparchive.org), which is an open source project that tracks how the web is built. 

**Certificate Validity Dates in the Wild**

The HTTP Archive requests table contains certificate details for every HTTPS request that was served to the 5.3 million websites tracked. The details are in the `$._securityDetails` and the data contains information on 4,397,690 unique hosts. Out of all of these, 136,007 hostnames served a validity date greater than 825 days! That’s 3.09% of all requests!

Certificate validity dates are widely distributed, but there seems to be a few popular ranges. THe most common validity date is 90 days, likely to the popularity of LetsEncrypt Overall 55% of certificates have a validity date of less than 364 days, which I’ve highlighted in green below. An additional 20% of certificates have a validity between 365 and 398 days, which will meet Safari’s requirements. The remaining 25% have a validity range of more than 398 days.

![624x181](/assets/img/blog/certificate-validity-dates/1.jpg)

Looking at this by certificate authority is also quite interesting. The graph below shows the top 15 certificate authorities. LetsEncrypt accounts for 38.4% of all certificates -

![624x227](/assets/img/blog/certificate-validity-dates/2.jpg)

When we look at certificate validity date ranges for these certificate authorities, we can see that there’s a mix. Certificates issued by LetsEncrypt, Cloudflare, cPanel and Amazon meet the 398 day requirement already. However, Sectigo, GoDaddy, DigiCert, Comodo and RapidSSL have a very large percentage of certificates that exceed 398 days. 

![624x284](/assets/img/blog/certificate-validity-dates/3.jpg)

[DigiCert released a public statement](https://www.digicert.com/position-on-1-year-certificates/) yesterday, which confirms that existing certificates with a validity range >398 days will continue to be trusted by Safari, but that certificates issued after August 30th won’t be able to exceed 398 days.

> For your website to be trusted by Safari, you will no longer be able to issue publicly trusted TLS certificates with validities longer than 398 days after Aug. 30, 2020. Any certificates issued before Sept. 1, 2020 will still be valid, regardless of the validity period (up to 825 days). Certificates that are not publicly trusted can still be recognized, up to a maximum validity of 825 days. 

I’m interested in seeing how this will evolve in the coming months, especially once there are more formal announcements about this. If you would like to see the queries used for this analysis, I've detailed them in this [HTTP Archive discussion forum post](https://discuss.httparchive.org/t/certificate-validity-dates/1874).

Many thanks to Scott Helme for reviewing this.

