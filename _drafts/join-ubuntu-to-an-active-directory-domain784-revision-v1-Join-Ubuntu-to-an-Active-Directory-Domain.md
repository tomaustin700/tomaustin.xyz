---
id: 787
title: Join Ubuntu to an Active Directory Domain
date: 2021-03-31T09:56:59+01:00
author: tom
layout: revision
guid: https://tomaustin.xyz/?p=787
permalink: /?p=787
---
This tutorial will show you how to join Ubuntu to an Active Directory Domain quickly and easily, this will allow domain users to logon using their domain account and allows the use of Kerberos authentication with tools such as Ansible. Before I start I&#8217;m going to assume you already have a domain controller configured and setup with Active Directory Domain Services.

Start by using your favourite SSH client to connect to Ubuntu, once connected the first thing we need to do is configure the hosts file:

<pre class="wp-block-code"><code>sudo nano /etc/hosts</code></pre>

Update the hosts file so it looks something like this (I&#8217;m using Ubuntu as the name of my Ubuntu instance and lab.local as my domain address)

<pre class="wp-block-code"><code>127.0.0.1 ubuntu.lab.local ubuntu</code></pre>

Save and exit (ctrl+O, enter, ctrl+x)

Now we need to install the required packages to allow us to use Kerberos authentication. Run the following commands to get everything installed

<pre class="wp-block-code"><code>sudo apt-get update
sudo apt-get install krb5-user samba sssd sssd-tools libnss-sss libpam-sss ntp ntpdate realmd adcli</code></pre>

Once installed we need to configure Kerberos so let&#8217;s edit the krb5 configuration file

<pre class="wp-block-code"><code>sudo nano /etc/krb5.conf</code></pre>

The top of your file should already show the default realm as your domain address (in my case it&#8217;s LAB.LOCAL) so you shouldn&#8217;t need to edit that. In the [realms] section add a new entry for your domain and point it towards your domain controller.

<pre class="wp-block-code"><code>&#91;realms]
LAB.LOCAL = {
        kdc = dc.lab.local
        admin_server = dc.lab.local
}</code></pre>

Now go to the bottom of the file and add a domain realm to the [domain_realm] section

<pre class="wp-block-code"><code>.lab.local = LAB.LOCAL</code></pre>

Save and exit

Now we need to update NTP so the time and date is syncronised with the domain, start by editing the /etc/ntp.conf file

<pre class="wp-block-code"><code>sudo nano /etc/ntp.conf</code></pre>

Add an entry at the bottom of the file for your domains address

<pre class="wp-block-code"><code>server lab.local</code></pre>

We&#8217;re now going to syncronise with the domain, we can do that with the following commands

<pre class="wp-block-code"><code>sudo systemctl stop ntp
sudo ntpdate lab.local
sudo systemctl start ntp</code></pre>

We can now start the process of joining Ubuntu to the domain, we start by discovering the realm &#8211; **your domain address needs to be all uppercase**.

<pre class="wp-block-code"><code>sudo realm discover LAB.LOCAL</code></pre>

Now that the realm has been discovered we can initilise Kerberos. Make sure to use an account that is part of the domain and the **address must be uppercase again**.

<pre class="wp-block-code"><code>kinit -V admin@LAB.LOCAL</code></pre>

We&#8217;re almost there now, let&#8217;s now join the realm &#8211; **upper case domain address again here**.

<pre class="wp-block-code"><code>sudo realm join --verbose LAB.LOCAL -U 'admin@LAB.LOCAL' --install=/</code></pre>

If all has gone to plan you should be greeted with a success message

<pre class="wp-block-code"><code>Successfully enrolled machine in realm</code></pre>

There are a few more steps we need to do to finish the job, let&#8217;s start by editing sssd.conf

<pre class="wp-block-code"><code>sudo nano /etc/sssd/sssd.conf</code></pre>

Put a # at the start of the use\_fully\_qualified_names line

<pre class="wp-block-code"><code># use_fully_qualified_names = True</code></pre>

Save and exit and then restart the sssd service

<pre class="wp-block-code"><code>sudo systemctl restart sssd</code></pre>

We can now enable password authentication so your domain users can logon

<pre class="wp-block-code"><code>sudo nano /etc/ssh/sshd_config</code></pre>

Set PasswordAuthentication to yes

<pre class="wp-block-code"><code>PasswordAuthentication yes</code></pre>

Save and exit and then restart the SSH service

<pre class="wp-block-code"><code>sudo systemctl restart ssh</code></pre>

The last steps are enabling home directory creation and adding DC Administration to the sudo privileges group.

<pre class="wp-block-code"><code>sudo nano /etc/pam.d/common-session</code></pre>

Add the following line below session optional pam_sss.so

<pre class="wp-block-code"><code>session required pam_mkhomedir.so skel=/etc/skel/ umask=0077</code></pre>

Save and exit.

Now open the sudoers file

<pre class="wp-block-code"><code>sudo visudo</code></pre>

And add the following to the end

<pre class="wp-block-code"><code>%AAD\ DC\ Administrators ALL=(ALL) NOPASSWD:ALL</code></pre>

Save and exit as usual.

That&#8217;s it! You should now be able to login using your domain accounts. If you have any issues or questions feel free to leave a comment or get in touch with me on [Twitter](https://twitter.com/tomaustin700).