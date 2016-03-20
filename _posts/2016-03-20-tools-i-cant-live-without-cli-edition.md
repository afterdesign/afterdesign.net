---
layout: post
title:  "Tools I can't live without - CLI edition"
date:   2016-03-20
category: "tooling rant"
issue_id: 6
---

When I was writing about my pain with [OSX reinstallation](http://afterdesign.net/2016/03/07/os-x-reinstallation-pain.html) I had also idea to write a list of tools that I'm using and that I really like (or even love). So let's do this !

This time I'm going to list CLI tools.

# Command Line Tools installed with [homebrew](http://brew.sh)
"Homebrew installs the stuff you need that Apple didn’t."
In general this is package manager. I like it because I don't have to install packages with root privileges.

##### [autojump](https://github.com/wting/autojump)
I have more then 50 repositories cloned to my ```~/Projects``` directory so using ```cd``` is a bit painfull. With autojump I can change directory even when I remember part of the path.
![autojump screenshot](/images/posts/6/autojump.png)

##### [htop](http://hisham.hm/htop/)
I know there is a activity monitor app but I really like htop and it looks cool with colors, options to sort or even see tree of threads.
![htop screenshot](/images/posts/6/htop.png)

##### [icdiff](https://github.com/jeffkaufman/icdiff)
The best diff tool I know.
I'm using it always before I commit something (with this simple [bash script](https://github.com/afterdesign/dotfiles/blob/master/bin/git-diff-add)).
I'm using icdiff also as my git difftool:

{% highlight bash %}
[difftool]
    prompt = false
[difftool "icdiff"]
    cmd = icdiff $LOCAL $REMOTE | less -R
{% endhighlight %}

![icdiff screenshot](/images/posts/6/icdiff.png)

##### [ncdu](https://dev.yorhel.nl/ncdu)

If you're like me sometimes you're trying to figure out "why the hell all my disk space is gon".
Instead of using paid tools like [daisydisk](https://daisydiskapp.com) I'm using this free, open and awesome tool

![ncdu screenshot](/images/posts/6/ncdu.png)

##### [jq](https://stedolan.github.io/jq/)

Pretty print json files.

![jq screenshot](/images/posts/6/jq.png)

##### [pngquant](https://pngquant.org)

When I'm adding screenshots (like in this post) then I'm using pngquant. It can really crunch image size without much quality loss (hell, I can't even see the quality difference)

![ncdu screenshot](/images/posts/6/pngquant.png)

##### [nano](http://www.nano-editor.org)

Yup I'm one of those people that are not using vi/vim.
And to install newest nano you have to add [homebrew/dupes](https://github.com/Homebrew/homebrew-dupes) with ```brew tap homebrew/dupes```

![nano screenshot](/images/posts/6/nano.png)

##### [mobile-shell](https://mosh.mit.edu)
I'm using this to connect with my personal servers. I'm still using IRC and my connection at home is a bit flaky lately. Now I know that I lost connection to my server (and irssi) and the connection is reestablished right after the internet is back.

<img src="/images/posts/6/mosh.png" alt="mobile-shell screenshot" style="width:477px;">

##### [tmux-cssh](https://github.com/dennishafemann/tmux-cssh) and/or [csshx](http://code.google.com/p/csshx)

When I want to check something on multiple servers I'm just using one of those tools.
Depends if I want to have multiple windows or all in one window.

##### [bash](https://www.gnu.org/software/bash/) with [bash-completion2](https://github.com/scop/bash-completion)

Yes I'm using bash and I like it, also OSX by default is comming with 3.2 and one of first things I do is upgrade bash with brew to newest version

##### [git](http://git-scm.com) with [git-extras](https://github.com/tj/git-extras) and [git-number](https://github.com/holygeek/git-number)
I'm a programmer and I really like git

##### [multitail](https://www.vanheusden.com/multitail/)

Tail multiple files with nice ncurses windows.

##### [chromedriver](https://sites.google.com/a/chromium.org/chromedriver/) with [selenium-server-standalone](http://www.seleniumhq.org/download/)

I'm using this to locally run tests from behat/behave.
Or just to run [my project](https://github.com/afterdesign/unsuck-ads) to now how much faster sites are without ads.


# Command Line Tools installed manually from pkg/dmg

##### [vagrant](http://vagrantup.com)
Every project I'm starting is tested and run within debian stable. To create this environment I'm using virtualbox and to automate this I'm using vagrant. It's just easier for me to do ```vagrant up``` and have once configured OS running.

![vagrant screenshot](/images/posts/6/vagrant.png)

##### [packer](http://packer.io)

I'm using this tool to create custom boxes to use with vagrant. It's easier to create box with already installed databases cause installing it once when creating box is much simpler and faster then installing it with


# Command Line Tools installed with [npm](https://www.npmjs.com)
Npm is installed with [nodejs](http://nodejs.org) which I'm installing with homebrew.

##### [tldr](https://github.com/tldr-pages/tldr-node-client)
I don't like to search man pages all the time, this tool gives me all I need to know from man in shorter version (especially often used ```tldr ln```)

![tldr screenshot](/images/posts/6/tldr.png)


# Command Line Tools installed with [pip](https://pypi.python.org/pypi/pip)
Pip can be installed with [python](https://python.org) or with ```easy_install```.
I'm always installing newest python version from homebrew.

##### [HTTPie](https://github.com/jkbrzt/httpie)
The quote from the website is saying everything: "HTTPie (pronounced aitch-tee-tee-pie) is a command line HTTP client. Its goal is to make CLI interaction with web services as human-friendly as possible."

![httpie screenshot](/images/posts/6/httpie.png)
