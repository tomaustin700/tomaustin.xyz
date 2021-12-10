---
id: 91
title: 'Workrbot &#8211; Microsoft TFS Auto Moderator'
date: 2018-03-20T11:46:19+00:00
author: tom
layout: post
guid: http://3.8.252.104/?p=91
permalink: /2018/03/20/workrbot-microsoft-tfs-auto-moderator/
categories:
  - api
  - asp.net
  - 'c#'
  - tfs
---
I use [Microsoft TFS](https://www.visualstudio.com/tfs/) on a daily basis and have found that no matter how many documents and guides you write regarding processes people will log bugs/user stories incorrectly. To try and help with this I built a bot which can be given a ruleset and notify users when substandard work items are added. More information can be found on my github page [here](https://github.com/tomaustin700/Workrbot). Below you can find the installation instructions.

# Installation

Workrbot can be installed within IIS and requires an Active Directory user account to run. The below steps show a basic installation.

**Create new IIS Application Pool**

Open IIS and create a new application pool for Workrbot, once created open &#8216;Advanced Settings&#8217; for the pool and turn on &#8216;Load User Profile&#8217;

![IIS Application pool](https://i.imgur.com/gr2bs48.png) 

**Create new IIS Application**

This is where Workrbot will be accessed and is where you will need to deploy to. The settings are fairly straight forward for this (remember to select the application pool you created)  
![add iis application](https://i.imgur.com/3lUzQth.png) 

**Create Active Directory User Account**

Workrbot requires an AD user account to run, no specific permissions are required so just add a basic account.

![Add AD User](https://i.imgur.com/sOwNnjA.png) 

**Grant User Permissions Within TFS**

In order to update work items within TFS Workrbot needs some permissions, I have just added it to the Team I want to manage.

![Add User to TFS](https://i.imgur.com/PIpTKAl.png) 

**Deploy to IIS**

Deploy Workrbot to your IIS application in your preferred way. I normally do it from visual studio and publish it to the directory of the application however you can do it using web deploy or any other means of getting it to your application.

**Configure Workrbot Settings**

You should now have a site you can access (assuming you deployed to IIS correctly). Simply navigate to your workrbot instance using your preferred browser and click settings.  
![workrbot settings](https://i.imgur.com/JSj8GDu.png) 

**Configure TFS Web Hooks**

In order for TFS to communicate with Workrbot you need to setup web hooks within TFS (This can be done by hovering over the TFS configureation cog and selecting service hooks). You will then need to add the following web hooks.  
&#8211; Work Item Created (Point to http://_youriisinstance_/workrbot/api/TFS/WorkItemCreated/)  
&#8211; Work Item Deleted (Point to http://_youriisinstance_/workrbot/api/TFS/WorkItemDeleted/)  
&#8211; Work Item Restored (Point to http://_youriisinstance_/workrbot/api/TFS/WorkItemRestored/)  
&#8211; Work Item Updated (Point to http://_youriisinstance_/workrbot/api/TFS/WorkItemUpdated/)  
&#8211; Work Item Commented On (Point to http://_youriisinstance_/workrbot/api/TFS/CommentPosted/)

When adding the web hook you will need to set Resource Version to latest. Don&#8217;t worry if you test the web hook and it fails (some aren&#8217;t coded to accept the fake request that is sent when testing). Once all setup you should have something that looks like this  
![hooks](https://i.imgur.com/sNMoW7h.png) 

**Configure Workrbot**

You are almost done! Now just access your Workrbot instance and setup rules to do whatever you want  
![events](https://i.imgur.com/bPbMgDd.png)