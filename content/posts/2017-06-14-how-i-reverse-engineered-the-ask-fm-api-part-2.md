---
title: How I reverse-engineered the Ask.fm API – Part 2
author: Hexalyse
type: post
date: 2017-06-14T19:24:33+00:00
url: /2017/06/14/how-i-reverse-engineered-the-ask-fm-api-part-2/
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

In the second part, we&#8217;ll take a look at the requests we intercepted thanks to Fiddler, and see if Aks.fm implemented some sort of security (tokens, checksums, etc)
<!--more-->

#### Understanding the request headers and parameters

Here is the request we&#8217;ll analyze :

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

As we can see , there are lots of interesting headers here. I&#8217;ll not make you wait and explain directly what I deducted from my tests :

<!--more-->

  * **X-Client-Type**: This header indicates the version number of the Ask.fm application used. Let&#8217;s just use the same as the one we captured.
  * **X-Api-Version**: This is the&#8230; API version, you guessed it. It is important because the parameters we need to send to the API change with each version and they are NOT retro-compatible ! This must be a nightmare to maintain for the developers.
  * **X-Access-Token**: Before you make any requests, you need to make a GET request to the address https://ask.fm/token, with a few get parameters (the device id, the current timestamp and a third parameter we&#8217;ll explain later). Then you use this token in your authorization POST request. The token you get back from this request will be the token that identify you and keeps trace of your &#8220;session&#8221;.
  * **Authorization**: Aaaaah, this header is the one who gave me a headache. We can see it sends an HMAC, which is just a glorified checksum, used to verify the request. The application basically generates this checksum (HMAC) from the URL  and the parameters of the request. Then, when the server receive your request, it will compute the HMAC and compare it to the one you sent. If it&#8217;s the same, the request is valid. Otherwise, the server will send you a nasty &#8220;Invalid request&#8221; error message.
  * Other headers are simple and classical HTTP headers

So&#8230; the three first headers are simple to reproduce. But this HMAC Authorization header is a barrier : we need to find how this HMAC is generated. And for this, we&#8217;ll have to dwelve into the AskFM source code. We&#8217;ll look at it later. For now, let&#8217;s look at the request parameters.

Every POST request has only one parameter, called &#8220;json&#8221;, containing an URL-encoded version of all the parameters in JSON. What is funny, is that I discovered you must format this JSON parameter exactly like the application does, or else the request will be considered invalid (I&#8217;m really wondering how they parse it). The correct formatting is : the keys must be alphabetically sorted, and there must be no spaces or new lines. Everything must be on a single line. OK, why not&#8230;

Also, besides the parameters specific to each request, there are four parameters that will always be present :

  * &#8220;did&#8221;: The Android device ID, in hexadecimal
  * &#8220;guid&#8221;: The same ID (can it be different ? I&#8217;m not an Android expert)
  * &#8220;rt&#8221;: I discovered this is a counter that is incremented at each request made. Maybe they use it for rate analytics ? Or for an ugly rate limiting ?
  * &#8220;ts&#8221;: This is just the current timestamp (in seconds)

#### Decompiling the apk to find the HMAC generation routine

So now we only need one last thing : to know how this HMAC is generated. For this we&#8217;ll need to decompile the apk. You can do it with tools like APKTool, but there are websites like <https://www.apkdecompilers.com/> that allows you to send an APK and download the decompiled code. I used a software with a GUI but I don&#8217;t remember its name.

Anyway, now we have the decompiled &#8220;human readable&#8221;, somewhat-Java-ish code, we just need to go through the different packages name and find the ones that sounds useful. I did it months ago so I don&#8217;t exactly remember how I proceeded and how I ended up finding the code that was interesting. All I know is that the code we&#8217;re interested in is the package **com.askfm.network.utils** and the class is called **Signature**.

Here is the decompiled Java code : <a href="https://gist.github.com/Hexalyse/fe31295ba1ff6685397e3bee955350b0" target="_blank" rel="noopener noreferrer">https://gist.github.com/Hexalyse/fe31295ba1ff6685397e3bee955350b0</a>

As we can see, the code uses a private key (the HMAC secret key), which is stored somewhere else in the source code. I won&#8217;t tell you where since&#8230; I don&#8217;t remember. But it&#8217;s pretty easy to find.

**UPDATE** : I&#8217;ve been informed by a reader that the developers changed the way the HMAC secret key is generated in the newer versions of the app. It is no longer stored as plain-text in the code. It is instead generated via another Java class. If you want it, you can extract this class, import it in your own Android app project, compile it, run it, and you&#8217;ll have the key without spending too much time reversing the code.

From there, we can just replace this function call with the secret key and we have a Java class we can use to generate a valid HMAC 😀

#### Conclusion and source code

I will stop this article here, since all that is left is clean up the Java code or re-implement it in any language we want. Then we just have to spend a LOT of time analyzing the requests to each of the API endpoints we&#8217;re interested in, and implement it.

I chose to make a very basic implementation of the API with only the functions I needed for a few projects I had which needed some sort of automation (posting answer, sending questions, etc.), but I was too lazy to port the Java code to Python so I kept it in Java.

If you want to get the source code, you can get it on Github : <a href="https://github.com/Hexalyse/pyAskFm" target="_blank" rel="noopener noreferrer">Hexalyse/pyAskFm &#8211; Basic python implementation of the Ask.fm API</a>