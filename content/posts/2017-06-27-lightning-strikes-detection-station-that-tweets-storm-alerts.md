---
title: Raspberry Pi lightning strikes detection station that tweets storm information
author: Hexalyse
type: post
date: 2017-06-27T14:08:58+00:00
url: /2017/06/27/lightning-strikes-detection-station-that-tweets-storm-alerts/
categories:
  - Computer stuff
  - Uncategorized
tags:
  - lightning
  - raspberry pi
  - RPI
  - sensor
  - storm
  - tweet
  - twitter
---

A few days ago I was browsing the world wide web, when suddenly I stumbled upon an article speaking about an IC sensor made by AMS, called the <a href="http://ams.com/eng/Products/Wireless-Connectivity/Wireless-Sensor-Connectivity/AS3935" target="_blank" rel="noopener">AS3935 Franklin Lightnin Sensor</a>, which could detect the electrical signature of lightning strikes up to 40km and give the &#8220;strength&#8221; of the lightning as well as an estimation of the distance to the head of the storm. And all this in a tiny 4x4mm SMD circuit designed to be embedded in personal weather stations, watches, cars, etc.

My <del>dark</del> nerd side was tickled and I suddenly <del>needed</del> wanted to buy one and build a lightning detection station that could tweet about incoming storms, using my Raspberry Pi which is lying on a table doing nothing most of the time. And so I did it ! What I ended up with is a Twitter account on which lightning strikes info is tweeted in real time, with energy and distance estimation : <https://twitter.com/toulouse_orages>
<!--more-->

After some research, I found that <a href="http://www.embeddedadventures.com/as3935_lightning_sensor_module_mod-1016.html" target="_blank" rel="noopener">Embedded Adventure was selling a module</a> with a pre-soldered sensor with the right antenna and components added to the board, ready to be used via an I2C interface. Some more research and I found that somebody already implemented a <a href="https://github.com/pcfens/RaspberryPi-AS3935/" target="_blank" rel="noopener">Python library to use such sensors on a Raspberry Pi</a>. Perfect ! We have everything we need.

<img loading="lazy" class="alignnone size-medium wp-image-100" src="https://hexaly.se/wp-content/uploads/2017/06/MOD-1016v8_1_600-300x200.jpg" alt="MOD-1016" width="300" height="200" srcset="https://hexaly.se/wp-content/uploads/2017/06/MOD-1016v8_1_600-300x200.jpg 300w, https://hexaly.se/wp-content/uploads/2017/06/MOD-1016v8_1_600.jpg 600w" sizes="(max-width: 300px) 100vw, 300px" /> 

#### Wiring the sensor to the Raspberry Pi

Here is the wiring diagram to connect the sensor to the Raspberry Pi (you can use a different GPIO pin for the IRQ, but remember to change code accordingly):

| AS3935 Pin | MOD-1016 Pin | Raspberry Pi Pin |
| ----------:|:------------:|:---------------- |
|    4 (GND) |     GND      | 25 (Ground)      |
|    5 (VDD) |     VCC      | 1 (3v3 Power)    |
|   10 (IRQ) |     IRQ      | 11 (GPIO 17)     |
|  11 (I2CL) |     SCL      | 5 (SCL)          |
|  13 (I2CD) |  SDA / MOSI  | 3 (SDA)          |

Here is a picture of my setup. For now, it&#8217;s just a quick prototype set up on a breakboard :

[<img loading="lazy" class="alignnone wp-image-70 size-medium" src="https://hexaly.se/wp-content/uploads/2017/06/IMG_20170627_1518392-226x300.jpg" alt="Raspberry Pi + AS9535 Lightning sensor" width="226" height="300" srcset="https://hexaly.se/wp-content/uploads/2017/06/IMG_20170627_1518392-226x300.jpg 226w, https://hexaly.se/wp-content/uploads/2017/06/IMG_20170627_1518392.jpg 603w" sizes="(max-width: 226px) 100vw, 226px" />][1]

#### Writing the code to make it work

