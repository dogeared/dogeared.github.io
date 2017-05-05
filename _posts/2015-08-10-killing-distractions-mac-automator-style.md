---
layout: post
title:  "Killing Distractions, Mac Automator Style"
date:   2015-08-10 16:52:41 -0400
categories: [tech, productivity]
tags: [tech, productivity]
---

It has been a long long while since I've posted anything here. Life just kind of got in the way. Lame. Frankly, I've gotten a bit away from the "fit" part of afitnerd.com. Here's what I am excited about: I recently took a position as the Java Developer Evangelist for <a href="https://www.stormpath.com" target="_blank">Stormpath</a>. This is such a wonderful mix of two of my favorite things - programming and writing - I sometimes can't believe my good fortune! Shameless plug alert: I really love Stormpath and what we are up to. Check us out.  And, I've begun putting the "fit" back in. I'm beginning to run again, starting with 3 to 5 sessions per week on the elliptical.

I am very easily distracted. I wanted to set myself up for success in my new position. When I use the <a href="http://pomodorotechnique.com/" target="_blank">pomodoro technique</a>, I'm in a much better place.

Recently, I started using a combination of Google Chrome "people" (personas) and killing apps to try to keep distractions away while working in a pomodoro. I created a persona called "Distractions". Here's what the process of getting rid of distractions looks like for me:

<ol>
	<li>Close the "Distractions" Google Chrome window</li>
	<li>Quit a number of apps that are distracting when I am trying to work</li>
        <ul>
       	        <li><a href="http://youneedabudget.com" target="_blank">YNAB</a> - awesome budgeting software.</li>
                <li><a href="https://calendar.sunrise.am/" target="_blank">Sunrise Calendar</a> - awesome calendar app.</li>
                <li><a href="https://www.omnigroup.com/omnifocus" target="_blank">Omnifocus</a> - essential project and task management.</li>
                <li><a href="https://telegram.org/" target="_blank">Telegram</a> - great individual and group messaging app.</li>
                <li><a href="https://www.apple.com/support/mac-apps/messages/" target="_blank">Messages</a> - Apple messaging app that allows me to send SMS messages from the desktop.</li>
                <li><a href="https://www.hipchat.com/" target="_blank">Hipchat</a> - awesome messaging app for work. Annoyingly, disregards "Do Not Disturb" setting on Mac.</li>
         </ul>
	<li>Turn on Do Not Disturb mode</li>
</ol>

During the pomodoro break, I reverse the process.

On top of that, I like to have certain apps on certain <a href="https://support.apple.com/kb/PH18757?locale=en_US" target="_blank">spaces</a> on my two-display setup. This meant CMD+Tab to the app or jump to the space, and then start it up (or kill it, depending on where I am at). With only 5 minutes in a pomodoro break, all this switching around and launching can add up.

This post is all about how I automated this process using an application that's been in OS X for a very long time: <a href="https://support.apple.com/en-ph/HT2488" target="_blank">Automator</a>. Atomator's been built-in since the <a href="https://en.wikipedia.org/wiki/Mac_OS_X_Tiger" target="_blank">Tiger</a> release in 2005! And, I've completely neglected it even though I've been a full-time OS X user since 2008. Full disclosure here: I am an Automator and Applescript n00b. The Applescript examples that I am going to share with you work on my setup running Yosemite. YMMV!

There were a couple of challenges to get this working the way I wanted. First, I had to be able to launch certain apps on certain spaces. I also needed a way to open and close certain Google Chrome personas without exiting Chrome altogether.

Let's take a look at the close distractions script first:

{% highlight bash linenos %}
on run {input, parameters}

    tell application "Messages"
    	quit
    end tell

    tell application "HipChat"
    	quit
    end tell

    tell application "Telegram"
    	quit
    end

    tell application "YNAB"
    	quit
    end

    tell application "Sunrise"
    	quit
    end

    tell application "Omnifocus"
    	quit
    end

    tell application "Google Chrome"
    	activate
    	tell application "System Events"
    		tell process "Google Chrome"
    			click menu item "Distractions" of menu 1 of menu bar item "People" of menu bar 1
    			click menu item "Close Window" of menu 1 of menu bar item "File" of menu bar 1
    		end tell
    	end tell
    end tell

	return input
end run
{% endhighlight %}

Lines 3 - 25 are pretty self explanatory. For each named application, tell that application to quit. There were a couple of applications that Automator didn't know about even though they were running. I probably just didn't have the name exactly right. Automator gave me a picker to select the app and once I did that, the proper app would quit when I ran the script. Some secret sauce going on behind the scenes there.

Lines 27 - 35 are where things get interesting. In this case, I am activating the Google Chrome application and then interacting with the menu to accomplish the goal of closing the "Distractions" persona.

<a href="/images/2015/08/close_distractions.gif" target="_blank"><img src="/images/2015/08/close_distractions.gif"></a>

After testing this script in the Applescript editor, I was ready to create an Automator script and bind a key sequence to it. In the Automator app, I chose to create a new workflow of the "Service" type. I then dragged over the "Run Applescript" item from the library on the left. In addition to pasting in the above script, I selected some key settings that make the keyboard shortcut work the way I want it to.

<a href="/images/2015/08/close_distractions.png" target="_blank"><img src="/images/2015/08/close_distractions.png"></a>

Toward the top of the pane, there are two dropdowns. This service automation does not receive any input AND I want it to be available in "any application".

