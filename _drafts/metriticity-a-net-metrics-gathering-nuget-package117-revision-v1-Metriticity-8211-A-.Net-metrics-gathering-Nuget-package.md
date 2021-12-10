---
id: 122
title: 'Metriticity &#8211; A .Net metrics gathering Nuget package'
date: 2018-05-21T13:42:31+01:00
author: tom
layout: revision
guid: http://3.8.252.104/2018/05/21/117-revision-v1/
permalink: /?p=122
---
When people think of DevOps they frequently miss out or neglect the monitoring of their applications. One of the main goals of DevOps is to react quickly to situations and ensure the best experience for the end user; monitoring enables this and gives us the following advantages

  * Identifies problems in the application.
  * Drives insights into backlog from production.
  * Enables hypothesis-driven development.
  * Allows user telemetry to help the team take proactive actions instead of reactive actions.

Seeing problems as they occur is the most effective way of making sure that the problems are corrected as soon as possible. A system that is able to monitor, detect, and alert to problems will allow for mitigation or resolution of problems as soon as they occur.

To aid with this I have developed a Nuget package which can monitor c# applications and provide a wide range of metrics which should be able to be used to monitor application health. This package is [Metricity](https://www.nuget.org/packages/Metricity/).

Metricity can monitor method execution times, application cpu usage, application memory usage, method hit counts, timings, exception handling and a whole lot more. What separates Metricity from other packages is that it can natively send statistics to a remote database for live monitoring. You can use Metricity without any of the remote features and there is still a lot it can help you with without it however Metricity&#8217;s strengths lie with its database capabilities. Please see the [Metricity GitHub page](https://github.com/tomaustin700/Metricity) for up-to-date features however a rundown of the main methods can be seen below