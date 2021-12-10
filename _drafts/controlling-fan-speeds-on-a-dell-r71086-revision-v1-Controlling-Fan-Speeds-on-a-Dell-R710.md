---
id: 104
title: Controlling Fan Speeds on a Dell R710
date: 2018-03-20T11:29:02+00:00
author: tom
layout: revision
guid: http://3.8.252.104/2018/03/20/86-revision-v1/
permalink: /?p=104
---
A few months back I upgraded my home server to a Dell R710, I got a good deal on it and it came preloaded with 72GB ram and two Xeons so it was much more powerful than the server it replaced. The problem with it is that the R710 is an enterprise grade server and is intended for data centres and not the home. Because of this it has multiple blower fans which are great for airflow and keep everything nice and cool however they can be very noisy which was problematic. After a bit of research I found that the fan rpm could be controlled via [IPMI](https://en.wikipedia.org/wiki/Intelligent_Platform_Management_Interface). I spun up a Ubuntu server VM and modified a script I found on [GitHub](https://github.com/NoLooseEnds/Scripts/blob/master/R710-IPMI-TEMP/R710-IPMITemp.sh) to periodically check the system temperature and modify the fan speed accordingly. Most of the time the server is not under load so the fan speeds ramp down to acceptable noise levels. The final script can be found below (in order to use it you will need to install ipmitool). This script can be ran using a cronjob (I have mine running via Jenkins every 5 mins).