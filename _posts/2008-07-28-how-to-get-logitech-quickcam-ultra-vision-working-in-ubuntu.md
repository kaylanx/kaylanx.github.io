---
title: 'How to get Logitech QuickCam Ultra Vision working in Ubuntu'
layout: post
---

I have recently removed `Windows` from my Dell Inspiron 9400, and replaced it with Ubuntu, which is a flavour of `Linux` (or `GNU/Linux` if you're picky).

I was surprised at how easy the installation was, almost everything worked out of the box, wireless, all the media keys and 95% of the `fn` keys. (I'm having trouble with `fn+f8` and `fn+10` not working correctly)

Anyway I am very happy with the Installation, I love the effects you have on the windows, it looks really pretty.

I wanted to get my **Logitech QuickCam Ultra Vision** working with `Ubuntu`, so I could use it on **Skype**. I plugged it in and nothing happened…ho hum… on Windows it would have picked it up and installed it, and asked for the driver CD, but anyway, after a bit of googling around I found this [thread](https://answers.launchpad.net/ubuntu/+question/3743){:target="_blank"}, and I followed the advise of Andrew Barber on there.

Make sure first you have all the tools for the job:

{% highlight shell %}
sudo apt-get install linux-headers-`uname -r` linux-restricted-modules-`uname -r` build-essential subversion
{% endhighlight %}

Once you have got all them you will need to use subversion to get the driver from the svn repo..

{% highlight shell %}
svn checkout http://svn.berlios.de/svnroot/repos/linux-uvc/
{% endhighlight %}

Then you need to compile and install
{% highlight shell %}
$ cd linux-uvc/linux-uvc/trunk
$ make
$ sudo make install
{% endhighlight %}

Plug the camera in and take a look at `dmesg`. It may *hopefully* give you the device listing for it... eg `/dev/video1`

Point your application at that device and see if it works.
I checked dmseg to see what had happened...

{% highlight shell %}
$ dmesg | grep -i vid
[0.000000] BIOS-provided physical RAM map:
[8.445959] Boot video device is 0000:00:02.0
[26.441798] input: Video Bus as /devices/LNXSYSTM:00/device:00/PNP0A03:00/device:2b/LNXVIDEO:00/input/input7
[26.489145] ACPI: Video Device [VID] (multi-head: yes rom: no post: no)
[26.497079] input: Video Bus as /devices/LNXSYSTM:00/device:00/PNP0A03:00/LNXVIDEO:01/input/input8
[26.537089] ACPI: Video Device [VID1] (multi-head: yes rom: no post: no)
[26.537246] input: Video Bus as /devices/LNXSYSTM:00/device:00/PNP0A03:00/LNXVIDEO:02/input/input9
[26.585000] ACPI: Video Device [VID2] (multi-head: yes rom: no post: no)
[2902.096243] Linux video capture interface: v2.00
[2902.175522] uvcvideo: Found UVC 1.00 device <unnamed> (046d:08c9)
[2902.189674] usbcore: registered new interface driver uvcvideo
[2902.189683] USB Video Class driver (v0.1.0)
{% endhighlight %}

After that it worked in Skype, I needed to set the sound in device to be the microphone of the web cam in the options of Skype...

{% highlight shell %}
Sound In: USB Device 0x46d:0x8c9 (hw:U0x46d0x8c9,0)
Sound Out: Default device(default)
Ringing: Default device(default)
{% endhighlight %}

...and in the video settings of Skype...

{% highlight shell %}
Select webcam: UVC Camera (0x46d:0x8c9) (/dev/video0)
{% endhighlight %}

I am still currently having a weird issue where it seems that Skype takes over the sound of all other applications. If I try and play some music after a Skype call then it won't play until I reboot. 

Also it seems I'm allowed a max of 1 Skype call before the sound in Skype doesn't work, and therefore doesn't let me make or receive anymore calls. Again it only seems at the moment that a reboot will fix it. 

For the moment I will live with this as I don't use Skype that often anyway. (If anyone has any advice for this part please leave a comment)
