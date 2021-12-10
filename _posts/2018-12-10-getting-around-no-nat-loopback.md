---
id: 124
title: Getting Around No NAT Loopback
date: 2018-12-10T21:27:39+00:00
author: tom
layout: post
guid: http://3.8.252.104/?p=124
permalink: /2018/12/10/getting-around-no-nat-loopback/
categories:
  - homelab
---
I have been hosting this site internally for about a year now on a virtualised nginx instance, getting it up and running was a breeze but something always bugged me &#8211; I couldn&#8217;t access the site from within the local network.

After a bit of digging around I found the problem&#8230; my router didn&#8217;t support NAT Loopback. Without NAT Loopback if I typed my url into the address bar it would take me to my routers login page. Now I could work around this by using the internal ip of the nginx instance however because the site address is&nbsp;http://tomaustin.xyz WordPress seems to get a bit weird if you don&#8217;t access it from that address (certain things don&#8217;t render correctly, images don&#8217;t load).

After about a year I decided enough was enough and I was going to sort this out.

Internally I host my own domain and DNS server so I knew with a bit of DNS magic I could get this working. Normally DNS servers cause nothing but problems and you can spend hours trying to get something working before you realise it&#8217;s a DNS issue, it&#8217;s always DNS! This time DNS was on my side.

First off open DNS Manger on your DNS server and expand Forward Lookup Zones, then right click and select New Zone<figure class="wp-block-image">

<img loading="lazy" width="358" height="224" src="http://tomaustin.xyz/wp-content/uploads/2018/12/image-3.png" alt="" class="wp-image-128" srcset="https://tomaustin.xyz/wp-content/uploads/2018/12/image-3.png 358w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-3-300x188.png 300w" sizes="(max-width: 358px) 100vw, 358px" /> </figure> 

Click through the wizard and select Primary zone and select &#8220;To all DNS servers running on domain controllers in this domain:&#8221; (I think any of the options will work but this is what I selected). Now enter your zone name.<figure class="wp-block-image">

<img loading="lazy" width="528" height="414" src="http://tomaustin.xyz/wp-content/uploads/2018/12/image-1.png" alt="" class="wp-image-126" srcset="https://tomaustin.xyz/wp-content/uploads/2018/12/image-1.png 528w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-1-300x235.png 300w" sizes="(max-width: 528px) 100vw, 528px" /> </figure> 

Click to Allow only secure dynamic updates and finish. Your domain should now show in the Forward Lookup Zone of the DNS Server.<figure class="wp-block-image">

<img loading="lazy" width="194" height="185" src="http://tomaustin.xyz/wp-content/uploads/2018/12/image-7.png" alt="" class="wp-image-133" /> </figure> 

Now all we need to do is add a few A records and we are done, click on your zone then right click and select New Host (A or AAAA).<figure class="wp-block-image">

<img loading="lazy" width="351" height="424" src="http://tomaustin.xyz/wp-content/uploads/2018/12/image-4.png" alt="" class="wp-image-129" srcset="https://tomaustin.xyz/wp-content/uploads/2018/12/image-4.png 351w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-4-248x300.png 248w" sizes="(max-width: 351px) 100vw, 351px" /> </figure> 

The first record we want to add just wants to have a blank name, add the ip address of your web server.<figure class="wp-block-image">

<img loading="lazy" width="433" height="410" src="http://tomaustin.xyz/wp-content/uploads/2018/12/image-5.png" alt="" class="wp-image-130" srcset="https://tomaustin.xyz/wp-content/uploads/2018/12/image-5.png 433w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-5-300x284.png 300w" sizes="(max-width: 433px) 100vw, 433px" /> <figcaption>  
</figcaption></figure> 

Once that has added add a second entry but enter www in the Name field, use the same IP address.<figure class="wp-block-image">

<img loading="lazy" width="430" height="480" src="http://tomaustin.xyz/wp-content/uploads/2018/12/image-6.png" alt="" class="wp-image-131" srcset="https://tomaustin.xyz/wp-content/uploads/2018/12/image-6.png 430w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-6-269x300.png 269w" sizes="(max-width: 430px) 100vw, 430px" /> </figure> 

That should be it!