---
layout: post
title:  "Configure the Raspberry PI"
date:   2016-07-27 19:00:02 +0100
categories: raspberrypi
---

Use a micro SD card and connect it to your pc (MAC OSX). Check the mount point for the SD card [RPI install docs][rpi-install]

{% highlight bash %}
diskutil list

#=> 
/dev/disk1
#:                       TYPE NAME                    SIZE       IDENTIFIER
0:     FDisk_partition_scheme                        *8.0 GB     disk1
1:             Windows_FAT_32 boot                    66.1 MB    disk1s1
2:                      Linux                         7.9 GB     disk1s2
{% endhighlight %}

Unmount the SD card but do not eject it!
{% highlight bash %}
diskutil unmountDisk /dev/disk1
#=> Unmount of all volumes on disk1 was successful
{% endhighlight %}

Install the image (Get the image from [RPI Jessie Lite][rpi-jesie-lite])
{% highlight bash %}
sudo dd bs=1m if=Downloads/2016-05-27-raspbian-jessie-lite.img of=/dev/disk1
{% endhighlight %}

[rpi-install]: https://www.raspberrypi.org/documentation/installation/installing-images/mac.md
[rpi-jesie-lite]: https://www.raspberrypi.org/downloads/raspbian/
