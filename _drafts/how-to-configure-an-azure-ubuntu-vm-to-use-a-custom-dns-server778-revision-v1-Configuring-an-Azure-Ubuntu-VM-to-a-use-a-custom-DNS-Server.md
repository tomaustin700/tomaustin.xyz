---
id: 781
title: Configuring an Azure Ubuntu VM to a use a custom DNS Server
date: 2021-03-28T15:17:29+01:00
author: tom
layout: revision
guid: https://tomaustin.xyz/?p=781
permalink: /?p=781
---
I have recently been playing around with [Kerberos](https://en.wikipedia.org/wiki/Kerberos_(protocol)) and was attempting to join an Azure Ubuntu VM to domain I had set up within Azure, if you&#8217;ve ever had anything to do with [Active Directory](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) you&#8217;ll know that you need to set the DNS servers of the machines you want to join to the domain to the address of the domain controller and this was no different. I thought this would be a relatively simple task however despite my best efforts I couldn&#8217;t get the Ubuntu VM to use the DNS server I configured. Here&#8217;s how to get a Azure Ubuntu VM use custom DNS Server.

When you create a VM in Azure it will create a VNet for you (or let you connect to an already existing VNet) the VNet allows all the VM&#8217;s within the VNet to communicate and also handles DNS for you, you can read more about this [here](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances). Normally this is great and makes things nice and easy however in this case it was getting in the way and even though I could ping my domain controller using it&#8217;s name I couldn&#8217;t actually ping the DNS name of the domain. After a lot of trial and error and searching I finally found a solution which worked.

Firstly edit /etc/systemd/resolved.conf and set the DNS Server and the Domain

<pre class="wp-block-code"><code>sudo nano /etc/systemd/resolved.conf</code></pre>

Uncomment (remove the #) DNS and set it to your chosen DNS server and uncomment Domain and set this to your Domain address.

Now switch /etc/resolv.conf to the version provided by systemd

<pre class="wp-block-code"><code>sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf</code></pre>

Now we just have to restart systemd-resolved

<pre class="wp-block-code"><code>sudo systemctl stop systemd-resolved
sudo systemctl start systemd-resolved</code></pre>

That should be it. Make sure everything is still working by pinging something (I normally start with google) and then try and ping something within the VNet. If you have any questions or comments then please leave a comment below or contact me on [Twitter](https://twitter.com/tomaustin700).