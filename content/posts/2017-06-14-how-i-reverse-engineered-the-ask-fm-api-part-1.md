---
title: How I reverse-engineered the Ask.fm API – Part 1
author: Hexalyse
type: post
date: 2017-06-14T18:43:36+00:00
url: /2017/06/14/how-i-reverse-engineered-the-ask-fm-api-part-1/
categories:
  - Computer stuff
  - Uncategorized
tags:
  - api
  - ask
  - ask.fm
  - python
  - reverse engineering

---
In this first post, we&#8217;ll see how I managed to reverse engineer the <a href="https://ask.fm/" target="_blank" rel="noopener noreferrer">Ask.fm</a> API. In a subsequent post, I might give the (ugly) code I came up with, which implements some of the API features I needed for various bots and scripts.

<!--more-->

If you don&#8217;t already know [Ask.fm][1], it&#8217;s a social network where people can create profiles and can send each other questions&#8230; and answer it, obviously.

Unfortunately, their website has nasty limitations (<a href="https://www.google.com/recaptcha/intro/" target="_blank" rel="noopener noreferrer">Google reCAPTCHA</a>) that prevented me to do what I wanted. But I noticed their application never ask for any captchas. So I guessed it was using a backend API, and if I can use it, I can code bots more easily to automate various tasks (like sending &#8220;Group Questions&#8221;, a widespread practice amongst ask.fm users). I searched if their API was documented and if they offered developers access to it : they didn&#8217;t. I searched the web for people who would have implemented it in any language, and found nothing.

So it was my job to reverse engineer and implement (a basic version of) their API in Python. I will retrace my journey, step by step, of how I finally got to the point of intercepting API calls made from the Android app, to the ask.fm servers.


#### Step 1 : Using Fiddler to sniff API calls made from the app

The first obvious step was to install an HTTP-proxy application to be able to intercept the HTTP requests and replies between the app and their servers. I decided to use <a href="http://www.telerik.com/fiddler" target="_blank" rel="noopener noreferrer">Fiddler</a> because it&#8217;s free and supports HTTPS traffic recording (via a MitM attack using a crafted SSL certificate, thus allowing to decode HTTPS data, then re-encrypt it and sending it to the other party). So I installed Fiddler, activated the HTTPS decoding option, and installed the root certificate they provide you on my Android phone. You need to do this step, or otherwise, the SSL certificate crafted by Fiddler won&#8217;t be accepted by your phone, since it won&#8217;t have the root certificate to validate the chain. Then you just have to configure your WiFi connection to use a proxy, and use the correct IP address and port to point to the computer where Fiddler is running.

Unfortunately, even after doing all this, the Ask.fm application didn&#8217;t seem to work when I was proxying the network connection. It kept telling me &#8220;No internet connection&#8221;. But the connection was good, because everything worked fine in a browser and I could even access the HTTPS Ask.fm mobile website in Firefox and see every requests made, decrypted in Fiddler.

So there was another problem. And suddenly it stroke me : they must be using some sort of certificate pinning. What this mean is, even if the certificate given by Fiddler to the ask.fm application is valid, the application is waiting for a very specific certificate. Every others would be rejected. This is a very good practice, because it prevents such Man-In-The-Middle attack from potentials attackers who could then steal your credentials. And it also prevents people from reversing their API.

But I wasn&#8217;t done already.

#### Step 2 : Installing Xposed framework to disable certificate pinning

Because there is a way to bypass certificate pinning. If you have the <a href="http://repo.xposed.info/module/de.robv.android.xposed.installer" target="_blank" rel="noopener noreferrer">Xposed framework</a> installed on your phone (requires root access), there are plugins for it which sole purpose is to bypass certificate pinning. There are multiple of them, actually :

  * <a href="https://github.com/Fuzion24/JustTrustMe" target="_blank" rel="noopener noreferrer">https://github.com/Fuzion24/JustTrustMe</a>
  * <a href="https://github.com/ac-pm/SSLUnpinning_Xposed" target="_blank" rel="noopener noreferrer">https://github.com/ac-pm/SSLUnpinning_Xposed</a>
  * <a href="http://repo.xposed.info/module/mobi.acpm.sslunpinning" target="_blank" rel="noopener noreferrer">http://repo.xposed.info/module/mobi.acpm.sslunpinning</a>

Yaaay \o/ We just need to install one of this, and we&#8217;ll finally be able to intercept traffic from the app !

#### Step 3 : Intercepting API calls, then trying to figure how it works

Let&#8217;s try sniffing traffic again : It works !

Here is an example of a request (the one sent when we login in the app) :

<pre>POST /authorize HTTP/1.1
X-Api-Version: 0.8
Host: api.ask.fm:443
X-Client-Type: android_3.8.1
Accept: application/json; charset=utf-8
X-Access-Token: .4WspnFnDpwQNevsbXIEExPDgJZDM
Accept-Encoding: identity
Authorization: HMAC a9f98b69f649e0c96240cc6e36980da96f308cea
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
User-Agent: Dalvik/2.1.0 (Linux; U; Android 6.0.1; GT-N7100 Build/MOB30R)
Connection: Keep-Alive
Content-Length: 186

json {"did":"84a2b70bfae4ae65","guid":"84a2b70bfae4ae65","pass":"password123lol","rt":"4","ts":"1471967146","uid":"JohnDoe"}</pre>

So now we&#8217;ve captured traffic going between the AskFM app and their servers, we need to analyze it.

We&#8217;ll cover this in [the part 2 of this article][2].

 [1]: https://ask.fm/
 [2]: https://hexaly.se/2017/06/14/how-i-reverse-engineered-the-ask-fm-api-part-2/