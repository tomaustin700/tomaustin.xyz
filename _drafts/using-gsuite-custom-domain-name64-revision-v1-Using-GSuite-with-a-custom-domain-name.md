---
id: 81
title: Using GSuite with a custom domain name
date: 2018-01-22T10:31:45+00:00
author: tom
layout: revision
guid: http://3.8.252.104/2018/01/22/64-revision-v1/
permalink: /?p=81
---
I recently bought a domain name and needed email to go with it, after some research I decided to use GSuite as I&#8217;ve used Gmail for years and it has the added benefit of unlimited drive storage. This post should guide you through the steps needed to get GSuite connected to your domain and email up and running.

**1: Buy domain name**  
I used [1and1](https://www.1and1.co.uk/?ar=1) to buy my domain as they seemed the cheapest at the time. It shouldn&#8217;t matter where you got the domain from so don&#8217;t worry too much about that.

**2: Setup DNS**  
My webserver is hosted at home which means I don&#8217;t have a static IP address, to stop this being a problem I used [Dynu](https://www.dynu.com) for dynamic dns. I have used No-IP for years however they don&#8217;t allow connecting to custom domains for free. To point the domain towards dynu I created an account there and then set the nameserver in the 1&1 control panel.

<img loading="lazy" class="alignnone  wp-image-74" src="http://tomaustin.xyz/wp-content/uploads/2018/01/dns-300x127.png" alt="" width="477" height="202" srcset="https://tomaustin.xyz/wp-content/uploads/2018/01/dns-300x127.png 300w, https://tomaustin.xyz/wp-content/uploads/2018/01/dns-768x326.png 768w, https://tomaustin.xyz/wp-content/uploads/2018/01/dns-1024x434.png 1024w, https://tomaustin.xyz/wp-content/uploads/2018/01/dns-720x305.png 720w, https://tomaustin.xyz/wp-content/uploads/2018/01/dns.png 1118w" sizes="(max-width: 477px) 100vw, 477px" /> 

**3: Setup GSuite**

Now dns is working we can connect GSuite to our domain. Start with navigating to the [GSuite homepage](https://gsuite.google.com/together/?user-benefits_activeEl=tab-connect) and press the Get Started button in the top right. You should be able to follow the wizard through as it is fairly self explanatory, if you are only going to be using it for yourself you should select &#8216;Just You&#8217; on the about page.

<img loading="lazy" class="alignnone size-medium wp-image-75" src="http://tomaustin.xyz/wp-content/uploads/2018/01/goog-254x300.png" alt="" width="254" height="300" srcset="https://tomaustin.xyz/wp-content/uploads/2018/01/goog-254x300.png 254w, https://tomaustin.xyz/wp-content/uploads/2018/01/goog.png 507w" sizes="(max-width: 254px) 100vw, 254px" /> 

When you get to this page select &#8216;Yes, I have one I can use&#8217;

<img loading="lazy" class="alignnone size-medium wp-image-76" src="http://tomaustin.xyz/wp-content/uploads/2018/01/customdomain-300x138.png" alt="" width="300" height="138" srcset="https://tomaustin.xyz/wp-content/uploads/2018/01/customdomain-300x138.png 300w, https://tomaustin.xyz/wp-content/uploads/2018/01/customdomain.png 533w" sizes="(max-width: 300px) 100vw, 300px" /> 

<img loading="lazy" class="alignnone size-medium wp-image-77" src="http://tomaustin.xyz/wp-content/uploads/2018/01/test-300x198.png" alt="" width="300" height="198" srcset="https://tomaustin.xyz/wp-content/uploads/2018/01/test-300x198.png 300w, https://tomaustin.xyz/wp-content/uploads/2018/01/test.png 569w" sizes="(max-width: 300px) 100vw, 300px" /> 

Press next and fill in the details on the next few prompts and then continue to setup. When prompted if you want to add people just click &#8216;I have added all user email addresses currrently using &#8230;&#8217;

When you get to the verify domain section go back to dynu, click manage dns and add an TXT record (you can do this in the DNS records settings panel) and also setup the MX records. Once done your MX Records page should look something like this  
<img loading="lazy" class="alignnone  wp-image-79" src="http://tomaustin.xyz/wp-content/uploads/2018/01/sns-300x61.png" alt="" width="634" height="129" srcset="https://tomaustin.xyz/wp-content/uploads/2018/01/sns-300x61.png 300w, https://tomaustin.xyz/wp-content/uploads/2018/01/sns-768x157.png 768w, https://tomaustin.xyz/wp-content/uploads/2018/01/sns-720x147.png 720w, https://tomaustin.xyz/wp-content/uploads/2018/01/sns.png 965w" sizes="(max-width: 634px) 100vw, 634px" /> 

Once that is done google should verify correctly and you should now be able to use GSuite for your email and utilise the unlimited drive storage which comes with GSuite.

&nbsp;