---
title: Python version of the Ask.fm API
author: Hexalyse
type: post
date: 2017-06-14T19:37:57+00:00
url: /2017/06/14/python-version-of-the-ask-fm-api/
categories:
  - Computer stuff
  - Uncategorized
tags:
  - api
  - ask.fm
  - python
  - reverse engineering
---

In the previous articles [How I reverse-engineered the Ask.fm API &#8211; Part 1][1] and [Part 2][2] we saw how I proceeded to analyze and reverse engineer the Ask FM API so I could automate some tasks.

<!--more-->

From there, I decided to implement a basic version of the API, with only a few functions I needed (a lot are missing, like liking an answer, following and unfollowing people, blocking people, etc.). After some time using this ugly code, some people asked me to share my work. I was reluctant to do so for multiple reasons :  First, I&#8217;m not sure about the law around reverse engineering, it&#8217;s always been a grey area to me. Also, I&#8217;m not sure the people who asked me for it would benefit from this code or these articles because they won&#8217;t understand anything and won&#8217;t know how to use it. And lastly, I didn&#8217;t want to share an easy &#8220;working as-is&#8221; executable, because it would allow script kiddies to spam on Ask.fm because of the absence of captchas when using this.

For these reasons (and for intellectual property matters), I voluntarily stripped off the API secret key from the Java code on the Github repository, otherwise Ask.fm might get angry at me for sharing such a thing. I might contact them and if they&#8217;re okay with it, I&#8217;ll add it back.

#### The code

Anyway, enough talk, here is the link to the github repository :

<a href="https://github.com/Hexalyse/pyAskFm" target="_blank" rel="noopener">Hexalyse/pyAskFm &#8211; Basic python implementation of the Ask.fm API</a>

##### Final word

As you can see, there are a lot of things that could be done to improve this code. If you&#8217;re interested in contributing, read the README.md file on github, and feel free to send a Pull Request.

 [1]: https://hexaly.se/2017/06/14/how-i-reverse-engineered-the-ask-fm-api-part-1/
 [2]: https://hexaly.se/2017/06/14/how-i-reverse-engineered-the-ask-fm-api-part-2/