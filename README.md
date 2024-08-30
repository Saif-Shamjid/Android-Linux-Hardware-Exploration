# Android-Linux-Hardware-Exploration
Exploring my phone hardware devices from my phone android linux

# Exploring and Controlling the FM Radio Transceiver on My Device

This documentation outlines the steps I took to explore and attempt to control the FM radio transceiver on my device using various Linux tools. This journey involved identifying the device, understanding its drivers, and attempting to interact with it through system commands.

## Table of Contents
- [Initial Device Identification](#initial-device-identification)
- [Exploring the Character Device](#exploring-the-character-device)
- [Examining the V4L2 (Video4Linux2) Interface](#examining-the-v4l2-video4linux2-interface)
- [Controlling the Radio Transceiver](#controlling-the-radio-transceiver)
- [Identifying Processes Using the Device](#identifying-processes-using-the-device)
- [Conclusion](#conclusion)

## Initial Device Identification

I started by identifying the character device associated with the FM radio transceiver. Using the `file` command, I located a character special device with the major and minor numbers `81:21`.

```bash

$ file /dev/radio0
# Output:
# /dev/radio0: character special (81/21)

I navigated to the /sys/dev/char/81:21 directory to gather more information about the device. Here's what I found:

$ cd /sys/dev/char/81:21
$ ls
# Output:
# debug  dev  index  name  power  subsystem  uevent

$ cat name
# Output:
# radio-iris

$ cat uevent
# Output:
# MAJOR=81
# MINOR=21
# DEVNAME=radio0

$ ls -l subsystem
# Output:
# lrwxrwxrwx 1 root root 0 2024-08-30 08:01 subsystem -> ../../../../class/video4linux


This indicated that the radio0 device was associated with the video4linux subsystem.

I installed the v4l-utils package to interact with the device using the V4L2 utilities.
$ sudo apt-get install v4l-utils

I then listed all V4L2 devices and gathered more information about the radio0 device:

$ sudo v4l2-ctl --list-devices
# Output:
# msm_vdec_8974 ():
#    /dev/video32
#    /dev/video33
#    /dev/radio0
#    /dev/v4l-subdev12

$ sudo v4l2-ctl -d /dev/radio0 --all
# Output:
# Driver Info:
#    Driver name      : radio-iris
#    Card type        : QTI FM Radio Transceiver
#    Driver version   : 3.18.31
#    Capabilities     : 0x00250000
#        Tuner
#        Radio
#        Extended Pix Format

I attempted to control the FM radio transceiver using the v4l2-ctl commands. However, I encountered issues:

$ sudo v4l2-ctl -d /dev/radio0 --set-freq=90.5
# Output:
# VIDIOC_G_TUNER: failed: Invalid argument
# VIDIOC_S_FREQUENCY: failed: Invalid argument


To understand why /dev/video0 was busy, I identified which process was using it:
$ sudo fuser /dev/video0
# Output:
# /dev/video0: 814

$ sudo ps -p 814 -o comm=
# Output:
# mm-qcamera-daem


Conclusion
Through this journey, I learned about the character device handling the FM radio transceiver and explored various system utilities to interact with it. Although I encountered some challenges, such as busy devices and invalid arguments, this exploration provided valuable insights into how multimedia devices are managed on Linux.

This documentation will be updated as I continue to work on controlling the FM radio transceiver and understanding the underlying drivers and processes involved.