I always wondered what the universal menu item "Services" that I see on all mac apps was for. Now I know - a little at least. Once I saved the automation, the workflow I had created showed up in the services menu of (nearly) any app I run.

<a href="/images/2015/08/services.png" target="_blank"><img src="/images/2015/08/services.png"></a>

Having these workflows available in any application is critical to being able to being able to trigger them by a key sequence. In the Keyboard System Preference pane, there's a Shortcuts tab. On that tab, you can bind key sequences to variety of types, including Services.

<a href="/images/2015/08/keyboard_shortcuts.png" target="_blank"><img src="/images/2015/08/keyboard_shortcuts.png"></a>

I bound the "close_distractions" workflow to the key sequence:

{% highlight bash %}
CTRL+ALT+CMD+,
{% endhighlight %}

I bound the "open_distractions" workflow to the key sequence:

{% highlight bash %}
CTRL+ALT+CMD+.
{% endhighlight %}

Let's take a look at the "open distractions" workflow code. Again, some of this is very specific to my setup. YMMV.

{% highlight bash linenos %}
on run {input, parameters}

    do shell script "/Applications/Mission\\ Control.app/Contents/MacOS/Mission\\ Control"
    delay 0.5
    tell application "System Events" to click (first button whose value of attribute "AXDescription" is "exit to Desktop 4") of list 2 of group 1 of process "Dock"
    delay 0.5

    tell application "Google Chrome"
    	activate
    	tell application "System Events"
    		tell process "Google Chrome"
    			click menu item "Distractions" of menu 1 of menu bar item "People" of menu bar 1
    		end tell
    	end tell
    end tell

    do shell script "/Applications/Mission\\ Control.app/Contents/MacOS/Mission\\ Control"
    delay 0.5
    tell application "System Events" to click (first button whose value of attribute "AXDescription" is "exit to Desktop 5") of list 2 of group 1 of process "Dock"
    delay 0.5

    tell application "Messages"
    	activate
    end tell

    tell application "HipChat"
    	activate
    end tell

    tell application "Telegram"
    	activate
    end tell


    do shell script "/Applications/Mission\\ Control.app/Contents/MacOS/Mission\\ Control"
    delay 0.5
    tell application "System Events" to click (first button whose value of attribute "AXDescription" is "exit to Desktop 1") of list 1 of group 1 of process "Dock"
    delay 0.5

    tell application "Omnifocus"
    	activate
    end tell

    tell application "YNAB"
    	activate
    end tell

    tell application "Sunrise"
    	activate
    end tell

	return input
end run
{% endhighlight %}

Before I break down this script, let me tell you a little bit about my setup. I have dual 27" HDMI displays. The first display has 3 spaces and the second display has 2 spaces. On Desktop 1, I keep all my "productivity" apps. (The quotes are because I am very easily distracted by these apps if they are always running). These include apps like YNAB, Sunrise Calendar, and Omnifocus. Desktop 2 is my workspace. I usually have <a href="https://www.iterm2.com/" target="_blank">iTerm2</a> and <a href="https://www.jetbrains.com/idea/" target="_blank">IntelliJ</a> running on this desktop. Desktop 3 is a scratch space for apps I fire up and tear down quickly. Desktop 4 is for my browsers. And Desktop 5 is for my messaging apps, like Messages, Telegram and Hipchat.

<a href="/images/2015/08/desktop_1.png" target="_blank"><img src="/images/2015/08/desktop_1.png"></a>

<a href="/images/2015/08/desktop_2.png" target="_blank"><img src="/images/2015/08/desktop_2.png"></a>

Part of what I wanted to accomplish in this automation was having the apps launched on the correct space. Lines 3 - 6 in the code block above accomplish this. Line 3 gets us into Mission Control. Line 5 clicks on the appropriate Desktop. In this case, Desktop 4.

Once I am on Desktop 4, I can launch my Distractions Google Chrome persona, which is what lines 8 - 15 does.

Next, I switch to Desktop 5 and launch my messaging apps.

And, finally, I switch to Desktop 1 and launch my productivity apps.

I also bound the key sequence:

{% highlight bash %}
CTRL+ALT+CMD+]
{% endhighlight %}

to toggle Do Not Disturb mode.

Now, just before I start a pomodoro, I hit:

{% highlight bash %}
CTRL+ALT+CMD+,
CTRL+ALT+CMD+]
{% endhighlight %}

And, when I am in the pomodoro break, I hit:

{% highlight bash %}
CTRL+ALT+CMD+.
CTRL+ALT+CMD+]
{% endhighlight %}

During my testing of these scripts, I had to add certain applications to a list of applications allowed to interact with the Accessibility features of OS X. I probably should have kept better track, but it included Automator and Google Chrome.

<a href="/images/2015/08/accessibility.png" target="_blank"><img src="/images/2015/08/accessibility.png"></a>

I started looking into toggling Do Not Disturb mode as part of the key sequence for open and close distractions, but I bailed on that. I also started to look into hooking into <a href="http://www.irradiatedsoftware.com/sizeup/" target="_blank">SizeUp</a> so that I could lay out the messaging windows the way I like as part of the open distractions script, but I bailed on that too. Getting to the point I am at - which is very functional - took about an hour of Googling and experimentation.

Any feedback on how to make it more awesome or to correct any bone-headed things I am doing is much appreciated.
