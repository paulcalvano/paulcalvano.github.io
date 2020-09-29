---
layout: post
title: "Growth of the Web in 2020"
date: 2020-09-29 12:30:00 -0400
related_posts:
 
  - _posts/2018-05-15-analyzing-3rd-party-performance-via-http-archive-crux.md
  - _posts/2018-04-26-using-googles-crux-to-compare-your-sites-rum-data-w-competitors.md
  - _posts/2018-05-06-tutorial-using-bigquery-to-analyze-chrome-user-experience-report-data.md

---

For the past 10 years, the [HTTP Archive](https://httparchive.org/) has tracked the evolution of the web by archiving the technical details of desktop and mobile homepages. During its early years, the [Alexa top million](https://www.alexa.com/topsites) dataset (which was publicly available until 2017) was used to source the list of URLs included in the archive and the number of sites tracked increased from 16K to almost 500K as testing capacity increased. To keep the archive current and include new sites, towards the end of 2018 we started using the [Chrome User Experience Report](https://developers.google.com/web/tools/chrome-user-experience-report) as a source of the URLs to track. 

Throughout 2019 the size of the HTTP Archive dataset was mostly constant. However, the sample size has grown quite a bit in 2020 as you can see in the graph below! Additionally, if we combine both desktop and mobile URLs, there was a recent peak of 7.5 million sites!

![HTTP Archive Sample Size Trend](/assets/img/blog/growth-of-the-web-in-2020/image1.jpg){:loading="lazy"}

**How are sites Included in the Chrome User Experience Report** 

The Chrome User Experience Report (CrUX) is sourced from performance data collected from real Chrome users that have opted into syncing their browser history and sharing anonymized usage statistics reporting.It’s essentially real user measurement (RUM) data for Chrome users.

You can read more about CrUX on [Google’s Developer website](https://developers.google.com/web/tools/chrome-user-experience-report), as well as this informative [blog post from Rick Viscomi](https://web.dev/chrome-ux-report/). I’ve also written about it previously [here](https://paulcalvano.com/2018-04-26-using-googles-crux-to-compare-your-sites-rum-data-w-competitors/).

While Google doesn’t publish a definitive list on what it takes to be included in the Chrome User Experience Report dataset, they have [indicated](https://groups.google.com/a/chromium.org/g/chrome-ux-report/c/kVKP6LZpk7U/m/rYbcEXiCCQAJ) that:

* Origins are automatically curated based on real-user Chrome usage
* Websites must meet a traffic threshold to be included.
* Websites must be publicly accessible

Essentially, a website’s inclusion in the Chrome User Experience Report indicates that they’ve reached a certain threshold of activity. According to the CrUX [changelog](https://developers.google.com/web/tools/chrome-user-experience-report/bigquery/changelog), there have been no changes in the methodology.  So it can be inferred that analyzing the number of websites included in this dataset should provide some interesting insights into the month on month growth of the number of websites being visited by real users.

_Note: The Chrome User Experience Report does not contain traffic details, and as such this analysis should not be interpreted as growth of traffic on the internet.  This analysis is specifically about the growth in the number of websites that people are visiting._

**Global Growth of Origins Accessed in 2020**

The graph below illustrates the total number of websites that the Chrome User Experience reported across all form factors during the previous 12 months. There are a few interesting observations we can make:

* In both 2019 and 2020 there were increases in the number of websites at the start of the year.
* There was a linear increase in the number of websites through the first half of 2020. 
* The drop in March and April 2020 is interesting, since that coincides with the start of the global COVID-19 pandemic.

![CrUX Origins Per Month](/assets/img/blog/growth-of-the-web-in-2020/image2.jpg){:loading="lazy"}

There is a similar pattern with the total number of registered domains.  This indicates that much of the growth is new domains and not necessarily subdomains of existing domains. 



![CrUX Registered Domains Per Month](/assets/img/blog/growth-of-the-web-in-2020/image3.jpg){:loading="lazy"}


When we look at the month over month rate of change, you can see that the max change in 2019 was +/- 5%. The number of sites tends to fluctuate month to month. For example, between August and December 2019 there was an 8.6% decrease in sites. However at the start of 2020 there was a 7.5% increase. 

When comparing the number origins between December 2019 and August 2020, the total number of origins increased by 28.9% this year alone! That’s huge!



![CrUX Month/Month Change in Origins and Registered Domains](/assets/img/blog/growth-of-the-web-in-2020/image4.jpg){:loading="lazy"}


**Mobile vs Desktop Growth**

Looking at this by device type, we can see that there are consistently more mobile websites compared to desktop websites.  And over the past year the fluctuations between them have been fairly consistent.  The one exception is between May 2020 and June 2020, where desktop increased by 0.7% and mobile increased by 6.2%.


![CrUX Origins Per Month by Form Factor](/assets/img/blog/growth-of-the-web-in-2020/image5.jpg){:loading="lazy"}


Overall, there are 22.9% more mobile websites in the CrUX dataset compared to desktop.  We know from sources like [statcounter](https://gs.statcounter.com/platform-market-share/desktop-mobile-tablet/worldwide/#monthly-201908-202008) that mobile usage has grown significantly over the years, and consistently surpasses desktop.  But why are mobile users navigating to so many more websites compared to desktop users? 



![Desktop vs Mobile vs Tablet Market Share](/assets/img/blog/growth-of-the-web-in-2020/image6.jpg){:loading="lazy"}


Is there something about the mobile experience (such as social media links, email marketing, etc) that increases the change a user may navigate to an unfamiliar website?  

Or could it be growth in regions where mobile is more dominant? 

**How Has this Varied by Region?**

At the start of 2020, most regions of the world saw an increase in the domain of sites. The exception to this was western Asia. The regions that had the most substantial increase at the start of the year were Northern Europe, North America and South America.

Between May and June there was another large uptick in the number of sites. This appeared to be mostly South-East Asian and Western European countries

The tables below detail the number of sites included in the CrUX dataset during December 2019 as well as January, May and June 2020.  This first table contains the top 10 regions, most of which saw an increase of 15% to 25% during the previous six months!

<table >
<thead><tr><th></th><th colspan=5>Number of Sites Included in CrUX dataset</th></tr></thead>
<thead><tr><th>Sub Region</th><th>Dec 2019</th><th>Jan 2020</th><th>May 2020</th><th>June 2020</th><th>August 2020</th></tr></thead>
 <tbody>
 <tr><td>Northern America</td><td>1,257,159</td><td>1,406,284</td><td>1,676,120</td><td>1,681,454</td><td>1,730,260</td></tr>
 <tr><td>Western Europe</td><td>713,202</td><td>768,644</td><td>874,164</td><td>908,560</td><td>943,891</td></tr>
 <tr><td>Eastern Europe</td><td>640,145</td><td>694,632</td><td>821,024</td><td>913,037</td><td>926,202</td></tr>
 <tr><td>Eastern Asia</td><td>720,926</td><td>740,767</td><td>871,854</td><td>882,322</td><td>901,008</td></tr>
 <tr><td>South America</td><td>486,894</td><td>540,685</td><td>668,604</td><td>726,410</td><td>784,854</td></tr>
 <tr><td>Southern Europe</td><td>506,054</td><td>541,526</td><td>661,416</td><td>710,543</td><td>724,323</td></tr>
 <tr><td>Northern Europe</td><td>453,591</td><td>516,527</td><td>601,459</td><td>638,744</td><td>661,790</td></tr>
 <tr><td>South-Eastern Asia</td><td>473,962</td><td>485,143</td><td>524,249</td><td>584,214</td><td>629,815</td></tr>
 <tr><td>Southern Asia</td><td>403,325</td><td>419,118</td><td>441,328</td><td>462,282</td><td>500,600</td></tr>
 <tr><td>Western Asia</td><td>274,339</td><td>273,610</td><td>327,425</td><td>351,186</td><td>362,340</td></tr>
</tbody></table>


<table >
<thead><tr><th></th><th colspan=5>Percent Change of Sites Included in CrUX dataset</th></tr></thead>
<thead><tr><th>Sub Region</th><th>Dec 2019 - Jan 2020</th><th>May - Jun 2020</th><th>Jun - Aug 2020</th><th>Dec - Aug 2020</th></tr></thead>
<tbody>
 <tr><td>Northern America</td><td>10.60%</td><td>0.32%</td><td>2.82%</td><td>27.34%</td></tr>
 <tr><td>Western Europe</td><td>7.21%</td><td>3.79%</td><td>3.74%</td><td>24.44%</td></tr>
 <tr><td>Eastern Europe</td><td>7.84%</td><td>10.08%</td><td>1.42%</td><td>30.88%</td></tr>
 <tr><td>Eastern Asia</td><td>2.68%</td><td>1.19%</td><td>2.07%</td><td>19.99%</td></tr>
 <tr><td>South America</td><td>9.95%</td><td>7.96%</td><td>7.45%</td><td>37.96%</td></tr>
 <tr><td>Southern Europe</td><td>6.55%</td><td>6.91%</td><td>1.90%</td><td>30.13%</td></tr>
 <tr><td>Northern Europe</td><td>12.18%</td><td>5.84%</td><td>3.48%</td><td>31.46%</td></tr>
 <tr><td>South-Eastern Asia</td><td>2.30%</td><td>10.26%</td><td>7.24%</td><td>24.75%</td></tr>
 <tr><td>Southern Asia</td><td>3.77%</td><td>4.53%</td><td>7.65%</td><td>19.43%</td></tr>
 <tr><td>Western Asia</td><td>-0.27%</td><td>6.77%</td><td>3.08%</td><td>24.29%</td></tr>
</tbody></table>


Looking at the next 10 in the list, we can see significant growth in Central America, Australia as well as West, Southern and South Africa. Overall the regions with the most growth during the 7 month period was Australia and New Zealand, South America, and Central America.


<table >
<thead><tr><th></th><th colspan=5>Number of Sites Included in CrUX dataset</th></tr></thead>
<thead><tr><th>Sub Region</th><th>Dec 2019</th><th>Jan 2020</th><th>May 2020</th><th>June 2020</th><th>August 2020</th></tr></thead>
 <tbody>
 <tr><td>Central America</td><td>155,057</td><td>179,295</td><td>236,255</td><td>242,132</td><td>257,043</td></tr>
 <tr><td>Australia and New Zealand</td><td>124,763</td><td>141,523</td><td>194,212</td><td>196,841</td><td>214,757</td></tr>
 <tr><td>Northern Africa</td><td>68,754</td><td>69,497</td><td>83,312</td><td>88,672</td><td>88,606</td></tr>
 <tr><td>Southern Africa</td><td>50,618</td><td>59,139</td><td>64,978</td><td>66,392</td><td>70,218</td></tr>
 <tr><td>Central Asia</td><td>45,932</td><td>49,192</td><td>57,098</td><td>57,508</td><td>62,112</td></tr>
 <tr><td>Western Africa</td><td>44,692</td><td>49,868</td><td>47,834</td><td>51,257</td><td>50,853</td></tr>
 <tr><td>Caribbean</td><td>33,840</td><td>37,445</td><td>44,090</td><td>45,910</td><td>45,395</td></tr>
 <tr><td>Eastern Africa</td><td>31,010</td><td>34,822</td><td>36,073</td><td>37,388</td><td>38,609</td></tr>
 <tr><td>Middle Africa</td><td>8,873</td><td>9,149</td><td>9,121</td><td>10,057</td><td>10,032</td></tr>
 <tr><td>Melanesia</td><td>2,733</td><td>2,991</td><td>2,580</td><td>2,779</td><td>2,818</td></tr>
</tbody></table>


<table >
<thead><tr><th></th><th colspan=5>Percent Change of Sites Included in CrUX dataset</th></tr></thead>
<thead><tr><th>Sub Region</th><th>Dec 2019 - Jan 2020</th><th>May - Jun 2020</th><th>Jun - Aug 2020</th><th>Dec - Aug 2020</th></tr></thead>
<tbody>
 <tr><td>Central America</td><td>13.52%</td><td>2.43%</td><td>5.80%</td><td>39.68%</td></tr>
 <tr><td>Australia and New Zealand</td><td>11.84%</td><td>1.34%</td><td>8.34%</td><td>41.91%</td></tr>
 <tr><td>Northern Africa</td><td>1.07%</td><td>6.04%</td><td>-0.07%</td><td>22.40%</td></tr>
 <tr><td>Southern Africa</td><td>14.41%</td><td>2.13%</td><td>5.45%</td><td>27.91%</td></tr>
 <tr><td>Central Asia</td><td>6.63%</td><td>0.71%</td><td>7.41%</td><td>26.05%</td></tr>
 <tr><td>Western Africa</td><td>10.38%</td><td>6.68%</td><td>-0.79%</td><td>12.12%</td></tr>
 <tr><td>Caribbean</td><td>9.63%</td><td>3.96%</td><td>-1.13%</td><td>25.45%</td></tr>
 <tr><td>Eastern Africa</td><td>10.95%</td><td>3.52%</td><td>3.16%</td><td>19.68%</td></tr>
 <tr><td>Middle Africa</td><td>3.02%</td><td>9.31%</td><td>-0.25%</td><td>11.55%</td></tr>
 <tr><td>Melanesia</td><td>8.63%</td><td>7.16%</td><td>1.38%</td><td>3.02%</td></tr>
</tbody></table>


Many of the regions that had an increase in sites visited (based on CrUX data), also have a high percentage of mobile visitors compared to the global population (based on statcounter).  So while it’s difficult to say for certain, it’s entirely possible that location is a large factor in the gap between Desktop and Mobile.

**Analyzing by Top Level Domain**

The .com top level domain accounts for 43.7% of all websites tracked in the Chrome User Experience report. The next largest top level domain is .org, which consists of 3.7% of all sites. Overall there were 4111 TLDs in the dataset, and the top 20 of them represented 75% of all websites.


![Distribution of Websites by TLD - Chrome User Experience Report](/assets/img/blog/growth-of-the-web-in-2020/image7.jpg){:loading="lazy"}


Most of these top level domains experienced a > 20% growth in active websites since December 2019, with the exception of .info and .net. The domains with the largest percentage growth were co.uk, com.au and de.  


![% Growth in Websites by TLD - December 2019 - August 2020](/assets/img/blog/growth-of-the-web-in-2020/image8.jpg){:loading="lazy"}


If we look at the month to month growth trends for these TLDs, we can make a few interesting observations:



* There was a significant drop across all TLDs in March 2020.  
* The largest percentage drop was for .it domains in March 2020, although that rebounded with increases in April, May and June.
* In February 2020, there was a 23.9% increase in .edu domains receiving traffic. 
* In May 2020, more than a dozen popular TLDs saw a double-digit increase in the number of sites. 
* In August 2020 there was a 10.4% increase in edu domains



![Month/Month % Growth of Websites in CrUX](/assets/img/blog/growth-of-the-web-in-2020/image9.jpg){:loading="lazy"}


**Conclusion**

The web is constantly growing and evolving, and clearly it’s rate of growth can vary quite a bit. During this analysis we explored a public dataset that Google provides to show how the web has grown during 2020, and which regions are growing the most.  While this doesn’t speak to the traffic levels experienced in these locations, the number of websites can be used as a proxy for understanding usage of the web. As this analysis shows, 2020 has been a year of substantial global growth for the web.

If you are interested in seeing some of the SQL queries and raw data used in this analysis, I’ve created a post with all the details in the [HTTP Archive discussion forums](https://discuss.httparchive.org/t/growth-of-the-web-in-2020/2029). You can also see all the data used for these graphs in this [Google Sheet](https://docs.google.com/spreadsheets/d/1eGfT0dBslpSl8Xl6ey7a_fNT0Cj5tTfJzX6gpwpXDBE/edit?usp=sharing).

