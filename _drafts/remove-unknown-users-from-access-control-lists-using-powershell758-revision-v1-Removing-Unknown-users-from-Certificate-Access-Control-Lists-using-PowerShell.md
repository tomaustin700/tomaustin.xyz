---
id: 769
title: Removing Unknown users from Certificate Access Control Lists using PowerShell
date: 2021-03-21T17:17:19+00:00
author: tom
layout: revision
guid: https://tomaustin.xyz/?p=769
permalink: /?p=769
---
I recently had a scenario where I had a certificate with hundreds of unknown users added to it&#8217;s access control list which then needed removing, how had that happened and how did you resolve it? Good question! Let&#8217;s first quickly discuss how this had happened so you can avoid falling into the same issue. If you run IIS and host anything slightly complex you may need to grant an app pool permission to a certificate on the IIS server (a good example is if your hosting Identity Server), if you then remove that App Pool the permissions on the folder are left how they are and instead of your App Pool name showing within the ACL of the certificate you&#8217;ll just see a GUID (a SID). Eventually you&#8217;ll want to clean this up (or ideally fix the issue leaving the permissions behind when the App Pool&#8217;s are removed) and depending on how many entries there are in the ACL it could take some time. This is where PowerShell can save you a lot of time!