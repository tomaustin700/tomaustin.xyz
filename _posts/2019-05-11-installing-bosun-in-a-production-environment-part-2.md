---
id: 312
title: 'Installing Bosun in a Production Environment &#8211; Part 2'
date: 2019-05-11T18:06:04+01:00
author: tom
layout: post
guid: http://3.10.198.211/?p=312
permalink: /2019/05/11/installing-bosun-in-a-production-environment-part-2/
image: /wp-content/uploads/2019/06/bosun.png
categories:
  - bosun
  - devops
---
Parts 1, 2, 3 and 4 of this series can be found at the following locations: [Part 1](http://tomaustin.xyz/2019/05/05/installing-bosun-in-a-production-environment/), [Part 2](http://tomaustin.xyz/2019/05/11/installing-bosun-in-a-production-environment-part-2/), [Part 3](http://tomaustin.xyz/2019/06/01/installing-bosun-in-a-production-environment-part-3/) and [Part 4](http://tomaustin.xyz/2019/06/08/installing-bosun-in-a-production-environment-part-4/).

This is the second article in this series covering how to install [Bosun](https://bosun.org/) in a production environment. The first article can be found [here](http://tomaustin.xyz/2019/05/05/installing-bosun-in-a-production-environment/) and covers the installation and configuration of Bosun, this article will cover installing [TSDBRelay](https://godoc.org/bosun.org/cmd/tsdbrelay). Instead of directly sending data to Bosun we can send our data through TSDBRelay which can forward the data to OpenTSDB and then also to Bosun for indexing, this get&#8217;s Bosun out of the &#8216;critical path&#8217; and allow us to keep collecting data in the event Bosun goes down. 

A lot of these steps will be very similar to the steps we took to install Bosun so we will start with an Ubuntu 18.04 server before installing go and then grabbing the latest version of TSDBRelay.

**Install Go**

TSDBRelay requires [version 1.11.2](https://github.com/golang/go/issues?q=milestone%3AGo1.11.2) of [Go](https://golang.org/) to run so the first thing we need is download that version and install it. SSH into your Linux server and run the following commands: 

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

**Install Supervisor**

We&#8217;re going to use [Supervisor](http://supervisord.org/) to monitor and control the TSBDRelay process so we need to install that using the following command:

<pre class="wp-block-code"><code>sudo apt-get install supervisor</code></pre>

**Pull TSDBRelay from Github**

Let&#8217;s create a directory for TSDBRelay and then use wget to pull the latest release from github.

<pre class="wp-block-code"><code>sudo mkdir -m 777 /tsdbrelay</code></pre>

<pre class="wp-block-code"><code>sudo wget -O /tsdbrelay/tsdbrelay https://github.com/bosun-monitor/bosun/releases/download/0.8.0-preview/tsdbrelay-linux-amd64</code></pre>

Now we can use [chmod](https://www.poftut.com/chmod-x-command-linux-unix/) to mark tsdbrelay as executable.

<pre class="wp-block-code"><code>sudo chmod +x /tsdbrelay/tsdbrelay</code></pre>

**Create a Supervisor file to run TSDBRelay**

Like with Bosun we need to create a supervisor config file to run TSDBRelay, this config file will also contain the parameters to tell TSDBRelay where our Bosun and OpenTSDB servers are.

<pre class="wp-block-code"><code>sudo nano /etc/supervisor/conf.d/tsdbrelay.conf</code></pre>

[Here](https://gist.github.com/tomaustin700/d0d3fd6cf070281117c95260d468fb2d) is a basic config file for you to use, substitute bosunip, opentsdbip and redisip with the IP&#8217;s of your servers (you can also use url&#8217;s if you want), this file also tells TSDBRelay to listen on port 5252 so change that if you wish. Save and exit.

Now let&#8217;s tell supervisor to reread and update and that should be it!

<pre class="wp-block-code"><code>sudo supervisorctl reread</code></pre>

<pre class="wp-block-code"><code>sudo supervisorctl update</code></pre>

All data should now be able to be sent through TSDBRelay instead of directly to our Bosun server. This is a fairly basic configuration but TSDBRelay can also do normalisation, more info on that can be found [here](https://riptutorial.com/bosun/example/2665/tsdbrelay-systemd-unit-file).

[The next article](http://tomaustin.xyz/2019/06/01/installing-bosun-in-a-production-environment-part-3/) will cover running multiple TSDBRelay instances behind a HAProxy load balancer.