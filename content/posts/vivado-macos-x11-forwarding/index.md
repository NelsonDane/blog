---
date: '2024-11-12T08:13:28-05:00'
draft: false
title: 'How to Access Vivado on MacOS via X11 Forwarding'
description: 'Using Xilinx Vivado and Vitis on MacOS via X11 forwarding.'
series: ['Senior Capstone Design']
series_order: 1
tags: ['vivado', 'vitis', 'x11', 'ssh', 'macos', 'linux']
---

## Introduction
While I was working on my senior capstone project at Clarkson, I ran into a small problem. I required access to Xilinx's [Vivado](https://www.amd.com/en/products/software/adaptive-socs-and-fpgas/vivado.html) for my project. But, Apple lover than I am, my personal laptop is a MacBook Air M2. This posed a problem since Vivado only runs well on Linux (and somewhat well on Windows). I didn't want to have to use a school computer every time I needed to work on my project, and running an x86 VM on my laptop would be very slow. So to solve this, I decided to use X11 forwarding to "run" Vivado on my MacBook.

**IMPORTANT NOTE**: Before you get too excited, I'd like to point out that Vivado will be running on a remote machine, not on the local MacBook. If you don't have access to a remote machine, then this guide won't be of much use to you. Perhaps you'd rather use something like [vivado-on-silicon-mac](https://github.com/ichi4096/vivado-on-silicon-mac) instead?

## Prerequisites
- A remote machine with Vivado already installed (I'm using my Arch-based desktop PC)
- A MacBook with [XQuartz](https://www.xquartz.org/) installed
- SSH access to the remote machine (preferably with a key)
- A decent internet connection...

## Remote PC Setup
Do steps these on the remote machine (For me, this is my Arch (btw) Desktop PC).
1. Ensure that the package `xauth` is installed. To check, run `which xauth`. This should return a path to the `xauth` binary like `/usr/bin/xauth`. If it doesn't, install the package. On Arch, you can do this with `sudo pacman -S xorg-xauth`.
2. Edit the SSH config file. Run `sudo vim /etc/ssh/sshd_config` and add the following line to the file:
    ```
    X11Forwarding yes
    X11DisplayOffset 10
    X11UseLocalhost yes
    ```
Save and exit the file (hopefully you know how to exit vim hehe).
3. Restart the SSH service. Run `sudo systemctl restart sshd`. Or if you want to be extra sure, you can restart the whole machine with `sudo reboot`.

**Note**: I'm assuming that you already have Vivado installed on the remote machine. If you don't, you can download it from [Xilinx's website](https://www.xilinx.com/support/download.html). You'll need to create an account to download it. I'm currently using `2023.1`, but you can use any version you want.

## MacBook Setup
Do these on your MacBook. A private key isn't required, but highly recommended for security and convenience (see below).
1. Install [XQuartz](https://www.xquartz.org/). This is required for X11 Forwarding. After installing, you'll need to log out and log back in to enable it. I also recommend checking `Focus Follows Mouse` in the XQuartz preferences.
2. Create a private key if you don't already have one. Run `ssh-keygen -t rsa -b 4096 -f ~/.ssh/vivado_rsa` and follow the prompts. This will create a private key at `~/.ssh/vivado_rsa` and a public key at `~/.ssh/vivado_rsa.pub`.
3. Copy the public key to the remote machine. Run `ssh-copy-id -i ~/.ssh/vivado_rsa.pub user@remote-machine`. Replace `user` with your username on the remote machine and `remote-machine` with the IP address or hostname of the remote machine. If you need to access your remote machine when you're not on your home network, I'd recommend setting up [wireguard](https://www.wireguard.com/).
4. Test the connection. Run `ssh -i ~/.ssh/vivado_rsa user@remote-machine`. If everything is set up correctly, you should be logged into the remote machine without needing to enter a password.

That's a lot of typing, and we don't want to do that every time we connect. Wouldn't it be great if we could just run `ssh vivado` and be connected to the remote machine? Well, do I have some news for you! Create/edit the file `~/.ssh/config` and add the following:
```
Host vivado
    HostName your-remote-machine-ip
    User your-remote-username
    IdentityFile ~/.ssh/vivado_rsa
    ForwardX11 yes # To enable X11 forwarding
    Compression yes # To speed up the connection
    ForwardX11Trusted yes # Trust the X11 forwarding
```

## Running Vivado
1. Connect to the remote machine. Run `ssh vivado`. Isn't that great?
2. Launch Vivado. Run `vivado` in the terminal. If everything is set up correctly, you should see the Vivado GUI pop up on your MacBook. (You may need to type the full path to the Vivado binary if it's not in your `PATH`, by default it's in `/opt/Xilinx/Vivado/<version>/bin/vivado`). If you want to run Vivado in the background, you can append `&` to either command.

And that's it! You're now able to access Vivado from anywhere with an internet connection (now you can never escape...). This also works with Xilinx [Vitis](https://www.amd.com/en/products/software/adaptive-socs-and-fpgas/vitis.html) as well, just launch it from the Vivado GUI or run `vitis` in the terminal.
