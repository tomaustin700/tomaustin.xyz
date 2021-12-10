---
id: 511
title: Deploying to AKS using Azure DevOps and Helm 3
date: 2020-03-28T00:00:00+00:00
author: tom
layout: post
guid: http://3.10.198.211/?p=511
permalink: /?p=511
categories:
  - Uncategorised
---
Earlier in the year I wrote two articles describing how to deploy a website and an API to Azure Kubernetes Service using Azure DevOps, I was ready to write some more articles in the same series (such as deploying an Nginx ingress controller and lets encrypt) but I heard that the next release of Helm was getting rid of tiller (hooray!) so thought I better delay any more AKS tutorials until Helm 3 was released. Well Helm 3 was released a few weeks ago so here we go!

Helm 3 was in pre-release for quite some time and I was tempted to write a tutorial showing how to use it but I didn&#8217;t want to write something that was then obsolete when another version was released so I waited. When the release of Helm 3 was imminent I did try and test using the lastest version but ran into a few issues with it and Azure DevOps (I emailed [Jessica Deen](https://twitter.com/jldeen) about this and she gave me a workaround and then put out an [article outlining how to use the pre-release of Helm 3 with Azure DevOps](https://jessicadeen.com/using-helm-3-with-azure-devops/)) but before I had time to try everything out Helm 3 was officially released!