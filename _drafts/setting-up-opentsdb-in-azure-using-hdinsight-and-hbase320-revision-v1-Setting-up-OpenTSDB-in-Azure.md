---
id: 351
title: Setting up OpenTSDB in Azure
date: 2019-05-12T15:37:25+01:00
author: tom
layout: revision
guid: http://3.10.198.211/2019/05/12/320-revision-v1/
permalink: /?p=351
---
 

This article will guide you through the steps required to setup OpenTSDB in Azure, we are going to be using Azure HDInsight to host our Hbase cluster and then setup OpenTSDB to connect to the cluster.

The first thing we need to do is open the Azure portal and navigate to resource groups, once there create a new resource group for OpenTSDB.<figure class="wp-block-image">

<img loading="lazy" width="718" height="342" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image.png" alt="" class="wp-image-326" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image.png 718w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-300x143.png 300w" sizes="(max-width: 718px) 100vw, 718px" /> </figure> 

Now we need to create a virtual network for Hbase and OpenTSDB to talk to each other. Navigate to Virtual networks in azure and hit &#8216;Add&#8217; and set the name for your network and assign it to our resource group, all of the other settings can be left at default values.

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-1.png" alt="" class="wp-image-327" width="249" height="546" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-1.png 321w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-1-137x300.png 137w" sizes="(max-width: 249px) 100vw, 249px" /></figure>
</div>

Once the virtual network has been created we can get to setting up our Hbase cluster. Open &#8216;HDInsight clusters&#8217; in Azure and press &#8216;Add&#8217;, give your cluster a name, set cluster login password, assign it to the resource group we previously created and set the location. Click &#8216;Cluster type&#8217;, set this to Hbase and then press &#8216;Select&#8217; at the bottom.

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-3.png" alt="" class="wp-image-329" width="580" height="251" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-3.png 889w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-3-300x130.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-3-768x333.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-3-720x312.png 720w" sizes="(max-width: 580px) 100vw, 580px" /></figure>
</div>

Before hitting &#8216;Next&#8217; toggle the wizard mode from &#8216;Quick create&#8217; to &#8216;Custom (size, settings, apps), this will allow up to connect Hbase to our virtual network.

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="305" height="95" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-2.png" alt="" class="wp-image-328" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-2.png 305w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-2-300x93.png 300w" sizes="(max-width: 305px) 100vw, 305px" /></figure>
</div>

Press &#8216;Next&#8217; and you should now be on the &#8216;Security + networking&#8217; page, under &#8216;Virtual network&#8217; select the network we previously created. Without setting this your Hbase cluster and OpenTSDB will not be able to communicate.

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="322" height="191" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-4.png" alt="" class="wp-image-331" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-4.png 322w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-4-300x178.png 300w" sizes="(max-width: 322px) 100vw, 322px" /></figure>
</div>

Proceed onto the &#8216;Storage&#8217; page and create a new storage account for Hbase. One thing to be aware of is that Azure won&#8217;t check that there isn&#8217;t already a Storage account with the name you specify and then will fail to deploy the cluster later on. Name it something unique before pressing &#8216;Next&#8217;.

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="291" height="237" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-5.png" alt="" class="wp-image-332" /></figure>
</div>

You don&#8217;t need to set anything on the &#8216;Applications&#8217; page so just press &#8216;Next&#8217;. Once you are on the &#8216;Cluster size&#8217; page set the cluster to the specifications you require before moving onto the &#8216;Script actions&#8217; page.

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-6.png" alt="" class="wp-image-333" width="288" height="315" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-6.png 318w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-6-274x300.png 274w" sizes="(max-width: 288px) 100vw, 288px" /></figure>
</div>

You don&#8217;t need any script actions so move onto the summary. Check everything is as you configured (double check the Virtual network is set correctly) and hit &#8216;Create&#8217;.

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="783" height="663" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-7.png" alt="" class="wp-image-334" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-7.png 783w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-7-300x254.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-7-768x650.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-7-720x610.png 720w" sizes="(max-width: 783px) 100vw, 783px" /></figure>
</div>

It will take Azure about 20 minutes to build and configure the cluster so while it is doing that we can move onto creating our OpenTSDB server and doing some basic configuration.

Create a new virtual machine in our resource group. I&#8217;m going to be using Ubuntu 18.04 LTS for my image and Standard B1ms for my vm size but these can be set to your liking. Make sure the virtual network is set to network we previously created, if you are wanting to configure OpenTSDB outside of that network then add inbound port rules for 22 (SSH) and 4242 (OpenTSDB port).

Once Azure has created our VM SSH into it, download the OpenTSDB package and install OpenTSDB.

<pre class="wp-block-code"><code>wget "https://github.com/OpenTSDB/opentsdb/releases/download/v2.4.0/opentsdb-2.4.0_all.deb"</code></pre>

<pre class="wp-block-code"><code>sudo dpkg -i  opentsdb-2.4.0_all.deb </code></pre>

Before starting OpenTSDB we need to install some pre-requisites and update the OpenTSDB config file. Let&#8217;s start by installing gnuplot and the Java runtime environment.

<pre class="wp-block-code"><code>sudo apt-get update</code></pre>

<pre class="wp-block-code"><code>sudo apt-get install -y gnuplot </code></pre>

<pre class="wp-block-code"><code>sudo apt install default-jre </code></pre>

Once that&#8217;s finished we need to edit the opentsdb.conf file and set a few things up.

<pre class="wp-block-code"><code>sudo nano /etc/opentsdb/opentsdb.conf</code></pre>

