---
layout: post
title:  "Man in the Middle Yourself to See What Your IoT is Up To"
date:   2020-08-09 09:00:00 -0400
categories: [technology]
tags: [tech, diy]
---

![robot](/images/mitm-yourself/safe.jpg){:width="400"}

Can you name all of the Internet connected devices in your house right now? You might be surprised to learn what's connected and what's not and maybe even what you've forgotten about!

Take my mom, for example. She has a MacBook, an iPad, a Galaxy S8, an Amazon FireTV Stick, an Amazon Echo Show 8. And, that's just what I know about. Like many of us, she may also have a smart tv. She has a medic alert fob that she wears around her neck. You know, the "Help! I've fallen and I can't get up!" I bet that has an IP address or, at least the base station does.

In my house, I favor devices that I can hook up to Alexa to control. Every now and then, though, I end up with something that can't (yet) integrate with Alexa. I recently got a mini-split air conditioner installed to keep my office - which sits over a garage - cool in the summer without taxing the rest of the house. The unit I chose has an app, but it does not integrate with Alexa. I'll save the brand-name for a follow up post, because using the tools and techniques I talk about in this post, I discovered a serious security flaw in the app!

One of my favorite posts of all time is about reverse-engineering the [August Lock API](https://medium.com/@nolanbrown/the-process-of-reverse-engineering-the-august-lock-api-9dbd12ab65cb). In this post, the author uses a reverse-proxy, called [Charles Proxy](https://www.charlesproxy.com/) to affect a man-in-the-middle attack on his own network to discover the calls that the August App was making to its servers.

In this post, I will cover a mobile app proxy called [HttpCanary](https://play.google.com/store/apps/details?id=com.guoshi.httpcanary&hl=en_US) as well as [Charles Proxy](). `HttpCanary` is only available for Android, so apologies in advance if you're on an iOS device. Charles Proxy runs on your Mac, Windows or Linux desktop.

## Insomnia is a Good Time to Hack

Earlier in the week, I was struck with some bad insomnia. This happens from time to time with me. After watching 3 episodes of season 2 of Umbrella Academy, I got to thinking about how I could Alexa-ify my mini-split air conditioner. It had an app that required me to register for an account. I can use the app off my network, which means that both the app and the AC connect to some central server somewhere to communicate.

I was already familiar with Charles Proxy, but I (lazily) didn't want to get up out of bed to get my laptop. So, I did a quick search and found HttpCanary. Most reverse proxies available for Android require you to root your device. HttpCanary, ingeniously, installed a certificate and sets up a VPN all within bounds of stock Android. Once I had it configured, I launched the AC app, logged out, logged in again, turned it on and off, and changed the temperature. Switching back to HttpCanary, I could see all this traffic.

If an app is using a service properly, it will have [certificate pinning](https://owasp.org/www-community/controls/Certificate_and_Public_Key_Pinning) configured. This makes it harder for the reverse proxy to do its job. Ordinarily, you'd need a rooted-device with some additional software. You can do this both on Android (rooted) or iOS (Jailbreak). Fortunately, HttpCanary can install a root certificate in Firefox, so it's easy to snoop on all of its network traffic. My AC app did **not** have certificate pinning properly configured, and so I was able to use HttpCanary with it without issue. More on that in the next post!

### HttpCanary for Capturing in Bed

In HttpCanary, you can search for apps that you want to monitor. You then click on the paper airplane icon in the lower right to enable its vpn and start it listening. The cool thing is that on my Samsung (and maybe other Android 10 devices?), I can have canary in a floating window and see all the traffic it's capturing.

![canary-float]({{ "/images/mitm-yourself/canary-float.png" | absolute_url }}){:width="400"}

Here's a fullscreen view of the app:

![canary-full]({{ "/images/mitm-yourself/canary-full.png" | absolute_url }}){:width="400"}

### Use Charles Proxy for the Big Guns

Charles Proxy is a paid app, but it has a free trial. If you really want to dig into what your IoT devices are doing, Charles Proxy is for you. In the screenshot below, You can see some traffic I've captured from afitnerd.com

![afitnerd-charles]({{ "/images/mitm-yourself/afitnerd-charles.png" | absolute_url }}){:width="400"}

I can easily configure my mobile device to use the proxy server running on my laptop. On Android, you go to `Settings > Connections > Wi-Fi`. Touch the gear icon next to your Wi-Fi connection and choose `Advanced`. Then, choose `Proxy > Manual` and enter in the IP address of the machine your Charles Proxy is running on. 

**Note**: If you're not sure of the IP Address where Charles Proxy is running, you can choose `Help > Local IP Address` from the menu and it will tell you. Also, Charles Proxy runs on port `8888` by default.

## What's Next in IoT Hacking

This week I will disclose my security findings to the maker of my AC unit. Hopefully they will respond and indicate that they are fixing the issue. Keep an eye out for the next post, where I'll dig into hacking my AC unit with Charles Proxy and HttpCanary.