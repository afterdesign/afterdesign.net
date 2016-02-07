---
layout: post
title:  "Restart services after mounting NFS folders in Vagrant with udev events"
date:   2015-11-20
category: "tech gibberish"
issue_id: 1
---

Before jumping to scripts/udev there are few things I'm taking "as is" from my vagrant setup and this Vagrantfile example will help you to understand/recreate what I'm doing.

{% highlight ruby %}
Vagrant.configure('2') do |config|
  config.vm.hostname = 'test.com'

  config.vm.network 'private_network', ip: '192.192.1.2' # and the IP of my local computer is 192.192.1.1
  config.vm.synced_folder '.', '/project', type: 'nfs'

  config.ssh.forward_agent = true
end
{% endhighlight %}

When OS is booting up all of the services are starting. I'm using ```php5-fpm``` and ```nginx``` and both of those services require paths to exist for proper running. NFS shares are mounted in vagrant guest os after starting the services.

I'm using very simple script to restart services:

{% highlight bash %}
sleep 5 # wait for a bit for NFS to make sure resources are available
systemctl restart php5-fpm > /dev/null 2>&1
systemctl restart nginx > /dev/null 2>&1
{% endhighlight %}

I'm saving this to ```/root/.udev-mount-restart-services.sh``` file with ansible provisioning.

But to make it work we have to run this script after our directory is mounted over NFS. This is where I used ```udev```.

First of all I wanted to know what event and subsystem is triggered when I mount directory. To get this information I opened 2 terminals and used ```vagrant ssh``` to get to guest os.

In one terminal I started ```udevadm monitor``` to get informations about events that were triggered by mount/umount of /project directory.

In second terminal I got root (with ```sudo su``` in my case), checked what is mounted with ```df -h``` (cause I'm too lazy to type this):

{% highlight bash %}
192.192.1.1:/Users/afterdesign/Projects 233G  186G   47G  80% /project
{% endhighlight %}

So now what I did and what was the effect:

{% highlight bash %}
# Terminal 1:
→ udevadm monitor
monitor will print the received events for:
UDEV - the event which udev sends out after rule processing
KERNEL - the kernel uevent

KERNEL[336.177605] remove   /devices/virtual/bdi/0:37 (bdi)
UDEV  [336.177844] remove   /devices/virtual/bdi/0:37 (bdi)

# Terminal 2:
umount /project
{% endhighlight %}


{% highlight bash %}
# Terminal 1:
mount --source 192.192.1.1:/Users/afterdesign/Projects --target /project/

# Terminal 2 with still running udevadm:
KERNEL[432.289098] add      /devices/virtual/bdi/0:37 (bdi)
UDEV  [438.240571] add      /devices/virtual/bdi/0:37 (bdi)
{% endhighlight %}

With this informations (and a wiki [how to create udev rules](https://wiki.archlinux.org/index.php/Udev#Writing_udev_rules)) I could finally create rule and make it run the script to restart nginx and php5-fpm:

{% highlight bash %}
SUBSYSTEM=="bdi",ACTION=="add",RUN+="/bin/bash /root/.udev-mount-restart-services.sh"
{% endhighlight %}

With provisioning I'm putting this script in ```/root/.udev-mount-restart-services.sh``` and udev rule in ```/etc/udev/rules.d/50-vagrant-mount.rules```.

After ```vagrant up``` the default vagrant user has exactly the same uid and gid as my local user. No more problems with reading/writing in ```/project```.

There are also prerequisits for this method:
1. udev event is not started after getting machine up after ```vagrant suspend```
2. if you're provisioning this within ```Vagrantfile``` you might need to do additional ```vagrant reload``` to make it work after provisioning (not sure because I'm using [packer](https://packer.io/) to build our own boxes with preinstalled scripts like this)
