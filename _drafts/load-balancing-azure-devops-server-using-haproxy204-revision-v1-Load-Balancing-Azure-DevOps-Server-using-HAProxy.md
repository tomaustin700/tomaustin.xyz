---
id: 284
title: Load Balancing Azure DevOps Server using HAProxy
date: 2019-04-12T17:28:31+01:00
author: tom
layout: revision
guid: http://3.10.198.211/2019/04/12/204-revision-v1/
permalink: /?p=284
---
I have used and managed [Azure DevOps Server](https://azure.microsoft.com/en-us/services/devops/server/) (previous Team Foundation Server) for several years and now that my daily role includes doing more and more &#8216;DevOps&#8217; stuff I decided it was time to look into load balancing ADS. I&#8217;ve previously looked at this a few years ago and found one out-of-date tutorial which used [Windows Network Load Balancing](https://docs.microsoft.com/en-us/windows-server/networking/technologies/network-load-balancing) which did get it working however I&#8217;ve been wanting to work with [HAProxy](http://www.haproxy.org/) for a while and now seemed like the perfect opportunity.

This tutorial will cover installing ADS and pointing it to an existing SQL Server and then setting up HAProxy to act as a load balancer. Before starting you will need a configured Domain Controller and an existing SQL Server instance connected to the domain. We will configure ADS to have multiple application tiers and then use HAProxy in a round-robin configuration to load balance between them. You will need two VM&#8217;s (or more if you want higher availability) for your ADS Application Tiers, these also need to be joined to the domain. For HAProxy I will be using Ubuntu but pretty much any Linux distro you prefer will be capable. In total you should have something like this (everything is running Windows Server 2016 apart from HAP which is Ubuntu 18.04 LTS):



DC &#8211; Domain Controller

SQL &#8211; SQL Server

ADS1 &#8211; Azure DevOps Server Application Tier 1

ADS2 &#8211; Azure DevOps Server Application Tier 2

HAP &#8211; HAProxy server

## Configuring Azure DevOps Server Application Tiers

Now that you have your infrastructure created we can get Azure DevOps Server installed. Start by remoting onto ADS1, downloading [Azure DevOps Server](https://azure.microsoft.com/en-us/services/devops/server/) and starting the installation. After it has acquired the components it needs you should be at greeted with this prompt:<figure class="wp-block-image">

<img loading="lazy" width="912" height="640" src="http://tomaustin.xyz/wp-content/uploads/2019/04/startwizard.png" alt="" class="wp-image-212" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/startwizard.png 912w, https://tomaustin.xyz/wp-content/uploads/2019/04/startwizard-300x211.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/startwizard-768x539.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/startwizard-720x505.png 720w" sizes="(max-width: 912px) 100vw, 912px" /> </figure> 

Hit &#8216;Start Wizard&#8217; and then hit Next, when prompted about Deployment Type select &#8216;This is a new Azure DevOps Server delpoyment&#8217;.<figure class="wp-block-image">

<img loading="lazy" width="924" height="632" src="http://tomaustin.xyz/wp-content/uploads/2019/04/deploy.png" alt="" class="wp-image-213" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/deploy.png 924w, https://tomaustin.xyz/wp-content/uploads/2019/04/deploy-300x205.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/deploy-768x525.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/deploy-720x492.png 720w" sizes="(max-width: 924px) 100vw, 924px" /> </figure> 

Hit next again until prompted about deployment scenario, I recommend choosing Advanced as it will give you a bit more control over how things are configured.<figure class="wp-block-image">

<img loading="lazy" width="920" height="639" src="http://tomaustin.xyz/wp-content/uploads/2019/04/advanced.png" alt="" class="wp-image-214" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/advanced.png 920w, https://tomaustin.xyz/wp-content/uploads/2019/04/advanced-300x208.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/advanced-768x533.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/advanced-720x500.png 720w" sizes="(max-width: 920px) 100vw, 920px" /> </figure> 

Next you will be prompted to choose your language and then database configuration. When prompted to specify the SQL Server Instance enter the name of your SQL Server. I recommend you hit &#8216;Test&#8217; to ensure everything is working correctly. If everything is good you should get a green tick.<figure class="wp-block-image">

<img loading="lazy" width="926" height="634" src="http://tomaustin.xyz/wp-content/uploads/2019/04/sql.png" alt="" class="wp-image-215" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/sql.png 926w, https://tomaustin.xyz/wp-content/uploads/2019/04/sql-300x205.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/sql-768x526.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/sql-720x493.png 720w" sizes="(max-width: 926px) 100vw, 926px" /> </figure> 

On the next page you will be asked about the Service Account, I have added a user to the AD for my service account however you can use a system account if you wish.<figure class="wp-block-image">

<img loading="lazy" width="921" height="634" src="http://tomaustin.xyz/wp-content/uploads/2019/04/serviceaccount.png" alt="" class="wp-image-217" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/serviceaccount.png 921w, https://tomaustin.xyz/wp-content/uploads/2019/04/serviceaccount-300x207.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/serviceaccount-768x529.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/serviceaccount-720x496.png 720w" sizes="(max-width: 921px) 100vw, 921px" /> </figure> 

On the Application Tier page I generally leave everything at its default value however if you are installing into a production environment I recommend you configure your site settings and also specify the cache location.<figure class="wp-block-image">

<img loading="lazy" width="921" height="637" src="http://tomaustin.xyz/wp-content/uploads/2019/04/aptier.png" alt="" class="wp-image-218" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/aptier.png 921w, https://tomaustin.xyz/wp-content/uploads/2019/04/aptier-300x207.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/aptier-768x531.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/aptier-720x498.png 720w" sizes="(max-width: 921px) 100vw, 921px" /> </figure> 

On the next page you can configure Search, you will need to specify a user for search. If you are installing this in a production environment I highly recommend you set the search index location to a high speed storage location (the faster the better). <figure class="wp-block-image">

<img loading="lazy" width="1013" height="788" src="http://tomaustin.xyz/wp-content/uploads/2019/04/search.png" alt="" class="wp-image-219" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/search.png 1013w, https://tomaustin.xyz/wp-content/uploads/2019/04/search-300x233.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/search-768x597.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/search-720x560.png 720w" sizes="(max-width: 1013px) 100vw, 1013px" /> </figure> 

The next page allows you to configure reporting, I will not be enabling reporting for this installation so I will leave this unticked.<figure class="wp-block-image">

<img loading="lazy" width="1005" height="778" src="http://tomaustin.xyz/wp-content/uploads/2019/04/reporting.png" alt="" class="wp-image-221" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/reporting.png 1005w, https://tomaustin.xyz/wp-content/uploads/2019/04/reporting-300x232.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/reporting-768x595.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/reporting-720x557.png 720w" sizes="(max-width: 1005px) 100vw, 1005px" /> </figure> 

We&#8217;re nearly at the end of configuration! Make sure &#8216;Create a new team project collection&#8217; is enabled and enter a name for your collection. I am just using the default value however you can name yours whatever you want.<figure class="wp-block-image">

<img loading="lazy" width="1007" height="786" src="http://tomaustin.xyz/wp-content/uploads/2019/04/collection.png" alt="" class="wp-image-222" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/collection.png 1007w, https://tomaustin.xyz/wp-content/uploads/2019/04/collection-300x234.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/collection-768x599.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/collection-720x562.png 720w" sizes="(max-width: 1007px) 100vw, 1007px" /> </figure> 

The next page will give you chance to review all of your settings. Once you are happy with everything press &#8216;Verify&#8217;, this will run readiness checks to check everything. If you have configured Search you may be prompted to install Java 8 so tick to install if required.<figure class="wp-block-image">

<img loading="lazy" width="944" height="691" src="http://tomaustin.xyz/wp-content/uploads/2019/04/readiness.png" alt="" class="wp-image-223" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/readiness.png 944w, https://tomaustin.xyz/wp-content/uploads/2019/04/readiness-300x220.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/readiness-768x562.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/readiness-720x527.png 720w" sizes="(max-width: 944px) 100vw, 944px" /> </figure> 

I have also been prompted about having less than the recommended 4GB of memory but this is fine as this is just a test server. When you are ready hit &#8216;Configure&#8217;.<figure class="wp-block-image">

<img loading="lazy" width="942" height="693" src="http://tomaustin.xyz/wp-content/uploads/2019/04/configuring.png" alt="" class="wp-image-224" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/configuring.png 942w, https://tomaustin.xyz/wp-content/uploads/2019/04/configuring-300x221.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/configuring-768x565.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/configuring-720x530.png 720w" sizes="(max-width: 942px) 100vw, 942px" /> </figure> 

Go and grab and coffee and wait for the configuration to complete, once done you should be greeted with lots of green ticks!<figure class="wp-block-image">

<img loading="lazy" width="941" height="694" src="http://tomaustin.xyz/wp-content/uploads/2019/04/1done.png" alt="" class="wp-image-232" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/1done.png 941w, https://tomaustin.xyz/wp-content/uploads/2019/04/1done-300x221.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/1done-768x566.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/1done-720x531.png 720w" sizes="(max-width: 941px) 100vw, 941px" /> </figure> 

Press &#8216;Next&#8217; and then &#8216;Close&#8217;. Lets check everything is working by navigating to our instance. If everything has gone to plan you should be asked for credentials and then be asked to create a project.<figure class="wp-block-image">

<img loading="lazy" width="1219" height="949" src="https://i2.wp.com/tomaustin.xyz/wp-content/uploads/2019/04/connected.png?fit=640%2C498&ssl=1" alt="" class="wp-image-234" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/connected.png 1219w, https://tomaustin.xyz/wp-content/uploads/2019/04/connected-300x234.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/connected-768x598.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/connected-1024x797.png 1024w, https://tomaustin.xyz/wp-content/uploads/2019/04/connected-720x561.png 720w" sizes="(max-width: 1219px) 100vw, 1219px" /> </figure> 

Now we need to install our second application instance. Remote onto ADS2 and start the Azure DevOps Server installation, when prompted about deployment type choose &#8216;I have existing databases to use for this Azure DevOps Server deployment&#8217;.<figure class="wp-block-image">

<img loading="lazy" width="915" height="637" src="http://tomaustin.xyz/wp-content/uploads/2019/04/existingdb.png" alt="" class="wp-image-239" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/existingdb.png 915w, https://tomaustin.xyz/wp-content/uploads/2019/04/existingdb-300x209.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/existingdb-768x535.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/existingdb-720x501.png 720w" sizes="(max-width: 915px) 100vw, 915px" /> </figure> 

Enter the name of your SQL Server and then hit &#8216;List Available Databases&#8217;, select your previously created database and press &#8216;Next&#8217;.<figure class="wp-block-image">

<img loading="lazy" width="916" height="632" src="http://tomaustin.xyz/wp-content/uploads/2019/04/exitingdatabases.png" alt="" class="wp-image-240" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/exitingdatabases.png 916w, https://tomaustin.xyz/wp-content/uploads/2019/04/exitingdatabases-300x207.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/exitingdatabases-768x530.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/exitingdatabases-720x497.png 720w" sizes="(max-width: 916px) 100vw, 916px" /> </figure> 

Like when we configured ADS1 you will be asked to choose a deployment scenario, I will be choosing &#8216;Basic&#8217; however if you want to dive deeper into the settings choose &#8216;Advanced&#8217;.<figure class="wp-block-image">

<img loading="lazy" width="926" height="632" src="http://tomaustin.xyz/wp-content/uploads/2019/04/basic.png" alt="" class="wp-image-241" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/basic.png 926w, https://tomaustin.xyz/wp-content/uploads/2019/04/basic-300x205.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/basic-768x524.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/basic-720x491.png 720w" sizes="(max-width: 926px) 100vw, 926px" /> </figure> 

Next enter the service account information you previously used when configuring ADS1<figure class="wp-block-image">

<img loading="lazy" width="917" height="631" src="http://tomaustin.xyz/wp-content/uploads/2019/04/service2.png" alt="" class="wp-image-242" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/service2.png 917w, https://tomaustin.xyz/wp-content/uploads/2019/04/service2-300x206.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/service2-768x528.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/service2-720x495.png 720w" sizes="(max-width: 917px) 100vw, 917px" /> </figure> 

Configure the web settings and the cache location to your liking, I&#8217;ll be leaving mine at the default values however as previously stated if you are installing in a production environment I suggest setting the File Cache Location.<figure class="wp-block-image">

<img loading="lazy" width="919" height="633" src="http://tomaustin.xyz/wp-content/uploads/2019/04/cache.png" alt="" class="wp-image-243" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/cache.png 919w, https://tomaustin.xyz/wp-content/uploads/2019/04/cache-300x207.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/cache-768x529.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/cache-720x496.png 720w" sizes="(max-width: 919px) 100vw, 919px" /> </figure> 

Double check your settings and press verify, this shouldn&#8217;t take long as there is much less to verify this time. <figure class="wp-block-image">

<img loading="lazy" width="916" height="632" src="http://tomaustin.xyz/wp-content/uploads/2019/04/verifying.png" alt="" class="wp-image-245" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/verifying.png 916w, https://tomaustin.xyz/wp-content/uploads/2019/04/verifying-300x207.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/verifying-768x530.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/verifying-720x497.png 720w" sizes="(max-width: 916px) 100vw, 916px" /> </figure> 

You will get a message stating the Public URL does not resolve to this machine, this is expected as we will be load balancing between our application tiers. Confirm you have read the warning and would like to continue anyway and press &#8216;Configure&#8217;.<figure class="wp-block-image">

<img loading="lazy" width="915" height="631" src="http://tomaustin.xyz/wp-content/uploads/2019/04/publicurl.png" alt="" class="wp-image-247" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/publicurl.png 915w, https://tomaustin.xyz/wp-content/uploads/2019/04/publicurl-300x207.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/publicurl-768x530.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/publicurl-720x497.png 720w" sizes="(max-width: 915px) 100vw, 915px" /> </figure> 

This shouldn&#8217;t take too long so sit back and wait for it to complete.<figure class="wp-block-image">

<img loading="lazy" width="917" height="635" src="http://tomaustin.xyz/wp-content/uploads/2019/04/configuringads2.png" alt="" class="wp-image-248" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/configuringads2.png 917w, https://tomaustin.xyz/wp-content/uploads/2019/04/configuringads2-300x208.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/configuringads2-768x532.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/configuringads2-720x499.png 720w" sizes="(max-width: 917px) 100vw, 917px" /> </figure> 

Once completed press &#8216;Next&#8217; and &#8216;Close&#8217;. In the Azure DevOps Server Administration Console select &#8216;Application Tier&#8217; and scroll down to &#8216;Application Tiers&#8217;, both our instances should be visible.<figure class="wp-block-image">

<img loading="lazy" width="977" height="652" src="http://tomaustin.xyz/wp-content/uploads/2019/04/tiers.png" alt="" class="wp-image-253" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/tiers.png 977w, https://tomaustin.xyz/wp-content/uploads/2019/04/tiers-300x200.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/tiers-768x513.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/tiers-720x480.png 720w" sizes="(max-width: 977px) 100vw, 977px" /> </figure> 

We can check ADS2 is working correctly by navigating to the instance using your browser. If everything is working the ADS Projects Home should be shown.<figure class="wp-block-image">

<img loading="lazy" width="1211" height="942" src="https://i2.wp.com/tomaustin.xyz/wp-content/uploads/2019/04/ads2.png?fit=640%2C498&ssl=1" alt="" class="wp-image-254" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/ads2.png 1211w, https://tomaustin.xyz/wp-content/uploads/2019/04/ads2-300x233.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/ads2-768x597.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/ads2-1024x797.png 1024w, https://tomaustin.xyz/wp-content/uploads/2019/04/ads2-720x560.png 720w" sizes="(max-width: 1211px) 100vw, 1211px" /> </figure> 

## Installing and Configuring HAProxy

Now we can start congfiguring HAProxy! Start by SSHing into your &#8216;nix server and running apt-get update to make sure everything is up-to-date.<figure class="wp-block-image">

<img loading="lazy" width="665" height="425" src="http://tomaustin.xyz/wp-content/uploads/2019/04/hapupdate.png" alt="" class="wp-image-256" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/hapupdate.png 665w, https://tomaustin.xyz/wp-content/uploads/2019/04/hapupdate-300x192.png 300w" sizes="(max-width: 665px) 100vw, 665px" /> </figure> 

Once done run the following command to install HAProxy.

<pre class="wp-block-code"><code>sudo apt-get install haproxy</code></pre>

Once HAProxy is downloaded and installed we need to enable the init script, this will allow HAProxy to autostart. Run the following command to edit the script with Nano.

<pre class="wp-block-code"><code>sudo nano /etc/default/haproxy</code></pre>

Append Enabled=1 to the file and then save and close (ctrl+o, enter, ctrl+x).<figure class="wp-block-image">

<img loading="lazy" width="666" height="425" src="http://tomaustin.xyz/wp-content/uploads/2019/04/initenabled.png" alt="" class="wp-image-257" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/initenabled.png 666w, https://tomaustin.xyz/wp-content/uploads/2019/04/initenabled-300x191.png 300w" sizes="(max-width: 666px) 100vw, 666px" /> </figure> 

Before we start configuring HAProxy lets make a backup of the config file in case anything goes wrong.

<pre class="wp-block-code"><code>cd /etc/haproxy; sudo cp haproxy.cfg haproxy.cfg.orig</code></pre>

Now lets open the config file

<pre class="wp-block-code"><code>sudo nano /etc/haproxy/haproxy.cfg</code></pre>

Under defaults change mode to tcp and option to tcplog. This will configure HAProxy to perform layer 4 load balancing.<figure class="wp-block-image">

<img loading="lazy" width="664" height="420" src="http://tomaustin.xyz/wp-content/uploads/2019/04/layer4.png" alt="" class="wp-image-258" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/layer4.png 664w, https://tomaustin.xyz/wp-content/uploads/2019/04/layer4-300x190.png 300w" sizes="(max-width: 664px) 100vw, 664px" /> </figure> 

Now we need to configure our frontend and backends, at the end of the config file add the following configuration (enter the ip address of your HAProxy instance where I have blocked mine out). I wont go into detail what these settings do but read [this excellent article](https://www.digitalocean.com/community/tutorials/how-to-use-haproxy-as-a-layer-4-load-balancer-for-wordpress-application-servers-on-ubuntu-14-04) on Digital Ocean for more info.<figure class="wp-block-image">

<img loading="lazy" width="665" height="422" src="http://tomaustin.xyz/wp-content/uploads/2019/04/frontend.png" alt="" class="wp-image-260" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/frontend.png 665w, https://tomaustin.xyz/wp-content/uploads/2019/04/frontend-300x190.png 300w" sizes="(max-width: 665px) 100vw, 665px" /> </figure> 

Now lets add our backends! Put the ip addresses of your ADS instances where I have blocked mine out.<figure class="wp-block-image">

<img loading="lazy" width="659" height="419" src="http://tomaustin.xyz/wp-content/uploads/2019/04/backends-1.png" alt="" class="wp-image-266" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/backends-1.png 659w, https://tomaustin.xyz/wp-content/uploads/2019/04/backends-1-300x191.png 300w" sizes="(max-width: 659px) 100vw, 659px" /> </figure> 

Save and Exit and we&#8217;re almost done! Let&#8217;s just restart HAProxy to enable our changes.

<pre class="wp-block-code"><code>sudo service haproxy restart</code></pre>

If we now navigate to the IP address of our HAProxy server we should be prompted to enter our ADS credentials and then be able to see the Projects Home Page. We&#8217;re now load balancing between our ADS Instances!<figure class="wp-block-image">

<img loading="lazy" width="1208" height="940" src="https://i0.wp.com/tomaustin.xyz/wp-content/uploads/2019/04/hapads.png?fit=640%2C498&ssl=1" alt="" class="wp-image-263" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/hapads.png 1208w, https://tomaustin.xyz/wp-content/uploads/2019/04/hapads-300x233.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/hapads-768x598.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/hapads-1024x797.png 1024w, https://tomaustin.xyz/wp-content/uploads/2019/04/hapads-720x560.png 720w" sizes="(max-width: 1208px) 100vw, 1208px" /> </figure> 

## Adding a DNS Record

Entering the IP address of our HAProxy server is not the most user-friendly thing so there are a few more changes we can make to make things easier. Remote into your domain controller and open &#8216;DNS&#8217;. Expand &#8216;Forward Lookup Zones&#8217;, right click and select &#8216;New Host&#8217;. Enter &#8216;ads&#8217; in &#8216;Name&#8217; and enter the IP address of your HAProxy server in the &#8216;IP address&#8217; box.<figure class="wp-block-image">

<img loading="lazy" width="760" height="527" src="http://tomaustin.xyz/wp-content/uploads/2019/04/dns.png" alt="" class="wp-image-267" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/dns.png 760w, https://tomaustin.xyz/wp-content/uploads/2019/04/dns-300x208.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/dns-720x499.png 720w" sizes="(max-width: 760px) 100vw, 760px" /> </figure> 

Hit &#8216;Add host&#8217; and then press &#8216;Done&#8217;. You should now be able to access ADS by typing ADS in your browsers url bar.<figure class="wp-block-image">

<img loading="lazy" width="1207" height="937" src="https://i0.wp.com/tomaustin.xyz/wp-content/uploads/2019/04/adsurl.png?fit=640%2C497&ssl=1" alt="" class="wp-image-268" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/adsurl.png 1207w, https://tomaustin.xyz/wp-content/uploads/2019/04/adsurl-300x233.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/adsurl-768x596.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/adsurl-1024x795.png 1024w, https://tomaustin.xyz/wp-content/uploads/2019/04/adsurl-720x559.png 720w" sizes="(max-width: 1207px) 100vw, 1207px" /> </figure> 

Almost there! The last thing to do is change the Public URL of ADS. Remote back onto one of your ADS instances and open the Azure DevOps Server Administration Console. Click &#8216;Application Tier&#8217; and select &#8216;Change Public URL&#8217; on the right. Enter &#8216;ads&#8217; in the field and press &#8216;Test&#8217;, you should see a tick if everything is good!<figure class="wp-block-image">

<img loading="lazy" width="967" height="646" src="http://tomaustin.xyz/wp-content/uploads/2019/04/allgood.png" alt="" class="wp-image-270" srcset="https://tomaustin.xyz/wp-content/uploads/2019/04/allgood.png 967w, https://tomaustin.xyz/wp-content/uploads/2019/04/allgood-300x200.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/04/allgood-768x513.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/04/allgood-720x481.png 720w" sizes="(max-width: 967px) 100vw, 967px" /> </figure> 

That&#8217;s it! When your users browse to http://ads they are being load balanced between both of the ADS Instances. We could take things further by adding a second HAProxy instance to take over in the event our first one failed and even add a SQL cluster but both of these our outside the scope of this article.