---
layout: post
title:  "Local mirror of compiled python packages with pip wheel and devpi server"
date:   2016-01-24
category: "tech gibberish"
issue_id: 3
---

#### A problem.

```pip install``` can take quite a while when we try to install packages as big as [scikit-learn](http://scikit-learn.org/stable/) or [scipy](http://www.scipy.org/) from source. Especially if you're installing it quite often within virtualenv and not on that powerful vagrant virtual machine.

#### Meet the ```pip wheel```.
[```wheel```](https://wheel.readthedocs.org/en/latest/) is "a built-package format for Python".

This is the replacement for the egg format which never had proper pep specification. Wheel gives us an option to create binary "package" by ourself. If there already is ```.whl``` uploaded to pypi ```pip wheel``` will download and use it.

```pip wheel``` can create binaries and can get them from requirements file with ```-r``` flag.

#### Creating ```.whl``` packages.
I split this into 3 steps:

Create ```virtualenv``` for every ```requirements``` file for which I want to build wheels.

{% highlight bash %}
virtualenv venv
{% endhighlight %}

Install all dependencies from ```requirements``` file.

{% highlight bash %}
source venv/bin/activate
pip install -r requirements
{% endhighlight %}

Create wheels in seperate directory (to add them easily to devpi mirror).

{% highlight bash %}
pip wheel -r requirements.txt -w /home/$USER/wheels/
{% endhighlight %}

#### [```devpi```](http://doc.devpi.net/latest/) installation/configuration
Installation of ```devpi``` is the easiest part:

{% highlight bash %}
pip install devpi-server devpi-client
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

I have created this mirror "behind firewall" so I'm not using any authentication. With this in mind configuration of devpi server was limited to creation of user and new mirror index.

{% highlight bash %}
devpi use http://localhost:3141
devpi user -c $USER password=
devpi login $USER --password=
devpi index -c mirror
{% endhighlight %}

#### Adding ```.whl``` to ```devpi``` server
Earlier in post I wrote about creating seperate directory for wheels. Now we can use it to add packages we build locally to devpi:

{% highlight bash %}
devpi use http://localhost:3141
devpi login $USER --password=
devpi use $USER/mirror
devpi upload --no-vcs --formats=bdist_wheel --from-dir /home/$USER/wheels/
{% endhighlight %}

And now we have our locally available packages which we can use even if the internet (or pypi) is down.

{% highlight bash %}
pip install -i http://localhost:3141/$USER/mirror/ -r requirements
{% endhighlight %}

#### Example on your local computer

{% highlight bash %}
# create venv, directories and install deps
virtualenv venv
mkdir wheels
mkdir mirror
source venv/bin/activate
pip install wheel devpi-server devpi-client -U

# start devpi server
devpi-server --start --serverdir mirror (this one may take a few seconds to start)

# create wheel with deps for time/date management library
pip wheel arrow -w wheels


# configure
devpi use http://localhost:3141
devpi user -c $USER password=
devpi login $USER --password=
devpi index -c mirror

# upload arrow with dependencies
devpi use http://localhost:3141
devpi login $USER --password=
devpi use $USER/mirror
devpi upload --no-vcs --formats=bdist_wheel --from-dir wheels

#test if it works
pip install -i http://localhost:3141/$USER/mirror/ arrow
{% endhighlight %}

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
