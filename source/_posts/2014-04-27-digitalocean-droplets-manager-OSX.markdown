---
layout: post
title: "DigitalOcean Droplets Manager for OSX"
date: 2014-04-27 23:55:37 -0600
comments: true
categories: osx cocoa digitalocean github
---


A couple days ago, browsing the web, I came across this [DigitalOcean Indicator](http://blog.andrewsomething.com/2014/04/25/digitalocean-indicator-release/) made by [Andrewsomething](http://blog.andrewsomething.com). Which manages [DigitalOcean](https://www.digitalocean.com) droplets easily from right Ubuntu's panel.

Because I've got droplets to manage, and there was no OSX version for such a handy tool, I decided to make an OSX version of it.

{% img center /images/posts/DODropletsManager/screenshot.png Droplets Manager %}

So far, the manager does the following: 

 - List droplets
 - Show each droplets status, IP address, distro and region
 - Copy droplet's IP address to clipboard
 - Reboot, shutdown and power on droplets


This app works but is still under development, but for those who want to try it or start using it, ~~you can [download the app file](http://bit.ly/QOpaBs)~~ (See update).

The source code is at my [Github repository](https://github.com/deivuh/DODropletManager-OSX), for those willing to contribute or just look at it.

###Update
There have been quite a few contributions, and thanks to them, the app is growing quite fast :). For .app build releases, please visit [The repository's release section](https://github.com/deivuh/DODropletManager-OSX/releases) to download them. 