---
id: 434
title: 'Installing Bosun in a Production Environment  &#8211; Part 3'
date: 2019-06-01T15:42:56+01:00
author: tom
layout: post
guid: http://3.10.198.211/?p=434
permalink: /2019/06/01/installing-bosun-in-a-production-environment-part-3/
image: /wp-content/uploads/2019/06/bosun.png
categories:
  - bosun
  - devops
---
Parts 1, 2, 3 and 4 of this series can be found at the following locations: [Part 1](http://tomaustin.xyz/2019/05/05/installing-bosun-in-a-production-environment/), [Part 2](http://tomaustin.xyz/2019/05/11/installing-bosun-in-a-production-environment-part-2/), [Part 3](http://tomaustin.xyz/2019/06/01/installing-bosun-in-a-production-environment-part-3/) and [Part 4](http://tomaustin.xyz/2019/06/08/installing-bosun-in-a-production-environment-part-4/).

This is the third article in this series covering how to install [Bosun](http://bosun.org) in a production environment. The first two articles covered the [installation of Bosun](http://tomaustin.xyz/2019/05/05/installing-bosun-in-a-production-environment/) and then [configuring TSDBRelay to relay and aggregate data](http://tomaustin.xyz/2019/05/11/installing-bosun-in-a-production-environment-part-2/) for us. This article is going to explore how we can load balance between multiple TSDBRelay instances with [HAProxy](http://www.haproxy.org/).

Unfortunately Bosun does not support being ran in a load balanced configuration which is not great when we need high availability; [I have discussed this with Bosun&#8217;s creator Kyle Brandt](https://twitter.com/tomaustin700/status/1121780945634373633) and there is also a [Pull Request](https://github.com/bosun-monitor/bosun/pull/2345) proposing changes to support a Bosun cluster but this is yet to be merged in. So what do we do if we don&#8217;t want to worry about Bosun being there when we send data? As discussed in the previous article we can utilise TSDBRelay to get Bosun out of the &#8216;crtical path&#8217; by forwarding the data directly to our OpenTSDB server however because our applications will send data to TSDBRelay instead of Bosun we need to make TSDBRelay highly available. Fortunately TSDBRelay does support being ran in a load balanced configuration so we are going to add a second (you can add as many as you wish) TSDBRelay instance and then use HAProxy to load balance between them.

**Create second TSDBRelay instance**

Let&#8217;s start by creating a second TSDBRelay instance! I&#8217;m not going to go into too much detail about how to do this as the process is covered in depth in the [previous article](http://tomaustin.xyz/2019/05/11/installing-bosun-in-a-production-environment-part-2/). The basic process is create an Ubuntu instance (or other distro of your liking), install Go 1.11.2, pull TSDBRelay from GitHub and finally install and configure Supervisor to run TSDBRelay. Ideally I&#8217;d recommend having at least three instances but you are really free to add as many as you want.

**Install and Confiure HAProxy**

Create a Linux VM using your distro of choice (as usual I am using Ubuntu 18.04), SSH into it and run apt-get update to make sure everything is up-to-date.

<pre class="wp-block-code"><code>sudo apt-get update</code></pre>

Once done let&#8217;s install HAProxy

<pre class="wp-block-code"><code>sudo apt-get install haproxy</code></pre>

Once HAProxy is downloaded and installed we need to enable the init script, this will allow HAProxy to autostart. Run the following command to edit the script with Nano. 

<pre class="wp-block-code"><code>sudo nano /etc/default/haproxy</code></pre>

Append Enabled=1 to the file and then save and close (ctrl+o, enter, ctrl+x).

Now let&#8217;s open the config file using nano and start configuring our load balancer.

<pre class="wp-block-code"><code>sudo nano /etc/haproxy/haproxy.cfg</code></pre>

Firstly let&#8217;s set the default mode to tcp and option to tcplog. Update these in the &#8216;defaults&#8217; section.

<pre class="wp-block-code"><code>        mode    tcp
        option  tcplog</code></pre>

At the bottom of the file we are now going to add a frontend, this will instruct HAProxy what to listen and which backend to use. Append the following to the bottom of the config file.

<pre class="wp-block-code"><code>frontend tsdbrelay
   bind 0.0.0.0:5252
   default_backend tsdbrelay-backend
</code></pre>

This config is just telling HAProxy to listen on port 5252 and use the backend &#8216;tsdbrelay-backend&#8217; which we will now specify.

<pre class="wp-block-code"><code>backend tsdbrelay-backend
   balance roundrobin
   mode tcp
   server tsdbrelay1 10.0.0.8:5252 check
   server tsdbrelay2 10.0.0.10:5252 check</code></pre>

Set the tsdbrelay1 and 2 ip addresses and ports to the ip address and ports of your TSDBRelay instances, if you are using more than two instance then keep adding servers to the file until they are all specified.

If your HAPRoxy instance is not externally facing then you could enable the stats dashboard by adding the following configuration to the configuration file, this is not really necessary but it&#8217;s nice to see what HAProxy is doing. Make sure you replace admin:pass with strong credentials.

<pre class="wp-block-code"><code>listen  stats
        bind :80
        mode            http
        log             global

        maxconn 10

        clitimeout      100s
        srvtimeout      100s
        contimeout      100s
        timeout queue   100s

        stats enable
        stats hide-version
        stats refresh 30s
        stats show-node
        stats auth admin:pass
        stats uri  /haproxy?stats
</code></pre>

A complete copy of my HAPRoxy configuration file can be found [here](https://gist.github.com/tomaustin700/5bcb731abb1427cf810be96a3175c772).

Once you are happy with your configuration save and exit and then restart HAProxy

<pre class="wp-block-code"><code>sudo service haproxy restart</code></pre>

Now instead of sending data to TSDBRelay we can send data to HAProxy which will then load balance the data between our TSDBRelay instances.

The [next and final article](http://tomaustin.xyz/2019/06/08/installing-bosun-in-a-production-environment-part-4/) will cover installing [scollector](https://bosun.org/scollector/) to gather data about our Bosun, TSDBRelay and HAProxy instances and then send that data to Bosun.