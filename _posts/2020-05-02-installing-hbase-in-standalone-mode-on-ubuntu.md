---
id: 644
title: Installing HBase in Standalone mode on Ubuntu
date: 2020-05-02T18:13:58+01:00
author: tom
layout: post
guid: http://3.10.198.211/?p=644
permalink: /2020/05/02/installing-hbase-in-standalone-mode-on-ubuntu/
image: /wp-content/uploads/2020/05/hbase_logo_with_orca_large.png
categories:
  - apache
tags:
  - hbase
  - ubuntu
---
[HBase](https://hbase.apache.org/) can be installed in three modes: standalone, pseudo-distributed and distributed &#8211; each mode has uses and advantages and disadvantages and slightly different install steps because of it. This article will guide you through the installation of HBase in standalone mode on Ubuntu 18.04. 

So why use standalone mode? Firstly standalone mode is not recommended for production use however for development or non-important data standalone mode can be ideal. Standalone mode uses the local file system for data storage instead of Hadoop and because of this is pretty quick and easy to get running and you only need one server. Let&#8217;s begin!

The first think we are going to do is install OpenJDK 8. At time of writing this is the recommended OpenJDK version for Hbase 2.1+ however if this changes you can find supported versions [here](https://hbase.apache.org/book.html#java). SSH into your Ubuntu box and install OpenJDK 8.

<pre class="wp-block-code"><code>sudo apt install openjdk-8-jdk</code></pre>

Next we are going to make a directory for our HBase data. By default HBase in standalone mode will store data in a temporary directory that is wiped on reboot which is not what we want.

<pre class="wp-block-code"><code>sudo mkdir -p /var/hbase</code></pre>

Now we can get on with installing HBase. Head over to the [Apache download mirrors site](https://www.apache.org/dyn/closer.lua/hbase/) and click the recommended mirror. Once there we want to find the latest HBase version and locate the version ending in -bin.tar.gz. Once we have that we can use wget to download it.

<pre class="wp-block-code"><code>wget https://www.mirrorservice.org/sites/ftp.apache.org/hbase/2.2.4/hbase-2.2.4-bin.tar.gz</code></pre>

Now extract the downloaded file

<pre class="wp-block-code"><code>tar xzvf hbase-2.2.4-bin.tar.gz</code></pre>

and move into the extracted directory

<pre class="wp-block-code"><code>cd hbase-2.2.4</code></pre>

We now need to do a little bit of configuration, let&#8217;s start by setting the JAVA_HOME variable. You&#8217;re going to need to know your Java installation directory, for me this is /usr/lib/jvm/java-8-openjdk-amd64/jre however it may be different for you. Once located we need to edit conf/hbase-env.sh

<pre class="wp-block-code"><code>sudo nano conf/hbase-env.sh</code></pre>

Set JAVA_HOME to your Java installation directory and then save and exit

<pre class="wp-block-code"><code>export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre</code></pre>

Now we need to tell HBase about the directory we created earlier so let&#8217;s also edit conf/hbase-site.xml and set that directory.

<pre class="wp-block-code"><code>sudo nano conf/hbase-site.xml</code></pre>

Set hbase.tmp.dir to the directory we created earlier. Your configuration should look something like this:

<pre class="wp-block-code"><code>&lt;configuration&gt;
  &lt;property&gt;
    &lt;name&gt;hbase.tmp.dir&lt;/name&gt;
    &lt;value&gt;/var/hbase&lt;/value&gt;
  &lt;/property&gt;
&lt;/configuration&gt;</code></pre>

Save and exit.

That&#8217;s pretty much it when it comes to installing HBase. We should be able to start HBase by running the start-hbase script

<pre class="wp-block-code"><code>sudo ./bin/start-hbase.sh</code></pre>

After a few minutes HBase should have started, you can test this by connecting to it using the HBase shell

<pre class="wp-block-code"><code>./bin/hbase shell</code></pre>

If everything has gone to plan after a few seconds you will be at the HBase shell.

Well done so far! The only thing left is to configure HBase to start on system startup. Our current config requires the start-hbase.sh script to be manually ran which is not ideal. We are going to be using [Supervisor](http://supervisord.org/) to auto run our start script when the system starts. Install Supervisor using the following command:

<pre class="wp-block-code"><code>sudo apt-get install supervisor</code></pre>

Now we need to make a conf file so Supervisor knows about HBase and can start it for us.

<pre class="wp-block-code"><code>sudo nano /etc/supervisor/conf.d/hbase.conf</code></pre>

Enter the following configuration. One thing to be aware of is that the hbase-x.x.x directory may be in a slightly different directory for you so make sure you enter the correct path.

<pre class="wp-block-code"><code>&#91;program:hbase]
command=bash -c "/home/tom/hbase-2.2.4/bin/start-hbase.sh"
priority=100
stdout_logfile=/var/log/hbase.out.log
stderr_logfile=/var/log/hbase.err.log
autostart=true
autorestart=true</code></pre>

Save and exit like usual and run the following commands to tell Supervisor that there is a new configuration and to run it.

<pre class="wp-block-code"><code>sudo supervisorctl reread</code></pre>

<pre class="wp-block-code"><code>sudo supervisorctl update</code></pre>

You should now be able to access the HBase web UI! If you aren&#8217;t sure what port it&#8217;s running on you can cd into the HBase logs directory and run the following command to show the ports HBase is running on:

<pre class="wp-block-code"><code>grep 'Jetty' *</code></pre>

In my case the UI can be accessed on 16030. If you can&#8217;t access the web UI you can tail the hbase error log for hints as to what&#8217;s going wrong.

<pre class="wp-block-code"><code>tail /var/log/hbase.err.log</code></pre>

Remember that standalone is not recommended for production environments and you risk data loss by running it in this configuration. Thanks!