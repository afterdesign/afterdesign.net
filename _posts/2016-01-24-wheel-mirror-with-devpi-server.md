---
layout: post
title:  "Local mirror of compiled python packages with pip wheel and devpi server"
date:   2016-01-24
category: "tech gibberish"
issue_id: 3
---

This post is updated on 3rd of March 2016. If you would like to see what changed check [github commit history](https://github.com/afterdesign/afterdesign.net)

#### A problem.

```pip install``` can take quite a while when we try to install packages as big as [scikit-learn](http://scikit-learn.org/stable/) or [scipy](http://www.scipy.org/) from source. Especially if you're installing it quite often within virtualenv and not on that powerful vagrant virtual machine.

#### Meet the ```wheel```.
[```wheel```](https://wheel.readthedocs.org/en/latest/) is "a built-package format for Python".

This is the replacement for the egg format which never had proper pep specification. Wheel gives us an option to create binary "package" by ourself. If there already is ```.whl``` uploaded to pypi ```pip wheel``` will download and use it.

```pip wheel``` can create binaries and with help of ```devpi-builder``` it's easy to create local mirror of compiled packages.

#### Using ```devpi-builder``` to build packages.

Create ```virtualenv``` for every ```requirements.txt``` file for which I want to build wheels.

{% highlight bash %}
virtualenv venv
{% endhighlight %}

Install all dependencies from ```requirements.txt``` file

{% highlight bash %}
source venv/bin/activate
pip install -r requirements.txt
{% endhighlight %}

I'm doing this step just as précaution because for example ```scikit-learn``` needs ```numpy``` installed to build but ```numpy``` is not added as dependency.

#### [```devpi```](http://doc.devpi.net/latest/) installation/configuration
Installation of ```devpi``` is the easiest part:

{% highlight bash %}
pip install devpi-server devpi-client devpi-web devpi-builder
{% endhighlight %}

To make it easier to start/stop service I also created user in the system called devpi and enabled ```systemd``` to run processess for users even if they logged out.

The ```.service``` file looks like this:

{% highlight bash %}
[Unit]
Description=local devpi mirror
After=network.target

[Service]
Type=forking
PIDFile=/var/run/devpi-server.PID
Restart=always
ExecStart=/usr/local/bin/devpi-server --start --serverdir /home/$USER/mirror/
ExecStop=/usr/local/bin/devpi-server --stop --serverdir /home/$USER/mirror/

[Install]
WantedBy=multi-user.target
{% endhighlight %}

File was copied to ```/home/$USER/.config/systemd/user/devpi.service``` and the service was started with

{% highlight bash %}
systemctl --user enable devpi.service
systemctl --user start devpi.service
{% endhighlight %}

Now it's time to configure our local mirror.

{% highlight bash %}
devpi use http://localhost:3141
devpi user -c $USER password=TEST_PASSWORD
devpi login $USER --password=TEST_PASSWORD
devpi index -c mirror
{% endhighlight %}

#### Building and uploading packages to devpi server

{% highlight bash %}
source venv/bin/activate
devpi-builder requirements.txt http://localhost:3141/$USER/mirror $USER TEST_PASSWORD
{% endhighlight %}

And now we have our locally available packages which we can use even if the internet (or pypi) is down.

{% highlight bash %}
pip install -i http://localhost:3141/$USER/mirror/ -r requirements.txt
{% endhighlight %}

#### Demo/example.

From now you can go to [devpi-setup-example](https://github.com/afterdesign/devpi-setup-example) repository and test it with vagrant or just see scripts with simple devpi installation.

#### Global pip configuration with fallback to pypi
And the last thing I wanted to do is to create pip config which can be used everywhere.
When the local mirror is not available/dead/broken it's going to download packages from pypi.

{% highlight bash %}
[global]
index-url = http://localhost:3141/$USER/mirror/
extra-index-url = https://pypi.python.org/simple/
retries = 0
{% endhighlight %}

Where to put this ```pip.conf``` you can find on [pip documentation page](https://pip.pypa.io/en/stable/user_guide/#configuration)

#### Problems

I'm using this method to build packages for python2 and python3.
It takes about 3 hours right now for me so this is not the most efficient way.

This is why I'm going to write script taking all the dependencies across repositories, checks versions and builds all the things I need.
It's going to be totally for my use case but I think I'll opensource it and maybe hack a little around pip APIs.