The Python library is pretty well self-documented, so just browse through the code and it should be enough. If you want to understand exactly how each of the sensor parameter works (Watchdog Threshold, Signal Rejection, etc, even some of which are not implemented in the library), refer to the <a href="http://www.embeddedadventures.com/datasheets/AS3935_Datasheet_EN_v2.pdf" target="_blank" rel="noopener">AS9535 datasheet</a>.  
<span style="font-size: 80%;">It could be useful to change those parameters that are not directly exposed, and it&#8217;s pretty easy to do so by directly calling <span class="pl-en">RPi_AS3935</span>.<span class="pl-en">set_byte</span>(register, value) (just be sure to call <span class="pl-en">RPi_AS3935</span>.read_data() first, and get the current value of the register so you only change the bits you need in it). For the Twitter part of the script, I chose to use the Tweepy module.</span>

I just wanted to write a script that would listen for lightning, then tweet when a storm is happening. So first, to avoid getting false positives, I set the &#8220;minimum strikes&#8221; parameter to 5 (the sensor waits for 5 strikes in a predefined period before it starts sending IRQs for each subsequent strike). Then I have a handler which is run each time an IRQ is received. If it&#8217;s a strike, we tweet about it. But to avoid spamming tons of tweets, I set up a 5 minutes delay : if more than one strike happen during those 5 minutes, I&#8217;ll just tweet the number of strikes that have been detected. After 30min with no activity from the sensor, we can declare that the storm is over and tweet people that they should be okay now.

Here is an example tweet (yes it tweets in French, because I live in France. Baguette.) :

<img loading="lazy" class="alignnone wp-image-89 size-full" src="https://hexaly.se/wp-content/uploads/2017/06/Tweet-1.png" alt="" width="583" height="106" srcset="https://hexaly.se/wp-content/uploads/2017/06/Tweet-1.png 583w, https://hexaly.se/wp-content/uploads/2017/06/Tweet-1-300x55.png 300w" sizes="(max-width: 583px) 100vw, 583px" /> 

The translation would be &#8220;_/!\ 117 strikes detected in the last 5 minutes. Power of the last strike : 429714 &#8211; distance to the head of the storm : 1km_&#8220;.  
As you can see, it was a pretty violent storm, with tons of intra-cloud strikes happening and being detected almost every other second (which is close to the maximum the sensor can detect, since in the datasheet we can read it waits for 1,5s after a strike has been detected and analyzed, before it starts listening again).  
The Twitter account is : <a href="https://twitter.com/toulouse_orages" target="_blank" rel="noopener">https://twitter.com/toulouse_orages</a>

##### Final words

I had the chance to set it up right before a period of storms, here in Toulouse. At first I thought it might be crazy because it detected hundreds of strikes every few minutes. But no&#8230; it&#8217;s been 4 days now and I saw no false positives. The sensor didn&#8217;t detect any lightning for 48 hours, and started going crazy anytime a storm was approaching ! So I&#8217;m really happy with the results, that are much better than I would have thought from a little sensor like it. Especially since I didn&#8217;t take any time to calibrate it properly and mainly used factory settings. I just used the antenna capacitor tuning value given by Embedded Adventure, but it can be a good idea calibrating it again since temperature and other factors can influence it. A way to calibrate the antenna by reading the oscillation speed from the IRQ pin using an Arduino is <a href="https://github.com/evsc/ThunderAndLightning#tuning-the-antenna" target="_blank" rel="noopener">explained here</a> : as you can see, the process could be easily automatized.

If you&#8217;re interested in building your own station, **you can find the final Python code for this bot on Github : <a href="https://github.com/Hexalyse/LightningTweeter" target="_blank" rel="noopener">Github &#8211; Hexalyse / LightningTweeter</a>**

Feel free to fork it, modify it and hack it to your needs ! If you have any question, remark or observation, do not hesitate to post a comment.

Hope you enjoyed 🙂 Happy storm-hunting !

 [1]: https://hexaly.se/wp-content/uploads/2017/06/IMG_20170627_1518392.jpg