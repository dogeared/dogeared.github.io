---
layout: post
title:  "Weekend Project: Fix Dark Tower"
date:   2011-10-16 16:52:41 -0400
categories: [diy]
tags: [diy]
---
<p>This post is about replacing the keypad on the game Dark Tower. If you're not interested in my nostalgic blathering, click <a href="#howto">here</a> to get right to the instructions.</p>

<strong>Nostalgic Blathering</strong>

<p>For those of you not familiar (because you're too young or live under a rock), <a href="http://en.wikipedia.org/wiki/Dark_Tower_(game)" target="_blank">Dark Tower</a> was one of the first interactive electronic board games that was massed produced. It was released in 1981 and I got one for my 13th birthday. It blew my mind - not only because it was fun to play - but because of the possibilities that electronics could bring to gaming. I had been hacking around on <a href="http://en.wikipedia.org/wiki/Commodore_PET" target="_blank">Pet</a> computers for about 2 years by that point. While there were some character based green-screen games for the Pet, Dark Tower was the first mixed media game I had ever played. You set up a physical <a href="http://www.google.com/search?q=dark+tower+game&hl=en&prmd=imvns&source=lnms&tbm=isch&ei=EAubTsGuE-rv0gHl3PnpBA&sa=X&oi=mode_link&ct=mode&cd=2&ved=0CD4Q_AUoAQ&biw=1400&bih=737#hl=en&tbm=isch&sa=1&q=dark+tower+game+board&pbx=1&oq=dark+tower+game+board&aq=f&aqi=&aql=1&gs_sm=e&gs_upl=56646l57429l0l57772l6l6l0l3l0l1l171l484l0.3l3l0&bav=on.2,or.r_gc.r_pw.,cf.osb&fp=638d9931109a0b28&biw=1400&bih=737" target="_blank">map board</a>, complete with plastic buildings, flags and character pieces. You monitored your progress through the game with a cardboard status indicator which used re-purposed <a href="http://en.wikipedia.org/wiki/Battleship_(game)" target="_blank">battleship</a> red peg markers. On the peg board, you kept track of how many warriors, gold, food, keys and other items you had during game play. At the center of it all was the Dark Tower. It sported a 12 button keypad and a display that tells you what had happened on each turn of the game. Display is a generous word in this context as it really was just a vertical row of three lights behind a plastic drum with pictures. The drum turned to the correct position and the correct light illuminated behind the relevant picture for what just occurred in the game. The game came in a <a href="http://www.google.com/search?q=dark+tower+game&hl=en&prmd=imvns&source=lnms&tbm=isch&ei=EAubTsGuE-rv0gHl3PnpBA&sa=X&oi=mode_link&ct=mode&cd=2&ved=0CD4Q_AUoAQ&biw=1400&bih=737#hl=en&tbm=isch&sa=1&q=dark+tower+game+box&oq=dark+tower+game+box&aq=f&aqi=&aql=&gs_sm=e&gs_upl=9629l10581l0l10699l4l3l0l0l0l2l213l394l0.1.1l3l0&bav=on.2,or.r_gc.r_pw.,cf.osb&fp=638d9931109a0b28&biw=1400&bih=737" target="_blank">huge cardboard box</a> with artwork on it that fit right in with the <a href="http://en.wikipedia.org/wiki/Dungeons_%26_Dragons" target="_blank">Dungeons & Dragons</a> mindset of the early '80s.</p>

<strong>The Keypad Problem</strong>

<p>Complete Dark Tower game sets in good working condition go for as much as $400 on Ebay these days (average price is probably $100). Back In 2003, my brother bought me a complete working Dark Tower set from an antique shop in New York City for my 35th Birthday. My brother, sister and I (as well as others) played the hell out of the game for 2 years before the keypad crapped out. The original keypad was constructed of 3 thin plastic sheets (see pictures <a href="#keypads">below</a>). Two of the sheets have metal contacts and are separated from each other by a blank plastic sheet with holes in it. When you press a "key", the two metal contacts touch through the plastic sheet in the metal. Although pretty tough, it is this simple construction that tends to wear out first on these games.</p>
<p>When my siblings and I got together, we always had a hankering to play Dark Tower. So, we decided to try to fix the keypad. With a little copper tape and one solder joint, we were able to get the keypad working again. This fix lasted us another eight years - pretty good for our MacGyver-like fix. At a recent company game night, the keypad crapped out again in the middle of a game. This was after I had introduced the game to a bunch of twenty-somethings who had never heard of it before and who were enjoying it quite a lot. I thought, "This shall not stand, man!" That thought was quickly replaced with a "WTF" as I looked online and found Dark Tower repairs costing $70 - $100. Thankfully, I have been given the gift (and curse) of being a compulsive tinkerer and DIYer.</p>
<p>This weekend with the help of Adafruit.com's <a href="https://www.adafruit.com/products/419" target="_blank">keypad</a> (2 of them - 1 had to be sacrificed so you wouldn't have to), some scribbled notes, some nail-biting-trimming of said keypad with a scissor and a wire wrapping tool, my brother and I completely replaced the keypad and have a fully functional Dark Tower again. As an added bonus, the Adafruit keypad has a satisfying click when pressed - a feature missing from the 1980s pure membrane keypad. While I had hoped I would be able to simply plug in this new keypad, the universe laughed. If you want to skip the background on how these keypads work and just get to the steps, click <a href="#howto">here</a>.</p>

