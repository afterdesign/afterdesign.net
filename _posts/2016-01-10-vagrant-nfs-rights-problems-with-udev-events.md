---
layout: post
title:  "Hacking vagrant nfs rights problems with udev events"
date:   2016-01-10
category: "tech gibberish"
---
First things first in this post I'm using a lot of informations from my previous post about [restarting services after mounting with help of udev events](http://afterdesign.net/2015/11/20/vagrant-nfs-restarting-services-after-mount.html)

### A problem.

If you google for [vagrant nfs rights](https://www.google.pl/search?q=vagrant+nfs+rights) or something similar you'll find a ton of posts about problems with editing/writing files in vagrant guest OS.

I had similar problems while installing [karma runner](http://karma-runner.github.io/) from npm.
One of the dependencies is [bufferutil](https://github.com/websockets/bufferutil) which has to be compiled with help of [node-gyp](https://github.com/nodejs/node-gyp).

**node-gyp** is using a Makefile in which there is a copy command with "preserve rights" flag. And that ```cp -a``` is failing when uid of vagrant user is different then uid of files/directories mounted over NFS. And by default the uid/gid of files in vagrant are mapped from your local machine.

### Under the hood.

I'm using debian as guest os in vagrant and the uid/gid for created vagrant user is 1000 (you can check this with simple ```id``` command after ```vagrant ssh```). On my desktop I'm running OSX and the uid/gid is quite different:

```bash
→ id
uid=501(afterdesign) gid=20(staff) groups=20(staff),12(everyone) ...
```

So with help of udev events I've extended bash script to change UID and GID of vagrant user.
We can do this simply with using usermod and getting the user UID/GID over NFS.

To get GID/UID I'm using simple stat commands.
The only prerequisit here is that NFS is mounted (and it has to be to run some kind of provisioning) and there is ```Vagrantfile```:

```bash
stat -c '%u' /project/Vagrantfile
stat -c '%g' /project/Vagrantfile
```

With that we can go full mental and use that usermod:

```bash
usermod -u $(stat -c '%u' /project/Vagrantfile) vagrant
usermod -g $(stat -c '%g' /project/Vagrantfile) vagrant
```

There might be a problem with ```systemd``` process when you try to use usermod so there is always a good idea to kill systemd process for vagrant user and then restart ```systemd-user-sessions``` service.

So in the end I've created the script:

```bash
if [[ ! -d /home/vagrant ]]; then
    exit 0;
fi

if [[ ! -a /project/Vagrantfile ]]; then
    exit 0;
fi

pkill -u vagrant -9 systemd

if [[ $(stat -c '%u' /project/Vagrantfile) != $(stat -c '%u' /home/vagrant) ]]; then
    usermod -u $(stat -c '%u' /project/Vagrantfile) vagrant
fi

if [[ $(stat -c '%g' /project/Vagrantfile) != $(stat -c '%g' /home/vagrant) ]]; then
    usermod -g $(stat -c '%g' /project/Vagrantfile) vagrant
fi

systemctl restart systemd-user-sessions.service
```

It has few conditions.
Some of them are for the process of the box [creation](http://packer.io), some to make sure it's not going to fail with nasty exit code and some are just to make changes once not every ```vagrant reload/halt/up```
