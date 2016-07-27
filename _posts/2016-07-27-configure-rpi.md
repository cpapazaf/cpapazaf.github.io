---
layout: post
title:  "Configuring the Raspberry PI"
date:   2016-07-27 19:00:02 +0100
categories: raspberrypi
---

Use a micro SD card and connect it to your pc (MAC OSX). Check the mount point for the SD card [RPI install docs][rpi-install]

{% highlight bash %}
diskutil list

#=> 
#/dev/disk1
#:                       TYPE NAME                    SIZE       IDENTIFIER
#0:     FDisk_partition_scheme                        *8.0 GB     disk1
#1:             Windows_FAT_32 boot                    66.1 MB    disk1s1
#2:                      Linux                         7.9 GB     disk1s2
{% endhighlight %}

Unmount the SD card but do not eject it!
{% highlight bash %}
diskutil unmountDisk /dev/disk1
#=> Unmount of all volumes on disk1 was successful
{% endhighlight %}

Install the image (Get the image for [RPI Jessie Lite][rpi-jesie-lite])
{% highlight bash %}
sudo dd bs=1m if=Downloads/2016-05-27-raspbian-jessie-lite.img of=/dev/disk1
#1323+0 records in
#1323+0 records out
#1387266048 bytes transferred in 1061.956414 secs (1306330 bytes/sec)
{% endhighlight %}

Boot up the raspberry PI with the SD card and connect the ethernet cable to your home router. Find the IP assigned to the RPI:
{% highlight bash %}
nmap -sP 192.168.0.1/24
{% endhighlight %}

SSH to your RPI by using the default uname/passwd
{% highlight bash %}
ssh pi@192.168.0.? 
#=> password: raspberry
{% endhighlight %}

Extend the RPI filesystem to cover the whole SD card, and reboot
{% highlight bash %}
sudo raspi-config
{% endhighlight %}

If you have a wifi dongle attached to the RPI [then][rpi-connect-wifi]
{% highlight bash %}
sudo iwlist wlan0 scan
sudo vi /etc/wpa_supplicant/wpa_supplicant.conf
{% endhighlight %}

And add the following lines at the bottom
{% highlight bash %}
network={
    ssid="The_ESSID_from_earlier"
    psk="Your_wifi_password"
}
{% endhighlight %}

[rpi-install]: https://www.raspberrypi.org/documentation/installation/installing-images/mac.md
[rpi-jesie-lite]: https://www.raspberrypi.org/downloads/raspbian/
[rpi-connect-wifi]: https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md
