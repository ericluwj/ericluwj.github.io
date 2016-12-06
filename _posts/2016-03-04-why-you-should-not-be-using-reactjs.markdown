---
layout: post
title:  "Why you should not be using ReactJS"
description: "But let's say if I just want to create a really simple landing page with mostly static content and no routes, is React really a good technology to start with? Here is my conclusion: React is simply an overkill for a simple static page with mostly static content."
date:   2016-03-04
---

<p class="intro"><span class="dropcap">J</span>ust so that you don't get me wrong, I am certainly not a ReactJS hater. In fact, I have adopted the use of React for one of my latest web projects. However, with the rising trend in the adoption of React, I noticed the rise of a whole lot of React static site projects and generators, as though React is the perfect solution to creating a static site. Is that really the case?</p>

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- ericluwj.com -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-3690983003821502"
     data-ad-slot="1621079697"
     data-ad-format="auto"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

*Update: Just read [https://hackernoon.com/how-it-feels-to-learn-javascript-in-2016-d3a717dd577f](https://hackernoon.com/how-it-feels-to-learn-javascript-in-2016-d3a717dd577f) on the entire drama around React + Javascript. It is just so hilarious. The point is, if you just want to use ReactJS just because it is **cool**, it may not be such a good idea.*

Interestingly, React seemed to have recently become very popular for creating static sites (especially a single-page application), as though it serves a great need or purpose, and you may just have been misled into thinking that React is fundamentally important for static pages. But let's say if I just want to create a really simple landing page with mostly static content and no routes, is React really a good technology to start with? Here is my conclusion:

> React is simply an overkill for a simple static page with mostly static content.

I hope you got that right. React simply does not provide enough value for a simple landing page, not to mention the whole lot of hassles that come with its use. Here are some reasons:

1. React was built for fast and responsive user interfaces on *data-driven* sites. If your site contains mostly static content, where's the variable data for React to handle? What does your site need to respond to? What you want will probably a site that loads fast and not respond fast.
2. At 35Kb in gzipped format, React provides no useful utility function nor any user interface code that you will probably use for your static site. Even simple AJAX calls still require the use of `fetch` or jQuery. So much for 35Kb.
3. With React, you are going to have fun wrapping parts of your HTML in `components` which really does not serve much purpose because you would rarely reuse any component or extend it. You might as well just create one single big component. The time spent doing this is simply not worth it. If you have parts of code that need to be repeated on different pages or routes, you would be better off using a simple templating language.
4. Using JSX in React does not easily allow you to put scripts such as the Google Analytics tracking code into your HTML, so you are going to spend a lot more time than just simply copying and pasting a tracking code.

Before you start using React, the main criteria for really determining whether you should be using React is whether your site is going to be *data-driven*.

Keep that in mind.

So what technology is actually the key to creating a great static site? Check out [Webpack](http://webpack.github.io/).

Unfortunately, it is hard to find a Webpack static site generator that does not include React, and I am personally quite puzzled, considering that React is actually a pretty redundant technology for a simple landing page. FTW, Webpack/React are actually not related to each other, but you can use them together.