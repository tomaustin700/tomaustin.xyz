---
id: 56
title: Updating Ubuntu Server 17.10 to use Static IP
date: 2018-01-04T12:24:26+00:00
author: tom
layout: post
guid: http://3.8.252.104/?p=56
permalink: /2018/01/04/updating-ubuntu-server-17-10-to-use-static-ip/
categories:
  - homelab
tags:
  - "17.10"
  - netplan
  - static ip
  - ubuntu
---
A recent problem I came across was setting my Nginx server which is running Ubuntu Server 17.10 to use a static IP address. The normal approach of editing /etc/network/interfaces wasn&#8217;t working so I gave up on it and let DHCP do its thing, this worked fine until my Hyper-V server rebooted and then Nginx got a different address which broke my port forwarding rules and all sites nginx was hosting. After some digging around it turns out things are different in 17.10 and configuring network interfaces is done by editing  /etc/netplan/01-netcfg.yaml (a different yaml file exists for each interface you have). Once editing in your favourite editor you can set it to something like the following:

<pre># This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
 version: 2
 renderer: networkd
 ethernets:
   ens33:
     dhcp4: no
     dhcp6: no
     addresses: [192.168.1.2/24]
     gateway4: 192.168.1.1
     nameservers:
       addresses: [8.8.8.8,8.8.4.4]</pre>

Then save the file and run the following command to apply the changes

<pre>sudo netplan apply</pre>

&nbsp;