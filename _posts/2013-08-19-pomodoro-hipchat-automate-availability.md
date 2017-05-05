---
layout: post
title:  "Pomodoro & HipChat: Automate Availability"
date:   2013-08-19 02:33:41 -0400
categories: [tech, productivity]
tags: [tech, productivity]
---

The <a href="http://www.pomodorotechnique.com/" target="_blank">Pomodoro Technique</a> is a way to enhance focus while working. There is a high cost in brainpower to stop doing what you are focused on, look at the tweet that just came in, and then try to start up where you left off. The idea of working in "pomodoros" is that you work uninterrupted for 25 minutes. Then you get a 5 minute break. After you do 4 of those cycles, you get a 10 minute break. Speaking from my own experience, I get a lot more done in 4 hours of pomodoros than 8 hours of work disrupted by various notifications.

At <a href="https://www.workmarket.com" target="_blank">Work Market</a>, we rely on <a href="https://hipchat.com" target="_blank">HipChat</a> to communicate with each other throughout the day. This is a very handy way to connect our two offices and the various remote workers that we may have at any given time. However, when I am in a pomodoro, I don't want to be disturbed by HipChat.

Following the excellent post <a href="http://ertw.com/blog/2012/05/02/controlling-hipchat-status-through-applescript/" target="_blank">here</a>, I am able to automatically set my status to "Do Not Disturb" when I start a pomodoro and back to "Available" when the pomodoro is done.

I've taken it one step further by sending a notification to the main room we all use in HipChat to let my coworkers know what I am up to. This is a little more communicative than just having my status abruptly change with no other context.

The key is making use of the HipChat <a href="https://www.hipchat.com/docs/api" target="_blank">API</a> in conjunction with AppleScript.

Firstly, I created a shell script named HipChat.Pomodoro.sh that takes a message as an argument and sends it to the specified HipChat room using the API:
{% highlight bash linenos %}
#! /bin/bash
ROOM_ID=[HipChat Room ID]
AUTH_TOKEN=[HipChat Auth Token]
MESSAGE=$1
curl -s \
  -d "room_id=$ROOM_ID&from=Pomodoro&notify=1&color=yellow&message_format=text" \
  --data-urlencode "message=(pomodoro) $MESSAGE" \
  https://api.hipchat.com/v1/rooms/message?auth_token=$AUTH_TOKEN > /dev/null
{% endhighlight %}

You'll need to put in your own room id and auth token from HipChat.

You can test this right from the command line:

{% highlight bash %}
./HipChat.Pomodoro.sh "Test Message"
{% endhighlight %}

Next, I updated the settings in the <a href="https://github.com/ugol/pomodoro" target="_blank">Pomodoro</a> app preferences to make use of the script:

<a href="/images/2013/08/Screen-Shot-2013-08-19-at-1.57.16-AM.png" target="_blank"><img src="/images/2013/08/Screen-Shot-2013-08-19-at-1.57.16-AM-300x282.png" alt="" title="Screen Shot 2013-08-19 at 1.57.16 AM" width="300" height="282" class="aligncenter size-medium wp-image-949" /></a>

I added the call to my script after the AppleScript from Sean's original post. I have the Start, Interrupt Over, Reset and End events handled

Start:
{% highlight bash linenos %}
tell application "System Events" to tell UI element "HipChat" of list 1 of process "Dock"
  perform action "AXShowMenu"
  delay 0.5
  click menu item "Status" of menu 1
  click menu item "Do Not Disturb" of menu 1 of menu item "Status" of menu 1
end tell
do shell script "/Users/dogeared/Hipchat.Pomodoro.sh 'Micah is starting a Pomodoro. He will be back in 25 minutes.'"
{% endhighlight %}

Interrupt Over:
{% highlight bash linenos %}
tell application "System Events" to tell UI element "HipChat" of list 1 of process "Dock"
  perform action "AXShowMenu"
  delay 0.5
  click menu item "Status" of menu 1
  click menu item "Available" of menu 1 of menu item "Status" of menu 1
end tell
do shell script "/Users/dogeared/HipChat.Pomodoro.sh 'Too many distractions. (pomodoro) fail for Micah!'"
{% endhighlight %}

Reset:
{% highlight bash linenos %}
tell application "System Events" to tell UI element "HipChat" of list 1 of process "Dock"
  perform action "AXShowMenu"
  delay 0.5
  click menu item "Status" of menu 1
  click menu item "Available" of menu 1 of menu item "Status" of menu 1
end tell
do shell script "/Users/dogeared/HipChat.Pomodoro.sh 'Micah bailed! (timeforthat)'"
{% endhighlight %}

End:
{% highlight bash linenos %}
tell application "System Events" to tell UI element "HipChat" of list 1 of process "Dock"
  perform action "AXShowMenu"
  delay 0.5
  click menu item "Status" of menu 1
  click menu item "Available" of menu 1 of menu item "Status" of menu 1
end tell
do shell script "/Users/dogeared/HipChat.Pomodoro.sh 'And... Micah is back! (thumbsup)'"
{% endhighlight %}

Note: The Pomodoro app's script tab in preferences is finicky. The standard copy-and-paste shortcut keys do not work, but right-clicking and choosing copy or paste does work.

You will need to enable access for assistive devices in System Preferences -> Accessibility in order for the Pomodoro automation to work.

<a href="/images/2013/08/Screen-Shot-2013-08-19-at-9.21.39-AM.png"><img src="/images/2013/08/Screen-Shot-2013-08-19-at-9.21.39-AM.png" alt="" title="Screen Shot 2013-08-19 at 9.21.39 AM" width="668" height="476" class="aligncenter size-full wp-image-978" /></a>

Here are some screenshots from HipChat. In the first one, I started a pomodoro and reset in the middle of it. The second one shows a complete pomodoro.

<a href="/images/2013/08/Screen-Shot-2013-08-19-at-2.10.27-AM1.png" target="_blank"><img src="/images/2013/08/Screen-Shot-2013-08-19-at-2.10.27-AM1-1024x52.png" alt="" title="Screen Shot 2013-08-19 at 2.10.27 AM" width="1024" height="52" class="aligncenter size-large wp-image-969" /></a>

<a href="/images/2013/08/Screen-Shot-2013-08-19-at-2.10.42-AM1.png" target="_blank"><img src="/images/2013/08/Screen-Shot-2013-08-19-at-2.10.42-AM1-1024x48.png" alt="" title="Screen Shot 2013-08-19 at 2.10.42 AM" width="1024" height="48" class="aligncenter size-large wp-image-970" /></a>
