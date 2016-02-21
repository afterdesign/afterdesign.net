---
layout: post
title:  "Connecting to headless Xvfb session running selenium tests with VNC"
date:   2016-02-21
category: "tech gibberish"
issue_id: 5
---
Last time I wrote how to [record selenium testing to mp4 with ffmpeg.]("/2016/02/07/recording-headless-selenium-tests-to-mp4.html").
This time I would like to show how to connect to Xvfb and watch selenium testing live.

So as I wrote earlier we are using multiple builds to get all tests in under 10 minutes instead of hour+. Unfortunately selenium is not the most stable software available and sometimes after upgrade to new version there are "problems". Without option to debug them by clicking in the browser it's just pain in the ass to debug and get necessary informations what is wrong.

Fortunately it is easy to get working VNC server and with it connecting and debugging "live" is easy as doing it on your own computer.

First you have to run selenium server inside Xvfb. Make it listen for tcp connection and save the auth-file for easier connection:
{% highlight bash %}
xvfb-run --listen-tcp --server-num 44 --auth-file /tmp/xvfb.auth -s "-ac -screen 0 1920x1080x24"
{% endhighlight %}

Then install [x11vnc](http://www.karlrunge.com/x11vnc/) and create password file:
{% highlight bash %}
x11vnc -storepasswd YOUT_PASSWORD /tmp/vncpass
{% endhighlight %}

And the last step is to run x11vnc server:
{% highlight bash %}
x11vnc -rfbport 4544 -rfbauth /tmp/vncpass -display :44 -forever -auth /tmp/xvfb.auth
{% endhighlight %}

If you want to automate it just create systemd service file/template and that's all.
Now you can easily connect to Xvfb and play with browser running tests orchestrated by selenium.
