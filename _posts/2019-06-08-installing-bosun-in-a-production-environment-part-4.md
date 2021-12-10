---
id: 446
title: 'Installing Bosun in a Production Environment &#8211; Part 4'
date: 2019-06-08T20:03:23+01:00
author: tom
layout: post
guid: http://3.10.198.211/?p=446
permalink: /2019/06/08/installing-bosun-in-a-production-environment-part-4/
image: /wp-content/uploads/2019/06/bosun.png
categories:
  - bosun
  - devops
---
Parts 1, 2 and 3 of this series can be found at the following locations: [Part 1](http://tomaustin.xyz/2019/05/05/installing-bosun-in-a-production-environment/), [Part 2](http://tomaustin.xyz/2019/05/11/installing-bosun-in-a-production-environment-part-2/) and [Part 3](http://tomaustin.xyz/2019/06/01/installing-bosun-in-a-production-environment-part-3/).

This is the final article in this series covering how to install [Bosun](https://bosun.org/) in a production environment. The previous articles showed you how to [install Bosun](http://tomaustin.xyz/2019/05/05/installing-bosun-in-a-production-environment/), [configure TSDBRelay to relay and aggregate data](http://tomaustin.xyz/2019/05/11/installing-bosun-in-a-production-environment-part-2/) and then run TSDBRelay in a load balanced configuration using [HAProxy](http://tomaustin.xyz/2019/06/01/installing-bosun-in-a-production-environment-part-3/). This final article will guide you through installing [Scollector](https://bosun.org/scollector/) on your servers to collect metrics for Bosun. Scollector is a metric collection agent and has collectors for Linux, Darwin and Windows. We are going to be installing it on our Bosun server as well as our TSDBRelay instances and our HAProxy instance. The installations steps will be identical on every server as all of our existing servers are Ubuntu 18.04 so I will only go through the process once but do make sure you install it on every server. If you do want to install Scollector on a Windows box I already have a tutorial showing how to do that [here](http://tomaustin.xyz/2018/12/13/sending-windows-data-to-bosun-using-scollector/).

We are going to need to install GO and then Supervisor before we can install and configure Scollector and these steps are identical to those carried out in the first and second article so if you are wanting to install Scollector on a server which already has GO and Supervisor installed skip to the &#8216;Pull Scollector from Github&#8217; section.

**Install GO**

You will only need to install GO on your HAProxy instance as our Bosun and TSDBRelay servers already have GO installed.

Scollector requires [version 1.11.2](https://github.com/golang/go/issues?q=milestone%3AGo1.11.2) of [Go](https://golang.org/) to run so the first thing we need is download that version and install it. If you have followed the steps to install Bosun and TSDBRelay you should be familiar with these commands by now. SSH into your HAProxy server and run the following commands: 

<pre class="wp-block-code"><code>curl -O https://dl.google.com/go/go1.11.2.linux-amd64.tar.gz</code></pre>

<pre class="wp-block-code"><code>tar -xvf go1.11.2.linux-amd64.tar.gz   </code></pre>

<pre class="wp-block-code"><code>sudo mv go /usr/local</code></pre>

<pre class="wp-block-code"><code>sudo nano  ~/.profile</code></pre>

Add the following to the end of the file then save and exit (ctrl+o, enter, ctrl+x) 

<pre class="wp-block-code"><code>export GOPATH=$HOME/work
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin﻿</code></pre>

One last command now before GO should be installed: 

<pre class="wp-block-code"><code>source ~/.profile</code></pre>

To check everything worked correctly we can run &#8216;go version&#8217; and check the response: 

<pre class="wp-block-code"><code>go version</code></pre>

And we should get: 

<pre class="wp-block-code"><code>go version go1.11.2 linux/amd64</code></pre>

I**nstall Supervisor**

Like when we installed TSDBRelay we are going to use Supervisor to control the Scollector process. Install Supervisor with the following command:

<pre class="wp-block-code"><code>sudo apt-get install supervisor</code></pre>

**Pull Scollector from Github**

Like when we have previously pulled from Github we are going to start by creating a directory for Scollector before using Wget to download Scollector.

<pre class="wp-block-code"><code>sudo mkdir -m 777 /scollector</code></pre>

<pre class="wp-block-code"><code>sudo wget -O /scollector/scollector https://github.com/bosun-monitor/bosun/releases/download/0.8.0-preview/scollector-linux-amd64</code></pre>

Now we can use [chmod](https://www.poftut.com/chmod-x-command-linux-unix/) to mark Scollector as executable. 

<pre class="wp-block-code"><code>sudo chmod +x /scollector/scollector</code></pre>

Now we need to create our scollector.toml configuration file, normally you wont need to do this however if you are wanting to gather HAProxy metrics it is needed &#8211; skip this step if you are not requiring those.

<pre class="wp-block-code"><code>sudo nano /scollector/scollector.toml</code></pre>

Enter the following into the file, this will instruct Scollector to gather HAProxy metrics.

<pre class="wp-block-code"><code>[[HAProxy]]
  User = "admin"
  Password = "pass"
  [[HAProxy.Instances]]
    Tier = "1"
    URL = "http://0.0.0.0/haproxy?stats;csv"</code></pre>

**Create a Supervisor file to run Scollector**

Just like when we have previously used Supervisor to run Bosun and TSDBRelay we are going to create a configuration file to run Scollector.

<pre class="wp-block-code"><code>sudo nano /etc/supervisor/conf.d/scollector.conf</code></pre>

[Here](https://gist.github.com/tomaustin700/fd61d66ed6d2d5cd3b8f291b9e148320) is a basic config file for you to use. The ip address after -h is the address of our HAProxy instance (0.0.0.0 in this case as we are installing Scollector on the HAProxy server). You don&#8217;t need to specify the location of the scollector.toml file as it will be auto detected if it is in the same location as Scollector.

Now let&#8217;s tell supervisor to reread and update 

<pre class="wp-block-code"><code>sudo supervisorctl reread</code></pre>

<pre class="wp-block-code"><code>sudo supervisorctl update</code></pre>

That should be it for HAProxy. If we navigate to Bosun we should see HAProxy under the list of hosts, if we then click on the server and navigate to &#8216;Available Metrics&#8217; we should see our HAProxy data.

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/06/image.png" alt="" class="wp-image-458" width="395" height="417" srcset="https://tomaustin.xyz/wp-content/uploads/2019/06/image.png 447w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-284x300.png 284w" sizes="(max-width: 395px) 100vw, 395px" /></figure>
</div>

Now repeat these steps on your other servers. Remember that if you are only wanting system metrics and nothing bespoke you don&#8217;t need to specify the scollector.toml file.

Once done you should now be able to see a whole host of metrics from all your servers within Bosun. You can now start creating expressions to view this data and add rules using the Rule Editor to alert you when things are going wrong.<figure class="wp-block-gallery columns-3 is-cropped aligncenter">

<ul class="blocks-gallery-grid">
  <li class="blocks-gallery-item">
    <figure><img loading="lazy" width="1201" height="836" src="http://tomaustin.xyz/wp-content/uploads/2019/06/image-4.png" alt="" data-id="469" data-link="http://tomaustin.xyz/?attachment_id=469" class="wp-image-469" srcset="https://tomaustin.xyz/wp-content/uploads/2019/06/image-4.png 1201w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-4-300x209.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-4-768x535.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-4-1024x713.png 1024w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-4-720x501.png 720w" sizes="(max-width: 1201px) 100vw, 1201px" /></figure>
  </li>
  <li class="blocks-gallery-item">
    <figure><img loading="lazy" width="1201" height="843" src="http://tomaustin.xyz/wp-content/uploads/2019/06/image-3.png" alt="" data-id="468" data-link="http://tomaustin.xyz/?attachment_id=468" class="wp-image-468" srcset="https://tomaustin.xyz/wp-content/uploads/2019/06/image-3.png 1201w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-3-300x211.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-3-768x539.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-3-1024x719.png 1024w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-3-720x505.png 720w" sizes="(max-width: 1201px) 100vw, 1201px" /></figure>
  </li>
  <li class="blocks-gallery-item">
    <figure><img loading="lazy" width="1203" height="811" src="http://tomaustin.xyz/wp-content/uploads/2019/06/image-2.png" alt="" data-id="467" data-link="http://tomaustin.xyz/?attachment_id=467" class="wp-image-467" srcset="https://tomaustin.xyz/wp-content/uploads/2019/06/image-2.png 1203w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-2-300x202.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-2-768x518.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-2-1024x690.png 1024w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-2-720x485.png 720w" sizes="(max-width: 1203px) 100vw, 1203px" /></figure>
  </li>
  <li class="blocks-gallery-item">
    <figure><img loading="lazy" width="1208" height="784" src="http://tomaustin.xyz/wp-content/uploads/2019/06/image-1.png" alt="" data-id="466" data-link="http://tomaustin.xyz/?attachment_id=466" class="wp-image-466" srcset="https://tomaustin.xyz/wp-content/uploads/2019/06/image-1.png 1208w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-1-300x195.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-1-768x498.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-1-1024x665.png 1024w, https://tomaustin.xyz/wp-content/uploads/2019/06/image-1-720x467.png 720w" sizes="(max-width: 1208px) 100vw, 1208px" /></figure>
  </li>
</ul></figure> 

If you would like to build the entire environment in Azure [here](http://tomaustin.xyz/download/463/) are the template files to allow you to do that easily. I hope this series helped you and if you need help or have any questions please leave a comment.