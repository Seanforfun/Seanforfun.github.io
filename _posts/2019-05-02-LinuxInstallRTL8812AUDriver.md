---
layout: post
title:  "Linux | Install RTL8812AU Wireless Adapter Driver"
date:   2019-05-02 22:46
author: Botao Xiao
categories: Linux
description:
---
## Install RTL8812AU Wireless Adapter Driver

### Introduction
This driver works for my EDUP-Link usb adapter, I installed several times and each time I spent long time on finding the instruction. So it's time to record the process.

### Steps:
1. Must have the computer connected to Internet using wire.
2. Download the source file of the rtl 8812au driver by git.
  ```
  git clone https://github.com/aircrack-ng/rtl8812au.git
  ```
3. Go to the directory of the file.
  ```
  cd rtl8812au
  ```
4. Make and install the driver
  ```
  make -j8
  sudo make install
  ```
5. Reboot the system
  ```
  reboot
  ```

### Reference
1. [Install driver rtl8814au on ubuntu 18.04](https://askubuntu.com/questions/1042310/install-driver-rtl8814au-on-ubuntu-18-04)
