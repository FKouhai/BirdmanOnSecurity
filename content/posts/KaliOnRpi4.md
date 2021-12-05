---
title: "Install Kali on a headless Raspberry PI 4"
date: 2021-12-05T10:33:54+01:00
---

# Introduction

Before we begin with this journey, you may have some questions like, why would I want to install Kali Linus on a Raspberry Pi?

Well that's indeed a good question, in fact I asked myself the same question, and all I was able to come up with was:

- First its a cool experiment to do if you have an spare PI
- It's also good if you use sites like [HTB](https://www.hackthebox.com) or [tryhackme](https://tryhackme.com/), which instead of using a VM you could use a PI
- And you also get to understand a bit more about how X11 works

Now with that said, I want to point out the things that we are going to need:

- A Raspberry PI,[I'd prefer for this the 4B with 8GB of ram](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/).
- A micro SD reader/adapter.
- Not being scared of the cli.

In my case since I use [Manjaro](https://manjaro.org) as my daily driver I will guide you with the steps I followed, but they shouldn't vary from any other OS/distro.

After this introduction let's begin.

# Installing Kali

First we need to download the [kali](https://www.kali.org/get-kali/#kali-arm) image from their site.
_take into account it has to be the one for arm since the PI's are build with an arm processor_.

Then after we've downloaded the image file, we have 2 main methods in Linux, use a program like [Balena Etcher](https://www.balena.io/etcher/) or use
[dd](https://linux.die.net/man/1/dd).
In windows the main method I know is using [Rufus](https://rufus.ie) or Balena Etcher.

So after we have installed Balena Etcher(or if you haven't installed it and wanna use dd)
We need to connect our micro SD card into our SD reader/adapter.
Then after our OS recognise the card, we can check what name the device was given.

In order to do that, we have to run [lsblk](https://linux.die.net/man/8/lsblk) on Linux, this command prints out the physical storage devices connected to our computer.

Once we have located our device, we can either run Balena or dd.

Note that you must need to run Balena with super user privileges(you can open up a terminal and run sudo balena-etcher-electron this one is for Arch based OSes).

Then in Balena we pick the file we've downloaded from the Kali Linux website, after that we pick the device where we want to write the file, and we flash it.

If you are using dd run the following command:

```bash
sudo dd status=progress if=kali_image of=/dev/sdXY bs=4M
```

Now all we have to do is wait until the device is flashed with the Kali image, once this is done, we have to mount the device in our file system, to do that run:

```bash
sudo mount /dev/sdXY kali
sudo mount /dev/SDXZ kali/boot
```

Once we have mounted in our file system the SD with both of it's partitions, we have to do a few things

Firstly we have to create an empty file called ssh in the kali/boot folder, this is for the PI to enable ssh on boot time.

```bash
touch kali/boot/ssh
```

Then we need to edit the following file kali/etc/network/interfaces.d/eth0 , and it has to contain the following content.

```bash
allow-hotplu eth0
iface eth0 inet static
address A.B.C.D
netmask 255.255.255.0 
gateway A.B.C.E
```

Note that the network mask can vary in case you are in an other subnet usually Its a /24 (255.255.255.0).

After configuring the network we are almost done, now we need to edit the following file kali/boot/config.txt.

And find and edit/include the following content:

```bash
framebuffer_width=1280
framebuffer_height=720
hdmi_force_hotplug=1
```
Note that it may appear with a \# sign, if so just remove the hash and save the file.

After doing this, we can unmount the SD:

```bash
sudo umount kali/boot
sudo umount kali
```
And we can proceed to insert the SD card into our PI, we power on our PI, and wait until it finally boots(you can ping the IP address), after the IP is reachable
We have to wait for 30 seconds in order to let the ssh service start

Once we are able to login into our pi, we can safely ssh into it by running:

```bash
ssh kali@kali_ip
```

Note that the default password is kali, you may want to change this password by running:

```bash
passwd 
```

After changing the password we need to install x11vnc and configure it:

```bash
sudo apt update
sudo apt install -y x11vnc
sudo x11vnc -storepasswd /etc/vncserver.pass
```
We can create a systemd unit so we can start on boot time the vncserver, in order to do that

We need to create a file in /etc/systemd/system/vncserver.service which needs to contain the following:

```bash
[Unit]
Description=Start x11vnc at startup, before login.
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -display :0 -auth guess -forever -loop -noxdamage -repeat -rfbauth /etc/vncserver.pass -rfbport 5900 -shared

[Install]
WantedBy=multi-user.target
```
Then we can start and enable the service:

```bash
sudo systemctl start vncserver
sudo systemctl enable vncserver
```
We can check if the server is listening running:
 
```bash
sudo netstat -tuna | grep 5900
```

If you see something then, you are good to go, if you don't you may need to run

```bash
systemctl status vncserver
```
And start debugging, one first step may be restarting the PI.Then once everything is fine, we can safely connect using a vnc client like [remmina](https://remmina.org).
In my case, I have already installed the vnc server plugin for remmina from the AUR and I've created a quick connection, after creating it was able to connect to my Kali PI.

With that said I hope that you found this post helpful ✌

![](https://media.giphy.com/media/obQ0Q8dav3L5S/giphy.gif)


