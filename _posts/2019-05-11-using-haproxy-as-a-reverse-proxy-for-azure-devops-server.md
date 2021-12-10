---
id: 298
title: Using HAProxy as a reverse proxy for Azure DevOps Server
date: 2019-05-11T18:29:32+01:00
author: tom
layout: post
guid: http://3.10.198.211/?p=298
permalink: /2019/05/11/using-haproxy-as-a-reverse-proxy-for-azure-devops-server/
categories:
  - azure devops server
  - devops
---
I recently had the need to allow access to an on-premise [Azure Devops Server](https://azure.microsoft.com/en-gb/services/devops/server/) instance over the internet. This had been attempted in the past using [apache](https://httpd.apache.org/) as the reverse proxy but due to ADS using NTLM Authentication ADS (or TFS as it was at the time) would constantly prompt for credentials without really getting anywhere. After a bit of research it looked like [HAProxy](http://www.haproxy.org/) might help with this so I decided to spin up a lab environment in Azure to test it out.

Before we start I would heavily suggest you don&#8217;t just reverse proxy ADS, ADS provides no 2 factor authentication capabilities so you&#8217;re going to be opening yourself up to credential stuffing and a whole host of other attacks by making it publicly accessible. Instead you probably want to be looking at migrating to [Azure DevOps](https://azure.microsoft.com/en-gb/services/devops/).

This tutorial assumes you already have Azure DevOps Server installed and configured. 

There isn&#8217;t really a great deal to this so the first thing to do is create yourself a HAProxy server, I used Ubuntu 18.04 and then installed HAProxy.

<pre class="wp-block-code"><code>sudo apt-get install haproxy</code></pre>

Now lets open up the HAProxy config file using nano and get to work!

<pre class="wp-block-code"><code>sudo nano /etc/haproxy/haproxy.cfg</code></pre>

Below the defaults we need to add a backend to tell HAProxy where to send our traffic.

<pre class="wp-block-code"><code>
backend backend_tfs
    server static adsip:80 check maxconn 3
    mode http
    balance roundrobin
    option http-keep-alive
    option prefer-last-server
    timeout server 30s
    timeout connect 4s
</code></pre>

Substitute &#8216;adsip&#8217; with your internal ADS/TFS IP address. Now let&#8217;s declare the frontend.

<pre class="wp-block-code"><code>frontend frontend_tfs
    bind :80 name frontend_tfs
    mode http
    option http-keep-alive
    timeout client 30s
    default_backend backend_tfs
</code></pre>

The options which are doing the magic here are **http-keep-alive** and **prefer-last-server**. Save, exit and reload HAProxy and you&#8217;re all done!

<pre class="wp-block-code"><code>sudo service haproxy restart</code></pre>

If you want to try this for yourself in Azure [here](https://gist.github.com/tomaustin700/55e0ad640f58c0dde4d809e18fe6c8ab) is a resource template which will create vm&#8217;s for a domain controller, ads instance and HAProxy. The template also includes a vm for apache in case you want to see what happens when you try and use apache as the reverse proxy.