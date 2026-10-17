---
layout: post
title: "Investigating a Curious Case of Font Format Fallbacks"
date: 2026-10-17 00:00:00 -0400
related_posts:
  - _posts/2024-02-17-identifying-font-subsetting-opportunities.markdown
  - _posts/2017-07-25-performance-and-usage-implications-of-custom-fonts.md

---

Every month the [NY WebPerf Meetup](https://www.meetup.com/web-performance-ny/) runs a session called Meet4Speed, where [Sergey Chernyshev](https://www.linkedin.com/in/sergeychernyshev) and I meet with members of the developer community and talk about ways to optimize each others sites. It's an incredibly fun and interactive meetup. There is no presenter in this format. Instead we sit around a large conference room table, share a few pizzas (which also serves as an effective way to lure me into NYC), talk shop and learn from each other. 

![ Meetup listing for the in-person "Meet for SPEED" session](/assets/img/blog/investigating-a-curious-case-of-font-format-fallbacks/ny-webperf-meet4speed-meetup.jpg)

The websites we analyze are picked from attendees. I never know what we're going to optimize coming into this session. Sometimes attendees ask to analyze sites they work for or built, but occasionally we're analyzing sites that no-one has a connection to using browser DevTools, WebPageTest, etc.  Hilariously a dance troupe once showed up because they thought "Web Performance" was a type of dance routine!

During last week's session, we analyzed a popular eCommerce website. In a few short pizza consuming hours, we found:

- A third party performance feature loaded suboptimally and resulting in a single point of failure (SPOF)
- Fonts that were used to style a relatively tiny portion of the pages text.
- A font that was used for only 5 glyphs, and could be easily subsetted
- Preloaded scripts that were not being utilized at all
- A CDN image optimization silent failure that cached an unoptimized image
- Lazy Loaded images that... weren't lazy loading.

But one issue that we discovered surprised me: A font fallback to an alternative format due to a font decoding failure.  This is uncommon, and unlikely to be an issue with your site, but I learned a lot while digging into it and figured I'd share with you all!

## FontAwesome x3

While analyzing a waterfall graph, we noticed that the popular FontAwesome font was being loaded 3 times on this site's homepage. The graph below shows these 3 requests filtered out, and you can see that one font loads after the other. WOFF2, then WOFF, then TTF. The same behavior was evident on Chrome, Safari and Edge.

![A filtered waterfall showing three sequential FontAwesome requests](/assets/img/blog/investigating-a-curious-case-of-font-format-fallbacks/fontawesome-three-font-requests-waterfall.jpg)

The usual cause for seeing multiple font formats for the same font would be preloading multiple versions of the same font asset. However if that was happening, there wouldn't be a sequential pattern of requests. Nonetheless, I checked and there were no font preloads in their Early Hints, HTTP response headers or HTML response payload.

The only place the font was referenced was a single CSS file. And there is nothing wrong with the syntax of this font-face directive. The browser should have selected the best format based on its capabilities. For pretty much all modern browsers that would be WOFF2.

```css
@font-face {
font-family: 'FontAwesome';
src: url('../fonts/fontawesome-webfont.eot?v=4.7.0');
src: url('../fonts/fontawesome-webfont.eot?#iefix&v=4.7.0') format('embedded-opentype'),
url('../fonts/fontawesome-webfont.woff2?v=4.7.0') format('woff2'),
url('../fonts/fontawesome-webfont.woff?v=4.7.0') format('woff'),
url('../fonts/fontawesome-webfont.ttf?v=4.7.0') format('truetype'),
url('../fonts/fontawesome-webfont.svg?v=4.7.0#fontawesomeregular') format('svg');
font-weight: normal;
font-style: normal
}
```

## Font Conversion Errors?

So what's going on here? In Chrome DevTools, the Console provided a clue. There were multiple errors output for each, likely because every time the font was used resulted in an attempt to decode the preferred format.  
* First a "Failed to decode downloaded font" error.
* Next a "Failed to convert WOFF 2.0 font to SFNT" error 
* And finally an  "OTS parsing error: incorrect file size in WOFF header". 

*(I also checked Safari Web Inspector, but there were no errors logged to the console.)*

![Chrome DevTools Console filtered to font errors"](/assets/img/blog/investigating-a-curious-case-of-font-format-fallbacks/devtools-console-font-decode-errors.jpg)

[SFNT](https://en.wikipedia.org/wiki/SFNT) is an acronym that I was unfamiliar with. It stands for "Spline Font" and it's a file format that can contain other fonts. Apparently this format dates all the way back to Apple's [QuickDraw software](https://web.archive.org/web/20131023054643/https://developer.apple.com/fonts/tools/tooldir/TrueEdit/Documentation/TE/TE1sfnt.html), which started using the format in [1991!](https://www.truetype-typography.com/tthist.htm) Well that was a history lesson!

## Digging Into Chromium Source

But what on earth is a web browser doing with SFNT in 2026? Well, examining the source code in Chrome we can see that the WOFF 2.0 and WOFF font file formats are converted internally to TTF fonts using SFNT. So ultimately this container is what holds the font data after it's decoded. And those error messages show up when the browser is unable to parse the font.

*BTW - if the thought of digging into Chromium source code sounds somewhat terrifying, check out Eric Lawrence's excellent blog post on* [*demystifying browsers*](https://textslashplain.com/2020/02/09/demystifying-browsers/)*. It's a few years old, but incredibly helpful and relevant if you want to dig deep into browser internals.*

In Chromium, [web_font_decoder.cc](https://source.chromium.org/chromium/chromium/src/+/main:third_party/blink/renderer/platform/fonts/web_font_decoder.cc;l=233;bpv=1;bpt=1) attempts to decode the WOFF2 file and throws the error we observed when it is unable to decode the font.

![Chromium source at web_font_decoder.cc")](/assets/img/blog/investigating-a-curious-case-of-font-format-fallbacks/chromium-web-font-decoder-convertwoff2tottf.jpg)

The `ConvertWOFF2ToTTF` function also provides some context. The result_length should be the same as determined by `ComputeFinalSize()`. So something seems to be wrong with the size of the font asset.

![Documentation for bool ConvertWOFF2ToTTF](/assets/img/blog/investigating-a-curious-case-of-font-format-fallbacks/woff2-convertwoff2tottf-function-docs.jpg)

The other error appears in [font-face.cc](https://source.chromium.org/chromium/chromium/src/+/main:third_party/blink/renderer/core/css/font_face.cc;l=1015?q=%22OTS%20parsing%20error%22&ss=chromium), which contains a function named `InitCSSFontFace` that attempts to convert the WOFF format to TTF. It throws the error we saw when it is unable to parse the font using [OTS](https://chromium.googlesource.com/external/github.com/khaledhosny/ots/+/HEAD/docs/DesignDoc.md) (which is a sanitizer for OpenType fonts, and another acronym I learned today!).

![Chromium source at font_face.cc](/assets/img/blog/investigating-a-curious-case-of-font-format-fallbacks/chromium-font-face-ots-parsing-error.jpg)

## This font is how many bytes?

When looking at the response headers for the fonts, the `Content-Length` header tells the browser how many bytes are being delivered in the response payload. 
* The WOFF2 file was `77159` bytes
* The WOFF file was `98019` bytes

![DevTools response headers for fontawesome-webfont.woff2](/assets/img/blog/investigating-a-curious-case-of-font-format-fallbacks/woff2-response-headers-content-length.jpg)

Since the error message we observed clued us into a size mismatch, I downloaded each font file and looked at the actual size of the font asset on disk. The WOFF2 file was exactly `77159` bytes and the WOFF file was 98019 bytes. So the HTTP response header matches the file size, the file was fully downloaded by the client, and the CDN server properly reported the content-length.

```bash
$ wget -O fontawesome-webfont.ttf "URL"
$ wget -O fontawesome-webfont.woff "URL"
$ wget -O fontawesome-webfont.woff2 "URL"

$ stat -f "%z %N" fontawesome-webfont.*
165548 fontawesome-webfont.ttf
98019 fontawesome-webfont.woff
77159 fontawesome-webfont.woff2
```

## Interpreting the Font Header

Read that error in DevTools again: `OTS parsing error: incorrect file size in WOFF header`. This wasn't about HTTP, this is about the file itself!

Based on the [WOFF2 specification](https://www.w3.org/TR/WOFF2/#woff20Header), there is a header encoded in a WOFF2 file that contains a signature indicating the file format, the font type and the length. Each of these are UInt32 values, which equates to 4 bytes in the response payload for each value.  A WOFF file contains the same header format

![The WOFF2 specification's section 3.2 WOFF2 Header table](/assets/img/blog/investigating-a-curious-case-of-font-format-fallbacks/woff2-spec-header-table.jpg)

Extracting the first 12 bytes of that file, we get the following:

```bash
$ od -An -tx1 -N12 fontawesome-webfont.woff2
77  4f  46  32  00  01  00  00  00  01  2d  68

$ od -An -tx1 -N12 fontawesome-webfont.woff
77  4f  46  46  00  01  00  00  00  01  7e  e8
```

Grouped into 4 byte values, heres what the WOFF2 file contained:

- `77 4f 46 32` = "wOF2" (you can see some common file signatures [here](https://en.wikipedia.org/wiki/List_of_file_signatures))
- `00 01 00 00` = True Type font
- `00 01 2d 68` = 0x12d68 = 77,160 bytes

And the WOFF file has the same header table format -

- `77 4f 46 46` = "`wOFF`"
- `00 01 00 00` = True Type font
- `00 01 7e e8` = 0x17ee8 = 98,024 bytes

Putting this all together, here's what we have…

| Format | Content-Length | Actual File Size | Font Header Size | Outcome |
|---|---|---|---|---|
| WOFF2 | 77,159 | 77,159 | 77,160 | Failed to convert WOFF 2.0 font to SFNT |
| WOFF | 98,019 | 98,019 | 97,024 | OTS parsing error: incorrect file size in WOFF header |

How the file got corrupted is uncertain. Perhaps there was a version of it encoded incorrectly somewhere. Perhaps there was corrupt download when it was first retrieved for the site. However as it says on Font Awesome's website - "Font Awesome 4 is so 2017". The easy fix was to replace it with a new version of the file, and while they are at it, they should subset the font so that it only loads the bytes needed.

![A banner on the Font Awesome website reading "Font Awesome 4 is so 2017"](/assets/img/blog/investigating-a-curious-case-of-font-format-fallbacks/font-awesome-4-upgrade-banner.jpg)

## Conclusion

These NY WebPerf Meet4Speed sessions are super fun, and it's always interesting to see what rabbit holes it takes us down. Fixing this issue will likely have a measurable effect on FCP for this website. And while it's unlikely that this issue affects a large number of websites, I learned a lot from this investigation and I hope you found some of this insightful too. If you are in the NYC area and want to join us next time, please do!

We also host speaker sessions as well, the next of which is scheduled for this upcoming Thursday!   If you find yourself in NYC and want to give us talk, reach out and we can try to set something up!
