---
id: 175
title: Configuring Bosun running inside a Docker container
date: 2018-12-17T20:44:09+00:00
author: tom
layout: post
guid: http://3.8.252.104/?p=175
permalink: /2018/12/17/configuring-docker-bosun/
categories:
  - bosun
  - devops
  - homelab
---
This is the third part in a series of articles regarding [Bosun](https://bosun.org/), the first two parts can be found [here](http://tomaustin.xyz/2018/12/11/monitoring-c-applications-using-bosun/) and [here](http://tomaustin.xyz/2018/12/13/sending-windows-data-to-bosun-using-scollector/). Now that you have Bosun running and logging metrics from [Bosun Reporter](https://github.com/StackExchange/BosunReporter) and [Scollector](https://bosun.org/scollector/) we should look at configuration!

  


This article will cover configuration for Bosun running inside a Docker container, if you are running on Windows or Linux things will be a little different but ultimately you will want to locate bosun.toml and edit it with your favorite text editor. For Docker things are slightly more complicated but nothing we can&#8217;t manage. Firstly connect to your Docker server (I&#8217;d recommend putty and run the following command to list your containers:

<pre class="wp-block-code"><code>sudo docker ps</code></pre>

  


We should now be greeted with a list of all of the running containers on your Docker server<figure class="wp-block-image">

<img loading="lazy" width="650" height="126" src="http://tomaustin.xyz/wp-content/uploads/2018/12/image-16.png" alt="" class="wp-image-176" srcset="https://tomaustin.xyz/wp-content/uploads/2018/12/image-16.png 650w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-16-300x58.png 300w" sizes="(max-width: 650px) 100vw, 650px" /> </figure> 

Ignore most of this, we are only concerned with the container id. Now connect to your container using the exec command.

<pre class="wp-block-code"><code>sudo docker exec -it 10cbc024198e /bin/bash</code></pre>

Now all we need to do is edit bosun.toml using a text editor (I like nano) and set our configuration

<pre class="wp-block-code"><code>nano /data/bosun.toml</code></pre>

At this point we should refer to the [Bosun Configuration](https://bosun.org/system_configuration) documentation and set the options we need to set. Once we are happy we can save and exit (ctrl+o, enter, ctrl+x if you are using nano), exit the container shell by using the exit command and restart our container for the configuration to be applied.

<pre class="wp-block-code"><code>sudo docker restart 10cbc024198e</code></pre>