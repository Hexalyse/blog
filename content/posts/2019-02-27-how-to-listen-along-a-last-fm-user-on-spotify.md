---
title: How to listen along a Last.fm user on Spotify
author: Hexalyse
type: post
date: 2019-02-27T20:10:57+00:00
excerpt: |
  Lately I started using Last.fm a bit more thoroughly. For those who don't know what Last.fm is, it's a webstite that allows you to "scrobble" (log) every song you listen to.
url: /2019/02/27/how-to-listen-along-a-last-fm-user-on-spotify/
categories:
  - Computer stuff
tags:
  - lastfm
  - music
  - python
  - spotify
---

_tl;dr &#8220;I dont want to read your article, I want to download the code&#8221;_ : Skip to [this section of the article][1] for the link to the Github repository.

Lately I started using [Last.fm][2] a bit more thoroughly. For those who don&#8217;t know what Last.fm is, it&#8217;s a webstite that allows you to &#8220;scrobble&#8221; (log) every song you listen to.  
It&#8217;s cool because then you can have cool stats about your listening habits, but also discover new artists related to what you like (Spotify largely copied what Last.fm was doing, with the artist/album/song radio and &#8220;daily mix&#8221; playlists).

### The idea

I was browsing the website and looking at what my friends were listening to, when suddenly I thought that it would be cool to be able to listen to what another user is listening to, at the same time. It&#8217;s a cool way to discover new music while relying on someone&#8217;s tastes isntead of the Spotify algorithms, that unfortunately often brings you the same songs that you already know too well.

I started googling, but unfortunately didn&#8217;t find anything remotely close to what I was trying to do, except a very old website called &#8220;overhere&#8221; that apparently did the same thing, but is now long dead.

_So I knew I had to do it myself._

<!--more-->

### The program

After a few moments researching the Last.fm API and the Spotify API, I came to the conclusion it should be doable and reasonably easy to do.

I ended up doing it in Python, since it makes such little scripts easy and quick to code, and libraries were available to interact with the aforementionned APIs.

Basically, the program checks what a user (of your choice) is listening to on Last.fm, and if he/she is listening to a song, it will search for it on Spotify, and play it, if available. At the end of the song, it does it again. Rinse and repeat.

And voilà, you are listening along a Last.fm user, to the exact same songs, at the same time.

### Where to get it ? {#getit}

You can find the code on this Github repository :  
<a href="https://github.com/Hexalyse/LastFmListenAlong" target="_blank" rel="noreferrer noopener" aria-label=" (opens in a new tab)">Hexalyse/</a>**<a href="https://github.com/Hexalyse/LastFmListenAlong" target="_blank" rel="noreferrer noopener" aria-label=" (opens in a new tab)">LastFmListenAlong</a>** 

Everything needed is explained in the README.md file.

_Have fun discovering new music 🙂_

 [1]: #getit
 [2]: https://www.last.fm/