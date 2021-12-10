---
id: 721
title: Installing and configuring Apache Ambari to manage Hadoop and HBase
date: 2021-02-07T15:11:30+00:00
author: tom
layout: post
guid: http://tomaustin.xyz/?p=721
permalink: /?p=721
categories:
  - Uncategorised
---
## What is Ambari?

Apachi Ambari is an administration tool to help manage, monitor and configure Hadoop clusters using a web UI. If you&#8217;ve ever tried to manually install Hadoop or Hbase without Ambari you may know that it can be quite difficult to configure &#8211; Ambari changes all this and also adds the ability to easily monitor the Hadoop cluster and install various applications on top of Hadoop such as Hbase.

## Installation Steps

I&#8217;m going to be setting up Ambari within Microsoft Azure however this tutorial should work for any environment, if you do plan on setting up your Ambari instance in Azure you may be better off using their managed Hadoop cluster called [HDInsight](https://docs.microsoft.com/en-us/azure/hdinsight/) but you are free to manually set everything up if you wish.

To install Ambari you are going to need at least three virtual machines but I&#8217;d recommend four or more. One virtual machine will be the Ambari server with the rest being Ambari agents. I&#8217;m going to be running Ubuntu 18.04 on each virtual machine however the steps should be fairly similar for any version of Ubuntu. If you&#8217;re creating your virtual machines in Azure make sure all VM&#8217;s are in the same virtual network.

Once you have all your virtual machines SSH into the first one, this is going to be the Ambari server. Let&#8217;s start by switching to the root user.

<pre class="wp-block-code"><code>sudo su</code></pre>

Next we need to add the Ambari repository file to /etc/apt/sources.list.d

<pre class="wp-block-code"><code>wget -O /etc/apt/sources.list.d/ambari.list http://public-repo-1.hortonworks.com/ambari/ubuntu16/2.x/updates/2.7.3.0/ambari.list</code></pre>