Before we set anything in here we need to get the IP address of our zookeeper instances. Hopefully by now Azure has created our Hbase cluster so open HDInsight clusters in Azure and click on your cluster. In the &#8216;Overview&#8217; page there should be a url for your cluster, click it and login. <figure class="wp-block-image">

<img loading="lazy" width="695" height="268" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-9.png" alt="" class="wp-image-338" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-9.png 695w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-9-300x116.png 300w" sizes="(max-width: 695px) 100vw, 695px" /> </figure> 

Once logged in you should be at the Ambari dashboard. Ambari is a tool by apache and is used to monitor and manage our cluster. Click &#8216;Hosts&#8217; at the top. Don&#8217;t worry if you see some red exclamation marks, Azure probably hasn&#8217;t finished its configuration yet. We should now see our zookeeper instances listed at the bottom along with their IP addresses.<figure class="wp-block-image">

<img loading="lazy" width="636" height="327" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-10.png" alt="" class="wp-image-339" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-10.png 636w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-10-300x154.png 300w" sizes="(max-width: 636px) 100vw, 636px" /> </figure> 

Go back to your OpenTSDB server and enter those IP Addresses as a comma separated list next to tsd.storage.hbase.zk_quorum (make sure you uncomment the line by removing the #).<figure class="wp-block-image">

<img loading="lazy" width="614" height="67" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-11.png" alt="" class="wp-image-341" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-11.png 614w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-11-300x33.png 300w" sizes="(max-width: 614px) 100vw, 614px" /> </figure> 

Uncomment the tsd.storage.hbase.zk_basedir line and set this to /hbase-unsecure.<figure class="wp-block-image">

<img loading="lazy" width="678" height="57" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-12.png" alt="" class="wp-image-342" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-12.png 678w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-12-300x25.png 300w" sizes="(max-width: 678px) 100vw, 678px" /> </figure> 

You can also uncomment tsd.core.auto\_create\_metrics and set it to true if you want to turn on autometrics.<figure class="wp-block-image">

<img loading="lazy" width="619" height="81" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-13.png" alt="" class="wp-image-343" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-13.png 619w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-13-300x39.png 300w" sizes="(max-width: 619px) 100vw, 619px" /> </figure> 

Save and exit (ctrl+o, enter, ctrl+x).

Before starting OpenTSDB we need to SSH into our Hbase cluster and create a few tables for OpenTSDB to use. Go back to Azure and select the Hbase cluster, under &#8216;Settings&#8217; select &#8216;SSH + Cluster login&#8217; and set the &#8216;Hostname&#8217; field to the only available option. SSH into the cluster using the endpoint provided. Once logged in run the following command to get to the Hbase shell.

<pre class="wp-block-code"><code>hbase shell</code></pre>

After a few seconds you should be at the Hbase shell.<figure class="wp-block-image">

<img loading="lazy" width="636" height="397" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-14.png" alt="" class="wp-image-346" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-14.png 636w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-14-300x187.png 300w" sizes="(max-width: 636px) 100vw, 636px" /> </figure> 

Run the following commands to create the tables for OpenTSDB.

<pre class="wp-block-code"><code>create 'tsdb-uid',
  {NAME => 'id', COMPRESSION => 'NONE', BLOOMFILTER => 'ROW'},
  {NAME => 'name', COMPRESSION => 'NONE', BLOOMFILTER => 'ROW'}</code></pre>

<pre class="wp-block-code"><code>create 'tsdb',
  {NAME => 't', VERSIONS => 1, COMPRESSION => 'NONE', BLOOMFILTER => 'ROW'}</code></pre>

<pre class="wp-block-code"><code>create 'tsdb-tree',
  {NAME => 't', VERSIONS => 1, COMPRESSION => 'NONE', BLOOMFILTER => 'ROW'}</code></pre>

<pre class="wp-block-code"><code>create 'tsdb-meta',
  {NAME => 'name', COMPRESSION => 'NONE', BLOOMFILTER => 'ROW'}</code></pre>

Once done you can use the list command to check all your tables were created successfully.

<pre class="wp-block-code"><code>list</code></pre><figure class="wp-block-image">

<img loading="lazy" width="586" height="186" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-15.png" alt="" class="wp-image-347" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-15.png 586w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-15-300x95.png 300w" sizes="(max-width: 586px) 100vw, 586px" /> </figure> 

Now go back to your OpenTSDB and start OpenTSDB.

<pre class="wp-block-code"><code>sudo service opentsdb start</code></pre>

You can check everything started correctly by tailing the opentsdb log file.

<pre class="wp-block-code"><code>tail /var/log/opentsdb/opentsdb.log</code></pre>

You should see a &#8216;Ready to serve on&#8230;&#8217; message if everything has gone to plan.<figure class="wp-block-image">

<img loading="lazy" width="844" height="183" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-16.png" alt="" class="wp-image-348" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-16.png 844w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-16-300x65.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-16-768x167.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-16-720x156.png 720w" sizes="(max-width: 844px) 100vw, 844px" /> </figure> 

You should now be able to navigate to your OpenTSDB instance on port 4242!<figure class="wp-block-image">

<img loading="lazy" width="1044" height="359" src="https://i0.wp.com/tomaustin.xyz/wp-content/uploads/2019/05/image-17.png?fit=640%2C220&ssl=1" alt="" class="wp-image-349" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-17.png 1044w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-17-300x103.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-17-768x264.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-17-1024x352.png 1024w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-17-720x248.png 720w" sizes="(max-width: 1044px) 100vw, 1044px" /> </figure>