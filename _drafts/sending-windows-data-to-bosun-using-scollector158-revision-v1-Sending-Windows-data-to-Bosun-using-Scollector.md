---
id: 277
title: Sending Windows data to Bosun using Scollector
date: 2019-04-07T17:10:00+01:00
author: tom
layout: revision
guid: http://3.10.198.211/2019/04/07/158-revision-v1/
permalink: /?p=277
---
I have previously written about how to send .net application data to [Bosun](https://bosun.org/) however what if we wanted to send data from our OS to Bosun? Well for that we need [Scollector](http://bosun.org/scollector/)!

  


The first thing we need to do is download the correct binary for our OS, these can all be found on the Scollector homepage however for this article we will just be focusing on the Windows version. Once downloaded create a folder for Scollector on C:\ (this can be anywhere really but this makes it nice and easy). One thing that also makes things easy is to rename the exe to just Scollector. Put the exe in the directory you have just made. Now open a Powershell session as administrator and run the following command to install Scollector as a Windows service.

<pre class="wp-block-code"><code>c:\Scollector\Scollector.exe -winsvc="install"</code></pre>

Now we need to modify the service so it can connect to our Bosun server, for that we need to open the registry editor (run > regedit) and navigate to&nbsp;HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Services\scollector. We should see an entry called ImagePath which points to our Scollector exe.<figure class="wp-block-image">

<img loading="lazy" width="760" height="153" src="http://tomaustin.xyz/wp-content/uploads/2018/12/image-12.png" alt="" class="wp-image-159" srcset="https://tomaustin.xyz/wp-content/uploads/2018/12/image-12.png 760w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-12-300x60.png 300w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-12-720x145.png 720w" sizes="(max-width: 760px) 100vw, 760px" /> </figure> 

We now need to right click on this entry and select modify. Add the following parameter to the Value data (the ip and port correspond to your Bosun server).<figure class="wp-block-image">

<img loading="lazy" width="450" height="227" src="http://tomaustin.xyz/wp-content/uploads/2018/12/image-13.png" alt="" class="wp-image-160" srcset="https://tomaustin.xyz/wp-content/uploads/2018/12/image-13.png 450w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-13-300x151.png 300w" sizes="(max-width: 450px) 100vw, 450px" /> </figure> 

Now all we need to do is open Services (run > services.msc), find scollector and start the service. If everything has worked the service should start and its status should be &#8216;Running&#8217;. If the service won&#8217;t start then check the Windows Event Viewer and navigate to Windows Logs > Application.<figure class="wp-block-image">

<img loading="lazy" width="848" height="56" src="http://tomaustin.xyz/wp-content/uploads/2018/12/image-14.png" alt="" class="wp-image-161" srcset="https://tomaustin.xyz/wp-content/uploads/2018/12/image-14.png 848w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-14-300x20.png 300w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-14-768x51.png 768w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-14-720x48.png 720w" sizes="(max-width: 848px) 100vw, 848px" /> </figure> 

If we now open our Bosun server and click Items our host should be listed under Hosts. Once clicked we should be greeted with every metric we could possibly need for our OS!<figure class="wp-block-image">

<img loading="lazy" width="1179" height="718" src="https://i1.wp.com/www.tomaustin.xyz/wp-content/uploads/2018/12/image-15.png?fit=640%2C390&ssl=1" alt="" class="wp-image-162" srcset="https://tomaustin.xyz/wp-content/uploads/2018/12/image-15.png 1179w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-15-300x183.png 300w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-15-768x468.png 768w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-15-1024x624.png 1024w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-15-720x438.png 720w" sizes="(max-width: 1179px) 100vw, 1179px" /> </figure> 

Clicking &#8216;Available Metrics&#8217; will show us everything that is being sent and we can use any of these to setup alerts.