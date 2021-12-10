---
id: 358
title: Deploying an HTTPS Ingress Controller to Azure Kubernetes Service (AKS) using Azure DevOps
date: 2019-05-29T12:51:55+01:00
author: tom
layout: post
guid: http://3.10.198.211/?p=358
permalink: /?p=358
categories:
  - Uncategorised
---
This is the third article in a series exploring deploying services to Azure Kubernetes Service using Azure DevOps. This article will cover deploying an HTTPS ingress controller to your AKS instance to act as a gateway between your services and the outside world.

When we have deployed services to AKS in the past we have used &#8216;HTTP Application Routing&#8217; provided by Azure, this is good for getting things up and running quickly but we don&#8217;t want to use this in a production environment for a few reasons; firstly it does not support HTTPS (I&#8217;m sure I don&#8217;t need to go into the downsides of this) and secondly we get no control over how the routing is done. I&#8217;d recommend starting from scratch with your AKS instance