---
layout: post
title:  "Hacking vagrant nfs rights problems with udev events"
date:   2015-11-19 09:27 +0100
category: "tech gibberish"
---
### A problem.

If you google for [vagrant nfs rights](https://www.google.pl/search?q=vagrant+nfs+rights) or something similar you'll find a ton of posts about problems with editing/writing files in vagrant guest OS.

I had similar problems while installing [karma runner](http://karma-runner.github.io/) from npm.
One of the dependencies is [bufferutil](https://github.com/websockets/bufferutil) which has to be compiled with help of [node-gyp](https://github.com/nodejs/node-gyp).

node-gyp is using a Makefile in which there is a copy command with "preserve rights" flag. And that ```cp -a``` is failing when uid of vagrant user is different then uid of files/directories mounted over NFS. And by default the uid/gid of files in vagrant are mapped from your local machine.

### Under the hood.

I'm using debian as guest os in vagrant and the uid/gid for created vagrant user is 1000 (you can check this with simple ```id``` command after ```vagrant ssh```). On my desktop I'm running OSX and the uid/gid is quite different:

```bash
→ id
uid=501(afterdesign) gid=20(staff) groups=20(staff),12(everyone) ...
```



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


```bash
SUBSYSTEM=="bdi",ACTION=="add",RUN+="/bin/bash /root/.udev-mount-restart-services.sh"
```