<strong>How Membrane Keypads work</strong>

<p>For a 12-button keypad, you only need seven pins to determine which key was pressed (3 columns and 4 rows). Each key press combines two of the seven pins when contact is made. When I first hooked up the new keypad, I quickly realized that the pin mappings were very different as almost none of the keys worked. This is the reason that I bought two of the Adafruit keypads. I ripped one of them apart so that I could see exactly how each of the keys connected to the pins. I was able to do the same for the original keypad, which was pretty much toast anyway. I'm going to refer to the original keypad by number, just like the new keypad has and just like a standard telephone dialing keypad has. </p>
<pre>
Key: (x,y) <========= pins
       z   <========= number on keypad

          Old Keypad                        New Keypad
=============================     =============================
| (5,2)     (4,2)     (3,2) |     | (3,7)     (2,7)     (1,7) |
|   1         2         3   |     |   1         2         3   |
|                           |     |                           |
| (5,1)     (4,1)     (3,1) |     | (3,6)     (2,6)     (1,6) |
|   4         5         6   |     |   4         5         6   |
|                           |     |                           |
| (5,6)     (4,6)     (3,6) |     | (3,5)     (2,5)     (1,5) |
|   7         8         9   |     |   7         8         9   |
|                           |     |                           |
| (5,7)     (4,7)     (3,7) |     | (3,4)     (2,4)     (1,4) |
|   *         0         #   |     |   *         0         #   |
=============================     =============================
</pre>
<a name="howto"><strong>Cable Pin Mappings</strong></a>
<p>With the two pin maps above, I could now create a cable to properly connect the new keypad to the internal connector of the Dark Tower. Here is the pin map:</p>
<pre>
 DT Connector     New Keypad
   1 =============== 6
   2 =============== 7
   3 =============== 1
   4 =============== 2
   5 =============== 3
   6 =============== 5
   7 =============== 4
</pre>

NOTE: I arbitrarily chose "pin 1" to be the top of the internal connector with the Dark Tower laying on its side as pictured <a href="#dtside">below</a>.

<a name="keypads"/>
<p>Here are pictures of the two keypads, closed, open, and with pin explanations:</p>

<a href="/images/2011/10/OldAndNewKeyboardsClosed.jpg"><img src="/images/2011/10/OldAndNewKeyboardsClosed.jpg" alt="" title="OldAndNewKeyboardsClosed" width="409" /></a>

<a href="/images/2011/10/OldAndNewKeyboardsOpen.jpg"><img src="/images/2011/10/OldAndNewKeyboardsOpen.jpg" alt="" title="OldAndNewKeyboardsOpen" width="436" /></a>

<p><a href="/images/2011/10/OldKeyboardPinoutSmall.jpg"><img src="/images/2011/10/OldKeyboardPinoutSmall.jpg" alt="" title="OldKeyboardPinoutSmall" width="770" height="323" class="alignnone size-full wp-image-543" /></a></p>
<p><a href="/images/2011/10/NewKeyboardPinoutSmall.jpg"><img src="/images/2011/10/NewKeyboardPinoutSmall.jpg" alt="" title="NewKeyboardPinoutSmall"/></a></p>

<strong>Test Run: prototype</strong>

The first thing I wanted to do was test out my pin mappings. I used relatively heavy gauge wire connect the new keypad to the connector in the Dark Tower using the mappings above.

<a name="dtside"/>
<a href="/images/2011/10/DTPinMapTest.jpg"><img src="/images/2011/10/DTPinMapTest.jpg" alt="" title="DTPinMapTest" width="436" /></a>

And... it worked!

