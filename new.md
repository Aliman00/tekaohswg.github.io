---
layout: default
---

## Build a New SWG Server

This guide will help you build a new SWG Server VM from scratch. I recommend following this guide's every detail at least once before attempting to build a server with different settings or anything.

### Prerequisites

You need to have [Oracle VirtualBox](https://www.virtualbox.org/) installed on your host machine before you begin.

You should also download the latest installation media for Xubuntu 18.04 from [here](http://mirror.us.leaseweb.net/ubuntu-cdimage/xubuntu/releases/18.04/release/) (`-amd64.iso` is the one you want). This guide uses Xubuntu because I like it. Ubuntu is popular and makes some things a little easier to get running compared to Debian. And I prefer Xfce.

### Prepare your VM for installation

In VirtualBox, Select `Machine -> New...`. You will be prompted with a window to begin creating your VM. Enter Expert Mode if you're not already there. Then, give your VM a name, choose where it will be saved, and select `Type: Linux` and `Version: Ubuntu (64-bit)`. Choose how much RAM you will give your VM and be sure that you have the `Create a virtual hard disk now` option selected.

A note on memory size: To run the _entire_ game, you will need at least 16GB of RAM in your VM. If you don't have that much, just choose an amount as high as you can go while still leaving enough for the host to be comfortable running a SWG client. Individual mileage may vary.
