---
id: 307
title: Installing Bosun on Ubuntu
date: 2019-05-05T14:34:41+01:00
author: tom
layout: revision
guid: http://3.10.198.211/2019/05/05/296-revision-v1/
permalink: /?p=307
---
Whenever I&#8217;ve written about Bosun in the past I&#8217;ve always used the [official Bosun Docker image provided by Stack Exchange](https://hub.docker.com/r/stackexchange/bosun), this has been fine for demonstrations and examples however what if we wanted to run Bosun in a production environment? Stack Exchange recommend against using the Docker image in production so this will be the first article in a series covering how to install and configure Bosun in a production environment. 

We&#8217;re going to be using OpenTSDB as the backend data store for Bosun, due to the complexity of installing OpenTSDB I&#8217;m not going to be covering the installation here but if you want to get something working quickly [I do have a Docker image which you can use,](https://hub.docker.com/r/tomaustin/opentsdb) this image is a tweaked version of the [image provided by Peter Grace](https://hub.docker.com/r/petergrace/opentsdb-docker) with the following configuration additions which allow BosunReporter to send data to it:

<pre class="wp-block-code"><code>tsd.http.request.enable_chunked=true
tsd.http.request.max_chunk=33554432</code></pre>

Now that we have OpenTSDB setup let&#8217;s get down to installing Bosun. I am going to be using Ubuntu 18.04 LTS but the steps should work on any Linux distro.

**Install Go**

Bosun requires [version 1.11.2](https://github.com/golang/go/issues?q=milestone%3AGo1.11.2) of [Go](https://golang.org/) to run so the first thing we need is download that version and install it. SSH into your Linux server and run the following commands: 

<pre class="wp-block-code"><code>curl -O https://dl.google.com/go/go1.11.2.linux-amd64.tar.gz</code></pre>

<pre class="wp-block-code"><code>tar -xvf go1.11.2.linux-amd64.tar.gz   </code></pre>

<pre class="wp-block-code"><code>sudo mv go /usr/local</code></pre>

<pre class="wp-block-code"><code>sudo nano  ~/.profile</code></pre>

Add the following to the end of the file then save and exit (ctrl+o, enter, ctrl+x)

<pre class="wp-block-code"><code>export GOPATH=$HOME/work
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin﻿</code></pre>

One last command now before Go should be installed:

<pre class="wp-block-code"><code>source ~/.profile</code></pre>

To check everything worked correctly we can run &#8216;go version&#8217; and check the response:

<pre class="wp-block-code"><code>go version</code></pre>

And we should get:

<pre class="wp-block-code"><code>go version go1.11.2 linux/amd64</code></pre>

**Install Supervisor**

We&#8217;re going to use [Supervisor](http://supervisord.org/) to monitor and control the Bosun process so we need to install that using the following command:

<pre class="wp-block-code"><code>sudo apt-get install supervisor</code></pre>

**Pull Bosun from Github and configure**

Now that the pre-requisites are taken care of we can finally get down to installing Bosun. We&#8217;ll start by creating a directory for Bosun and then pull the latest release from Github before setting up the configuration files.

<pre class="wp-block-code"><code>sudo mkdir -m 777 /bosun</code></pre>

<pre class="wp-block-code"><code>sudo wget -O /bosun/bosun https://github.com/bosun-monitor/bosun/releases/download/0.8.0-preview/bosun-linux-amd64</code></pre>

<pre class="wp-block-code"><code>sudo chmod +x /bosun/bosun</code></pre>

<pre class="wp-block-code"><code>sudo mkdir -m 777 /data</code></pre>

Now we need to create our bosun.toml configuration file, this is where the connection strings for OpenTSDB need to go and any other config we want to specify

<pre class="wp-block-code"><code>sudo nano /data/bosun.toml</code></pre>

[Here is a sample bosun.toml file](https://gist.github.com/tomaustin700/3b26613f09aca5a1037ba64ddabe6cfe), pay attention to the OpenTSDBConf section as you will have to specify the address of your OpenTSDB instance. I&#8217;d also recommend using [Redis](https://redis.io/) instead of Ledis in a production environment so set the RedisHost in DBConf to your Redis server. Once you are happy with your configuration save and exit.

Now we just need to create the bosunrules.conf file where our rules will be stored. We can create this as a blank file and then add configuration later through the Bosun UI.

<pre class="wp-block-code"><code>sudo nano /data/bosunrules.conf</code></pre>

Just save and exit without specifying anything.

**Create a supervisor file to run Bosun**

Now we need to configure supervisor to run Bosun, this is just a case of creating a config file for Bosun specifying a few paths.

<pre class="wp-block-code"><code>sudo nano /etc/supervisor/conf.d/bosun.conf</code></pre>

The [following configuration](https://gist.github.com/tomaustin700/17a2371b4a2f6ca26375a1f7a4df5e9d) should work if you have followed along exactly.

Now we just need to inform supervisor that we have a new configuration and hopefully Bosun will start.

<pre class="wp-block-code"><code>sudo supervisorctl reread</code></pre>

<pre class="wp-block-code"><code>sudo supervisorctl update</code></pre>

You can check Bosun is running by running the following command, it should respond with &#8216;bosun RUNNING&#8217; along with some uptime data and the pid. 

<pre class="wp-block-code"><code>sudo supervisorctl</code></pre>

Part 2 or this series will cover installing [TSDBRelay](https://godoc.org/bosun.org/cmd/tsdbrelay). Instead of directly sending data to Bosun we can send our data through TSDBRelay which can forward the data to opentsdb and then also to Bosun for indexing, this get&#8217;s Bosun out of the &#8216;critical path&#8217;.