---
# Featured tags need to have either the `list` or `grid` layout (PRO only).
layout: about

# The title of the tag's page.
title: Speaking

# The name of the tag, used in a post's front matter (e.g. tags: [<slug>]).
slug: example

# (Optional) Write a short (~150 characters) description of this featured tag.
description: >

# (Optional) You can disable grouping posts by date.
# no_groups: true
---


Here's a few of the talks that I've given over the past few years. I've included the slides for each presentation since I often update the stats before each event, and sometimes add some stats specific to the locale I'm presenting in.

**Font Performance**

Only 10 years ago, custom web fonts were a niche feature, but today they are used by 83% of websites. While it’s easy to add and use web fonts, there are many ways that they can negatively impact web performance and user experience.

During this talk Paul will provide an overview of custom web fonts and some web performance techniques you can use to optimize them. We’ll look at examples from the HTTP Archive and explore some free tools that can be used to analyze and optimize fonts on your site.

<iframe loading="lazy" src="https://www.slideshare.net/slideshow/embed_code/key/c9l7x8cWKAfiCE" width="597" height="486" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> 

Presented at: 
- 2024-April: [NYC WebPerf Meetup](https://www.meetup.com/web-performance-ny/events/299395150/) (New York, NY) [Slides](https://www.slideshare.net/slideshow/font-performance-nyc-webperf-meetup-april-24-1abf/267363735)

**Performance Mistakes - an HTTP Archive Deep Dive**

Web performance is a complicated topic, but over the years it’s become easier to articulate thanks to incredible advancements in performance features, their adoption in the browser ecosystem and tools that test and give insight into which techniques might speed up your site.

However, all too often a feature is implemented incorrectly, resulting in a lost opportunity for performance improvement. During this talk I explored a few common web performance techniques - some that you are likely already familiar with. Looking at the HTTP Archive we found some examples of sites that are using them incorrectly, as well as the impact and potential benefits of fixing them.

<iframe loading="lazy" src="https://www.slideshare.net/slideshow/embed_code/key/udVefOIzNvOst3?startSlide=1" width="597" height="486" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px;max-width: 100%;" allowfullscreen></iframe>

Presented at: 
- 2022-June: [Lazy Load Conference](https://webdirections.org/lazyload/speakers/paul-calvano.php) (Virtual) [Slides](https://www.slideshare.net/PaulCalvano/lazy-load-22-performance-mistakes-an-http-archive-deep-dive)
- 2022-Sept: [NYC WebPerf Meetup](https://www.meetup.com/web-performance-ny/events/286338923/) (New York, NY) [Slides](https://www.slideshare.net/PaulCalvano/ny-webperf-sept-22-performance-mistakes-an-http-archive-deep-dive)

**WebP + AVIF + more: Image format adoption stats for 2021**

2020 was the year AVIF went stable in Chrome and WebP secured footing in all major browsers. Did this have any impact on adoption of these formats across the web?  During this session, I presented an HTTP Archive analysis on how these image formats are being used in 2021    

<iframe loading="lazy" width="560" height="315" src="https://www.youtube.com/embed/tz5bpAQY43k" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Presented at:
- 2021-September: [Toronto WebPerf Meetup - ImageReady 2.0](https://www.meetup.com/Toronto-Web-Performance-Group/events/280848011/) (Virtual)    [Slides](https://docs.google.com/presentation/d/1VS5QjNR6lh2y9jL5xaeainQ2cTAWyy7QiEjDMh4hNQA/)
    
**Measuring the Adoption of Web Performance Techniques**

Performance optimization is a cyclical process. We are constantly learning new ways to optimize, while simultaneously adopting new technologies and techniques that negatively impact performance. The HTTP Archive provides a great historical record of the technical side of the web, with almost 10 years of history and an ever growing dataset of sites.

During this session I provided a brief overview of the HTTP Archive and then dove into some insights into the adoption of common web performance techniques and some of their measurable impacts.

<iframe loading="lazy" src="//www.slideshare.net/slideshow/embed_code/key/i3sEaeA2xHdglT" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> 


Presented at:
- 2020-February: [NYC Web Performance Meetup](https://www.slideshare.net/PaulCalvano/nyc-webperf-meetup-feb-2020-measuring-the-adoption-of-web-performance-techniques) (New York, NY)
- 2019-September: [FITC Web Unleashed](https://www.slideshare.net/PaulCalvano/web-unleashed-19-measuring-the-adoptions-of-web-performance-techniques-172106797) (Toronto, Canada)

**Common Traits of High Performing Websites**

Have you ever wondered how often some web performance best practices are implemented on the web? How much faster are sites that implement them, and what are some key things to avoid? During this session, well review data from the HTTP Archive, CrUX and mPulse to help answer some of these questions.

<iframe loading="lazy" src="//www.slideshare.net/slideshow/embed_code/key/E1j066R2PLETks" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> 


Presented at:
- 2019-February: [CMG Impact](https://slideslive.com/38913525/a-quickstart-guide-to-web-performance-analysis?ref=speaker-14940-latest) (Seattle, WA)
- 2019-January: [BairesWeb Meetup](https://www.slideshare.net/PaulCalvano/common-traits-of-high-performing-websites-bairesweb-argentina) (Buenos Aires, Argentina)
- 2018-November: [WebPerfDays](https://www.slideshare.net/PaulCalvano/common-traits-of-high-performing-websites-webperfdays-amsterdam-07nov2018) (Amsterdam, Netherlands)

**Tracking the Performance of the Web With the HTTP Archive**

Have you ever thought about how your sites performance compares to the web as a whole? Or maybe youre curious how popular a particular web feature is. How much is too much JavaScript? The HTTP Archive has been keeping track of how the web is built since 2010. It enables you to find answers to questions about the state of the web past and present.

Paul Calvano explores how the HTTP Archive works, how people are using this dataset, and some ways that Akamai has leveraged data within the HTTP Archive to help its customers.

<iframe loading="lazy" src="//www.slideshare.net/slideshow/embed_code/key/BGEWAgPmUBd9a8" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> 

Presented at:

- 2019-February: [CMG Impact](https://slideslive.com/38913944/leveraging-http-archive-for-better-web-comparatives-and-analytics) (Seattle, WA)
- 2018-June: [Fluent Conference](https://www.slideshare.net/PaulCalvano/fluent-2018-tracking-performance-of-the-web-with-http-archive-102455732) (San Jose, CA)


**Crushing it - Compression on the Web**

Less is more. Your customers want more information, more functionality and more engagement all in less time. Most websites use some sort of compression to help speed up their sites and reduce costs. With the average size of web pages growing over 3MB, gzip is no longer enough. In this session, the basics of modern compression are just the beginning, we will dig deeper into best practices and techniques that you can immediately apply to make the most out of an array of compression options, including multiple types of text compression, image and video compression, and even JavaScript compression.

How you configure compression matters to your customers experience and your bottom line. Dont just rely on a basic familiarity of compression. Maximize your compression knowledge and minimize your content volume.

<div id="presentation-embed-38913781"></div>
<script src='https://slideslive.com/embed_presentation.js'></script>
<script>
    embed = new SlidesLiveEmbed('presentation-embed-38913781', {
        presentationId: '38913781',
        autoPlay: false, // change to true to autoplay the embedded presentation
        verticalEnabled: true
    });
</script>

Presented at:
- 2019-February: [CMG Impact](https://slideslive.com/38913781/crushing-it-compression-on-the-web) (Seattle, WA)

**Real User Measurement Insights**

Many websites use real user measurement (RUM) data to analyze their performance, as well as to validate the impact of optimizations. During this session, well discuss how RUM is used and then explore some of the fascinating insights into the web that we can learn from it.

<iframe loading="lazy" src="//www.slideshare.net/slideshow/embed_code/key/z0iKvoNIZh08Sl" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe>


Presented at:
- 2018-November: [London WebPerf Meetup](https://www.slideshare.net/PaulCalvano/real-user-measurement-insights-london-webperf-2018nov06) (London, England)
- 2018-August: [NYC Web Performance Meetup](https://www.slideshare.net/PaulCalvano/real-user-measurement-insights-nywebperf-2018aug09) (New York, NY)


**State of the Web - Web Transparency**

In this episode of The State of the Web, Rick speaks with Paul Calvano, a Senior Web Performance Architect at Akamai Technologies, a company that delivers 20-30% of the worlds web traffic. 


<iframe loading="lazy" width="560" height="315" src="https://www.youtube.com/embed/hqTtkdNwYwk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


**3 Tips to Increasing Mobile App Performance**

Sr. Web Performance Architect, Paul Calvano and Sr. Strategist Eric Mingorance share their experience and insights to help you make your Mobile Apps faster. Theyll demo techniques to help identify problems and youll learn how to solve them with Akamai.

This session covers these 3 important areas:
- How to capitalize on the benefits of serving API traffic through Akamai
- Analyzing Mobile Applications for performance issues
- Advanced Offload/Optimization Techniques for Mobile Applications

[![](/assets/img/blog/speaking/mobile-app-perf.jpg){:loading="lazy"}](http://videop-community.akamai.com/video/5c325d1b5d00e_11727282.mp4)

Presented at:
- 2016-July: [Akamai Edge Conference](http://videop-community.akamai.com/video/5c325d1b5d00e_11727282.mp4) (Las Vegas, NV)

**Advanced Caching Concepts**

The caching ecosystem has evolved over the years  what, where, and how long you cache your web assets are now important considerations for anyone doing business on the internet. Browser cache, html5 application cache, sophisticated reverse proxies like Varnish, and the evolution of CDNs have all elevated caching as the single most effective tool for creating high performing and scalable web applications.

Using live demos, we will dive into some advance caching concepts that will enable you to squeeze the most benefits from this caching ecosystem.
<p></p>
<iframe loading="lazy" src="//www.slideshare.net/slideshow/embed_code/key/9jRfREH8SF8h3r" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe>

Presented at: 
- 2015-October: [Velocity Conference](https://www.slideshare.net/RakeshChaudhary4/advanced-caching-concepts-velocity-ny-2015) (New York, NY)
