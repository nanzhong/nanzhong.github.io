---
layout: post
title:  Juniper Pulse VPN with Two-Factor Auth on Archlinux
date:   2016-09-04 13:19:46 -0400
tags:   [linux,vpn,juniper-pulse]
---
_Edit: Oct 2, 2016 - This is irrelevant now as [OpenConnect](https://wiki.archlinux.org/index.php/OpenConnect) is able to handle all of this in a very straightforward and easy way now_
_Edit: Dec 4, 2014 - Updated for jre 8_

Recently at [work](http://digitalocean.com), we switched from openvpn to Juniper VPN, and I had quite the experience trying to get it working on Arch Linux. Hopefully this guide can save some of you guys a good day's worth of headache.

Depending on the policy configuration of your company/organization, setting up Juniper can be reasonably straightforward or very hackish. If your organization does not require you to login through the browser (eg. simple username/password auth), then you can use the `jnc` or `msjnc` command line tools to connect directly; the [archwiki](https://wiki.archlinux.org) has a [great article](https://wiki.archlinux.org/index.php/Juniper_VPN) that walks through the steps for getting that setup.

If your organization employs some sort of two-factor authentication, then the command line tools are a no-go and you will need to get the browser applet `network_connect` working. On the surface this seems pretty straight-forward but the problem is that it expects that you are running Red Hat, another rpm based distro (Juniper provides a rpm package), or a debian based distro (expects the `dpkg` based `update-alternatives` to be present). 

This leaves us archers in an akward spot. The following will essentially install `dpkg` on arch, and setup `update-alternatives` to return the correct java paths to allow juniper to work; this assumes you are running 64-bit (Juniper is 32-bit only but has recently been updated to support running on a 64-bit os provided that the 32-bit dependencies are installed), 32-bit users will likely not have these problems.

1. Install `jre7` and `bin32-jre7` from the AUR
  - I tried `openjdk` and `icedtea`, but it doesn't seem to work. Juniper doesn't seem to recognize the 32-bit jvm. Suprisingly, it works on ubuntu...
  
2. Make sure you have multilib enabled and install the necessary depenedencies:
  - `yaourt -S xterm lib32-glibc lib32-libzip lib32-gcc-libs lib32-libxext lib32-libxrender lib32-libxtst dpkg`, replace `yaourt` with pacman + whatever tool you use to install packages from the AUR
  - you can test whether you have all the needed libraries by running `sudo ldd ncsvc` in `~/.juniper_networks/network_connect` (sudo because `ncsvc` is setuid root)
  
3. Setup `update-alternatives` for `java`
  - `sudo update-alternatives --install /usr/bin/java java /opt/java32/jre/bin/java 50`
  - `sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-8-jre/bin/java 100`

4. Setup `update-alternatives` for `javaws`
  - `sudo update-alternatives --install /usr/bin/javaws javaws /opt/java32/jre/bin/javaws 50`
  - `sudo update-alternatives --install /usr/bin/javaws javaws /usr/lib/jvm/java-8-jre/bin/javaws 100`

5. Setup `update-alternatives` for `libnpjp2.so`
  - `sudo update-alternatives --install /usr/lib/mozilla/plugins/libnpjp2.so java_plugin /opt/java32/jre/lib/i386/libnpjp2.so 50`
  - `sudo update-alternatives --install /usr/lib/mozilla/plugins/libnpjp2.so java_plugin /usr/lib/jvm/java-8-jre/lib/amd64/libnpjp2.so 100`
  
6. You should be set to go! Login through the browser and starting `network_connect` should now present you with the vpn client.

![](/content/images/2014/Sep/2014-09-04-091453_554x347_escrotum.png)

Hopefully this will save some of you some time.
