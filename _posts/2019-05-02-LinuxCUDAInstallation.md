---
layout: post
title:  "Linux | 18.10 Install GPU and related software"
date:   2019-05-02 22:46
author: Botao Xiao
categories: Linux
description:
---
## Linux 18.10 Install GPU and related software

### Install Linux with ISO
1. Need to provide a UEFI partition and bios area.
2. Install the system to the bios area.
3. Reserve enough space on ssd.
4. Give swap 2 * Ram size.

### Set up your Integer net
1. Settings -> Internet->IPv4. Choose manual
2. setup ip address: 130.113.23.XXX
3. set mask: 255.255.0.0
4. set Gateway: 130.113.31.5
5. set DNS 130.113.184.55
6. Apply.

### Set up aptitude
```
sudo apt-get update
```

### Install GPU
#### Install Driver and CUDA / CUDNN for GPU
* Driver is the communication between hardware and OS, we need to install drive to let OS talk with GPU. We need to install the driver first.
    1. Download drive from NVIDIA website, select the correct version for the GPU, mine is RTX 2080 tI, here is the [link](https://www.nvidia.com/download/driverResults.aspx/138279/en-us).
    2. Go to the directory, change the permission of the .run file
      ```
      sudo chmod 777 NVIDIA-Linux-x86_64–410.57.run # Name might different
      ```
    3. Run the file.
      ```
      sudo ./NVIDIA-Linux-x86_64–410.57.run
      ```

      * Continue Installation -> Yes
    ４. Up to current step, the GPU is visible to OS, we can use command to check the GPU
      ```
      watch -n 1 nvidia-smi # Make sure you have watch and you are using a nvidia gpu.
      ```

* Once we install the GPU, we can start installing the CUDA for the system. CUDA is the base platform for GPU to do calculations.
  1. Download cuda from NVIDIA website, I am downloading [cuda toolkit 10.1](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1604)
  2. Since we are using the Integrated Graphic card on the motherboard, we need to add a blacklist to get rid of the effection.
    * Create a file called "blacklist-nouveau.conf" and write:
      ```Configure
      blacklist nouveau
      options nouveau modeset=0
      ```

    * Add the conf file to system Configure
      ```
      sudo cp (path to your conf)/blacklist-nouveau.conf /etc/modprobe.d
      ```

    * Make the conf file effective
      ```
      sudo update-initramfs -u
      ```
  3. Remove all cuda installed before
    ```
    sudo apt-get purge nvidia-cuda*
    ```
  4. Logout from GUI, and use "ALT + CTRL + F3" to open a new linux tty, type your user name and password to login.
  5. Go to the directory of the CUDA and run it with
    ```
    sudo sh cuda_10.0.130_410.48_linux.run # Name may differ.
    ```
  6. Select all option and start installation.
    * Before going back to GUI, add the path to global
      ```
      sudo vim ~/.bashrc
      export PATH=/usr/local/cuda-10.0/bin${PATH:+:${PATH}} # add
      export LD_LIBRARY_PATH=/usr/local/cuda-10.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}  # Add
      source ~/.bashrc #activate bashrc file
      ```
  7. After installation, you can use "ALT + ->" to go back to login GUI.
  8. If you meet a infinite loop when login with correct password after installation of CUDA problem(Optional)
    ```
    sudo rm -f /etc/modprobe.d/blacklist-nouveau.conf
    sudo update-initramfs -u
    sudo X -configure
    sudp cp (path to the file)/xorg.conf.new /etc/X11/xorg.conf # Give a new configure file for DirectX11, which is a new one with cuda.
    init 5        
    ```    

* Intall cudnn. CUDNN is a toolkit for stupid deep learning.
  1. Download your cudnn from nvidia and select the correct version.
  2. Copy and paste the key files
    ```
    tar -xf cudnn-10.0-linux-x64-v7.3.1.20.tgz

    sudo cp -R cuda/include/* /usr/local/cuda-10.0/include

    sudo cp -R cuda/lib64/* /usr/local/cuda-10.0/lib64
    ```

### Install Anaconda
1. Download the anaconda from its website and extract it.
2. Add anaconda to global environment
  ```
  vim ~/.bashrc
  export PATH="<path to anaconda>bin:$PATH"
  ```
3. Create your working environment:
  ```
  conda create -n tensorflow python=3.7
  ```
4. Now you can play with your anaconda.

### Reference
1. [How to install Nvidia drivers and cuda-10.0 for RTX 2080 Ti GPU on Ubuntu-16.04/18.04](https://medium.com/@avinchintha/how-to-install-nvidia-drivers-and-cuda-10-0-for-rtx-2080-ti-gpu-on-ubuntu-16-04-18-04-ce32e4edf1c0)
2. [Install NVIDIA Drivers in RHEL/CentOS/Fedora and Debian/Ubuntu/Linux Mint](https://www.tecmint.com/install-nvidia-drivers-in-linux/)
