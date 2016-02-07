---
layout: post
title:  "Recording headless selenium tests to mp4 with Xvfb and ffmpeg"
date:   2016-02-07
category: "tech gibberish"
issue_id: 4
---
We are using selenium (with help of [behat](http://docs.behat.org/en/v3.0/)) quite a lot.
When test failed we had problem to debug it. And running all tests on our local machines is taking about 60min. We have parallel test suites in jenkins so it takes less than 10min so we are using jenkins to run tests for every commit we push to our git repository.
First we were taking screenshots of every step but Behat v3.0 doesn't have option to run step within step and writing each step in gherkin is not the way we want to pursue, we would like to write more business focused tests.

So when my friend started reading "[Continuous Delivery: Reliable Software Releases through Build, Test, and Deployment Automation](http://www.amazon.com/Continuous-Delivery-Deployment-Automation-Addison-Wesley/dp/0321601912)" book he read about recording selenium sessions with [pyvnc2swf](http://www.unixuser.org/~euske/vnc2swf/pyvnc2swf.html) (or something similar).

The idea was so simple and genious that I started to look for options to record sessions which would save us time with debugging problems.
First I tried to use [pyvnc2swf](http://www.unixuser.org/~euske/vnc2swf/pyvnc2swf.html) but this is quite old software. So I started to read about options of Xvfb.

My first dicovery was that Xvfb can listen for remote connections. So I could easily start Xvfb running selenium:

{% highlight bash %}
xvfb-run --listen-tcp --server-num 44 --auth-file /tmp/xvfb.auth -s "-ac -screen 0 1920x1080x24" java -jar selenium.jar
{% endhighlight %}

```--server-num``` is the option to set what is the "display id" which is used in next step.


With this I started to search for informations "how to record X sessions with ffmpeg". And to my surprise [ffmpeg](https://www.ffmpeg.org) has the ```x11grab``` option. And I was even more surprised when it started to do this over IP. So now I can start to record when job with behat is running. Some additional switches and stuff and I had movie in 12fps (cause this is not a movie and I don't need silky smooth fps rate).

{% highlight bash %}
ffmpeg -f x11grab -video_size 1920x1080 -i 127.0.0.1:44 -codec:v libx264 -r 12 /tmp/behat_1.mp4
{% endhighlight %}

The ```-i``` flag is setting up which display to use from which server. I have 6 builds running in paralel and they're using different displays to see what is happening.

The last problem I had was with properly quitting ffmpeg when the recording was done.
```kill``` command would kill the process but the information about video length wasn't correctly saved to container. This made seeking through video problematic.
To quit ```ffmpeg``` properly I had to send ```q``` keystroke to ffmpeg process.

This is the place where [```tmux```](https://tmux.github.io) saves the day.
I had to start detached session with ```tmux``` running ```ffmpeg``` and name session so I knew where to send the keystroke when the build was ending.

So when the build is starting I'm starting tmux with ffmpeg recording

{% highlight bash %}
tmux new-session -d -s BehatRecording1 'ffmpeg -f x11grab -video_size 1920x1080 -i 192.168.102.1:44 -codec:v libx264 -r 12 /tmp/behat_1.mp4'
{% endhighlight %}

And when the build is finishing (no matter if tests are succesfull or not) I'm sending ```q``` to that session with:

{% highlight bash %}
tmux send-keys -t BehatRecording1 q
{% endhighlight %}
