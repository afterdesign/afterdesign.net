---
layout: post
title:  "OS X reinstallation pain"
date:   2016-03-07
category: "tech rant"
---
My laptop is over 2 years old now (macbook pro 13" late 2013).
I'm one of those people who upgrade to new OS version right after it's release (sometimes even beta version). So it wasn't shocker when I upgraded from 10.10 to 10.11 on day 1.

Unfortunately osx is not the best system handling upgrade of major versions. So after 10.11.3 I started to have problems with Dock and all over the places. Also caches grow big (to 20GB) and I just didn't want to clean manually. It was time to reinstall OS.

So after downloading OSX from app store I have created bootable USB drive with help of [DiskMaker X](http://diskmakerx.com). During download I also backuped up my files and updated [dotfiles](https://github.com/afterdesign/dotfiles).

Next step is to run installer, erase disk and install fresh OS.
Next, setup time - wifi connectivity, apple login, whole disk encryption with filevault aaaaand installation of Xcode (which took about 1.5 hour), [brew](http://brew.sh), [ansible](https://www.ansible.com) and [dropbox](https://www.dropbox.com) finally I could start my automation.

Right now I have [ansible playbook](https://github.com/afterdesign/dotfiles/blob/master/darwin/playbook.yml) from within I'm installing all of my cli tools and setting up my configurations.

This was the moment I remembered that reinstalling osx is pain in the ass for me.
Automation can be done on some level but the first 2-3 hours is really painfull.
So after this experience I realized I need to update my flow to make it work for itself right after Xcode is installed.

Looks like I can move my automation a notch with [brew cask](https://github.com/caskroom/homebrew-cask) and [mas](https://github.com/argon/mas) so hopefully next reinstall will be much quicker.

Still I find this experience a bit "windows like". As much as I like osx I still think that Linux repositories are so much better to make it all work from scratch.