<strong>Make the Cable</strong>
<p>The new keypads each came with a 7 pin header. I thought I would simply solder wires to the headers in the appropriate order outlined above, and then plug one end into the socket in the Dark Tower and the other end into the keypad. Sadly, the universe laughed again. My soldering skills are adequate at best. What I had forgotten is that the pins on these headers are meant to break away. That way, you can have a 2-pin header or a 10-pin header - whatever you need. As I began soldering the first wire onto the first pin, the heat melted the connecting plastic in the middle and the pin just fell off from the rest of the bunch. No to be deterred, I remembered that I in my college days, I used a super handy <a href="http://www.radioshack.com/product/index.jsp?productId=2103243" target="_blank">wire wrapping tool</a> and very thin <a href="http://www.radioshack.com/product/index.jsp?productId=2062642" target="_blank">wrapping wire</a>. This tool made it very easy for the lazy and soldering inept (me, basically) to still put together complex projects involving connecting up electronic components without having to solder them. One issue I found with wire wrapping on the square pins of the header was that it is easy to break the wire if you're not careful. After a few attempts, I got the technique down, which involves wrapping a little more slowly and moving the wrapping tool out as you wrap. I built my cable, but you can see that for the 7th pin, I used a bit of heavier gauge wire since I had destroyed the original pin from the header.</p>
<p><a href="/images/2011/10/KeyboardConnectorCable.jpg"><img src="/images/2011/10/KeyboardConnectorCable.jpg" alt="" title="KeyboardConnectorCable" width="436" class="alignnone" /></a></p>

<strong>Test Run: for realz</strong>

The next step was to test to the cable in the Dark Tower:

<a href="/images/2011/10/DTNewPinTest.jpg"><img src="/images/2011/10/DTNewPinTest.jpg" alt="" title="DTNewPinTest" width="436"/></a>

And... it failed. The first time it failed because I had screwed up the pin connections. The second header only having 6 pins threw me off (pictured above is the final working cable). I re-wrapped all the wires (so much easier than de-soldering!) and tried again. This time, all of the keys worked except the bottom row. The reason for this, we found, was that pin 7 was not making good contact in the connector on the Dark Tower (hey, the thing's 30 years old - cut it some slack). Jamming a small piece of the heavier gauge wire in with the header on pin 7 got it working. I realized that I needed to ensure that the header wouldn't fall out of the internal connector. I thought about gluing it in, but I wanted a less permanent solution. So, I jammed small pieces of the heavier gauge wire in at pin 1 and pin 3 on the internal connector. We'll see how this holds up long term. I may need to come up with a better solution for keeping the cable plugged in internally. (After the picture below was taken, I trimmed the bits of wire so there is no exposed copper).

<a href="/images/2011/10/InternalConnector.jpg"><img src="/images/2011/10/InternalConnector.jpg" alt="" title="InternalConnector" width="436"/></a>

In order for the keypad to fit back into the dark tower, I had to trim it. I did this by measuring against the plastic bezel that it sits in and by holding the keypad up to a bright light to ensure that I would not be cutting through any of the traces. I then put electrical tape carefully around the exposed header pins on my cable. I needed to put the original keypad decal on top of the new keypad and to put the whole together with the plastic bezel. I thought about using adhesives, but since the whole thing is held in with a fitted plastic outline from the bottom, I decided to just layer everything and let the pressure hold it together. This way, if any future adjustments need to be made, I won't have to damage anything getting it apart again.

<a href="/images/2011/10/InternalHookup.jpg"><img src="/images/2011/10/InternalHookup.jpg" alt="" title="InternalHookup" width="436"/></a>

<strong>Closing Time</strong>

It was finally time to close the patient up. The bottom of the new keypad could not be trimmed at all since all the traces connect there. By trimming the top, I was able to at least make it flush with the bottom of the Dark Tower. This resulted in a slight warping of the outer tower once assembled and there is some pressure being put on the ribbon cable, but neither of those issues are show stoppers.

<a href="/images/2011/10/DTBottom.jpg"><img src="/images/2011/10/DTBottom.jpg" alt="" title="DTBottom" width="436"/></a>

Once everything was closed up, we did another test where we exercised all the keys. Everything worked great. The tactile feedback of a click when a key is pressed is great. The keys are not 100% lined up with the original overlay, but it's close enough that neither my brother, nephew or I hit the wrong key by mistake when we played. Here, you can see me spanking some Brigands.

<a href="/images/2011/10/Warriors.jpg"><img src="/images/2011/10/Warriors.jpg" alt="" title="Warriors" width="436"/></a>

<a href="/images/2011/10/Brigands.jpg"><img src="/images/2011/10/Brigands.jpg" alt="" title="Brigands" width="436"/></a>

<strong>DIY Satisfaction</strong>

So, for $12.06 in parts from Adafruit (includes 2 keypads, tax and shipping) and $11.60 in parts from Radio Shack (the wire wrap tool and the wrapping wire - which I will use again), I was able to bring back to life a classic (and favorite) game. I'll be bringing it back to game night in November at work and we should be able to get a full game in this time!

<strong>Parts List and tools</strong>
<ul>
<li><a href="https://www.adafruit.com/products/419" target="_blank">Adafruit keypad</a>
<li><a href="http://www.radioshack.com/product/index.jsp?productId=2103243" target="_blank">Wire wrapping tool</a>
<li><a href="http://www.radioshack.com/product/index.jsp?productId=2062642" target="_blank">Wrapping wire</a>
<li>electrical tape
<li>scissors
<li>Philips head screwdriver
</ul>
