---
layout: post
title: "Internet Explorer's Decline in Usage in 2021"
date: 2022-01-31 10:00:00 -0400
related_posts:
 
  - _posts/2017-06-27-performance-debugging-leads-to-discovery-of-a-bizarre-browser-bug.md
  - _posts/2017-11-29-measuring-the-performance-of-firefox-quantum-with-rum.md
  - _posts/2018-07-02-impact-of-page-weight-on-load-time.md

---

In May 2021 Microsoft [announced](https://blogs.windows.com/windowsexperience/2021/05/19/the-future-of-internet-explorer-on-windows-10-is-in-microsoft-edge/) that it would be officially retiring Internet Explorer in favor of the Chromium based [Microsoft Edge](https://www.microsoft.com/en-us/edge). Usage for the legacy browser had been very low over the past few years, although many websites have still maintained polyfills for the older browser. In fact a number of my clients have recently told me that supporting IE 11  is required by their business, and is still a consideration when adding new features to their sites. I’m sure that the official retirement of Internet Explorer will help numerous organizations embrace modern web features and move on from some expensive [polyfills](https://developer.mozilla.org/en-US/docs/Glossary/Polyfill). The official retirement date is June 15, 2022.

A few years ago [Philip Walton](https://twitter.com/philwalton) wrote an excellent blog post about [loading polyfills only when needed](https://philipwalton.com/articles/loading-polyfills-only-when-needed/). His strategy focused on optimizing the experience for users on modern browsers, and loading polyfills based on browser support for required features (or lack thereof). One thing I really like about this approach is that he recommends creating separate bundles for modern browsers. This avoids unnecessary CPU load on modern browsers, since they won’t have to [parse, compile and evaluate](https://v8.dev/blog/cost-of-javascript-2019) the polyfill JavaScript.

More recently, [Alan Davalos](https://twitter.com/AlanGDavalos) published a blog post where he made the argument that the [baseline for web development in 2022 ](https://engineering.linecorp.com/en/blog/the-baseline-for-web-development-in-2022/)should change as a result of this. He provided numerous examples of websites that have officially dropped support for the legacy browser in 2021. 

This made me curious about the current traffic levels for Internet Explorer, and how that has changed over the past year. Looking at [Akamai’s mPulse](https://www.akamai.com/products/mpulse-real-user-monitoring) data, which measures real user performance data for users across all browsers, I can see a steady decline since March 2021. Most of the Internet Explorer traffic was from the IE 11 browser, with earlier browsers being a small fraction (&lt; 0.01% of pages). 


![Internet Explorer Traffic - Monthly](/assets/img/blog/internet-explorer-decline-in-2021/internet_explorer_monthly_pct.jpg){:loading="lazy"}

Looking at this data daily, you can see that the decline in Internet Explorer usage was linear throughout the year. It also seems that the decline accelerated after the announcement in May 2021. Usage dropped even further in November 2021.

The zig-zag pattern in the daily traffic indicates that the majority of users utilizing IE 11 are doing so during the Monday-Friday work week, with traffic dropping by two thirds on the weekends. If business users are the primary source of Internet Explorer usage, then they could be at risk of 0-day exploits once official support ends.


![Internet Explorer Traffic - Daily](/assets/img/blog/internet-explorer-decline-in-2021/internet_explorer_daily_pct.jpg){:loading="lazy"}

I thought it would be interesting to see which countries had the highest percentage of Internet Explorer traffic during January 2022.  Sure enough this varied from country to country. For example, in the United States 0.97% of pages were loaded from an Internet Explorer browser. In Great Britain, it only accounted for 0.29% of pages.  

Some countries had a noticeably higher percentage of Internet Explorer traffic - for example South Korea (2.36%), China (1.76%), Hong Kong (1.78%). And then there are some countries with a shockingly high percentage of Internet Explorer traffic: Haiti (26.15%), Belize (9.12%), Jamaica (7.13%), Cambodia (6.8%) and more. The graph below shows a summary Internet Explorer usage per country. You can also view an interactive version of it [here](https://public.tableau.com/app/profile/paul.calvano8666/viz/InternetExplorerUsagebyCountry-Jan2022/Sheet1).

![Internet Explorer Usage by Country - Jan 2022](/assets/img/blog/internet-explorer-decline-in-2021/internet_explorer_pct_by_country_jan2022.jpg){:loading="lazy"}


Internet Explorer is going away, and that is ultimately a good thing for the web. As support officially ends, website owners should consider removing expensive polyfills for the browser and add support for technologies that are supported on modern browsers. However this won’t happen by itself, and sites that have been bundling polyfills into a large monolithic bundle will need to put in the effort to analyze which ones are still needed. Fortunately there are some great resources on bundle analysis, such as [Sia Karamalegos’](https://twitter.com/thegreengreek) guide to [Lighthouse Treemap](https://sia.codes/posts/lighthouse-treemap/) and [Nolan Lawson’s](https://nolanlawson.com/about/) guide to [bundle analysis tools](https://nolanlawson.com/2021/02/23/javascript-performance-beyond-bundle-size/).

Organizations should also look to remove the legacy browsers on their employees' machines to avoid security risks down the line. Microsoft provides guides and resources to remove legacy Internet Explorer browsers from organizations, which are available [here](https://www.microsoft.com/en-us/download/details.aspx?id=102119).
