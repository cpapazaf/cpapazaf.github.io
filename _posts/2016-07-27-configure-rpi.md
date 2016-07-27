---
layout: post
title:  "Configure the Reapberry PI"
date:   2016-07-27 19:00:02 +0100
categories: raspberrypi
---

Use a micro SD card and connect it to your pc (MAC OSX). Check the mount point for the SD card ([rpi-install])

{% highlight bash %}
diskutil list

#=> 
/dev/disk1
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *8.0 GB     disk1
   1:             Windows_FAT_32 boot                    66.1 MB    disk1s1
   2:                      Linux                         7.9 GB     disk1s2
{% endhighlight %}


References:
[rpi-install]: https://www.raspberrypi.org/documentation/installation/installing-images/mac.md
