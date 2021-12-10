---
id: 800
title: Using Ansible to manage Windows hosts
date: 2021-04-02T18:08:03+01:00
author: tom
layout: revision
guid: https://tomaustin.xyz/?p=800
permalink: /?p=800
---
In this article I will guide you through what&#8217;s needed to configure [Ansible](https://www.ansible.com/products/engine) so it can be used to manage Windows hosts. Using Ansible to manage Windows hosts gives you great benefits such as installing updates, installing software, joining domains and a whole lot more all using infrastructure as code. We&#8217;re going to be using [Kerberos](https://docs.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview) to authenticate with our Windows hosts so you&#8217;re going to need a server running [Active Directory Domain Services](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) and then a Windows host joined to that domain, Ansible is going to be running on an Ubuntu host so that will also need joining to the domain (if you need assistance with that [check out this article](https://tomaustin.xyz/2021/03/31/join-ubuntu-to-an-active-directory-domain)).

Let&#8217;s start by configuring our Windows host to allow Ansible to connect to it, simply run the following Powershell script on your Windows host.

<pre class="wp-block-code"><code>$url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"

(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)

powershell.exe -ExecutionPolicy ByPass -File $file</code></pre>

Now that&#8217;s done let&#8217;s SSH into our Ubuntu machine, this is going to be our Ansible controller (or control-node as Ansible like to call it) and install everything we need.

<pre class="wp-block-code"><code>sudo apt install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt update
sudo apt install ansible
sudo apt-get install python-dev libkrb5-dev krb5-user
sudo apt-get install python-pip
pip install pywinrm
pip install pywinrm&#91;kerberos]</code></pre>

Now that Ansible is installed we can get to configuring everything. Let&#8217;s add our Windows hosts to the /etc/ansible/hosts file

<pre class="wp-block-code"><code>sudo nano /etc/ansible/hosts</code></pre>

Let&#8217;s declare a group called windows and add our hosts to it, make sure to use the full FQDN of the Windows machine you want to manage (in this case s2019 is the name of my Windows host and lab.local is the domain)

<pre class="wp-block-code"><code>&#91;win]
s2019.lab.local</code></pre>

Below this we need to add some configuration to let Ansible know how to communicate with our Windows hosts. I&#8217;ve created an domain Anisble user that I&#8217;m going to use but you can use any domain account. Ideally we wouldn&#8217;t set our credentials here but this is fine just for an example, in production I&#8217;d pass these credentials in as part of a CI/CD pipeline.

<pre class="wp-block-code"><code>&#91;win:vars]
ansible_user=ansible@LAB.LOCAL
ansible_password=password
ansible_connection=winrm
ansible_winrm_transport=kerberos
ansible_winrm_server_cert_validation=ignore</code></pre>

Save and exit (Ctrl + O, Enter, Ctrl + X)

Now that the configuration is done let&#8217;s write a basic playbook. A playbook is a collection of tasks that Ansible will execute against the hosts.

<pre class="wp-block-code"><code>sudo nano updates.yml</code></pre>

<pre class="wp-block-code"><code>- name: Windows Updates
  hosts: win

  tasks:
    - name: Install updates
      win_updates:
        category_names:
          - SecurityUpdates
          - CriticalUpdates
        reboot: yes</code></pre>

This playbook will install security and critical Windows updates and reboot if necessary. Let&#8217;s Save and Exit again.

Now we can run the playbook.

<pre class="wp-block-code"><code>ansible-playbook updates.yml</code></pre>

If all has gone to plan Ansible should connect to your Windows hosts and install the updates we told it to before rebooting if required. 

This is a fairly basic example and like I said earlier; we ideally want to be doing this from a CI/CD pipeline &#8211; look for an article about that in the future. If you have any questions or comments then please leave a comment or contact me on [Twitter](https://twitter.com/tomaustin700). Thanks.