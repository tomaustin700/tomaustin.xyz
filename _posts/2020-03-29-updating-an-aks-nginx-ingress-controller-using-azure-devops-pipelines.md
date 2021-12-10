---
id: 573
title: Updating an AKS Nginx ingress controller using Azure DevOps pipelines.
date: 2020-03-29T15:21:04+01:00
author: tom
layout: post
guid: http://3.10.198.211/?p=573
permalink: /2020/03/29/updating-an-aks-nginx-ingress-controller-using-azure-devops-pipelines/
image: /wp-content/uploads/2020/03/ingresscicd-1.png
categories:
  - aks ingress tutorial
  - Azure DevOps
  - azure kubernetes service
  - CI CD
  - kubernetes
tags:
  - aks ingress tutorial
  - azure devops
  - azure kubernetes service
  - ci cd
  - ingress
  - nginx
---
This is the second article in a series exploring how to setup an [Azure Kubernetes Service](https://azure.microsoft.com/en-gb/services/kubernetes-service/) instance, implement a Nginx ingress controller and how to deploy apps to AKS using [Helm](https://helm.sh/). All the articles in the series can be found by [clicking here](http://tomaustin.xyz/category/aks-ingress-tutorial/). In this article we will use [Azure DevOps](https://azure.microsoft.com/en-gb/services/devops/) to build a CI/CD pipeline to automatically update the configuration of an AKS Nginx ingress controller when changes are made to the config file.

I&#8217;m going to assume you already have a running AKS instance with a correctly configured ingress controller, if you don&#8217;t then use the [first article to guide you through that process](http://tomaustin.xyz/2020/03/27/deploying-an-azure-kubernetes-service-aks-instance-with-an-nginx-ingress-controller/). I&#8217;m going to be using Azure DevOps to run my pipelines as it&#8217;s free for public projects, works really well and it&#8217;s what I&#8217;m familiar with &#8211; any other CI/CD pipeline should be able to accomplish what we are going to setup though.

Let&#8217;s start by creating a new Azure DevOps project for our pipelines, if you already have a project ready then you can skip this step.<figure class="wp-block-image size-large">

<img loading="lazy" width="651" height="361" src="http://tomaustin.xyz/wp-content/uploads/2020/03/image-1.png" alt="" class="wp-image-574" srcset="https://tomaustin.xyz/wp-content/uploads/2020/03/image-1.png 651w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-1-300x166.png 300w" sizes="(max-width: 651px) 100vw, 651px" /> </figure> 

Once created the first thing we are going to do is setup a service connection to our AKS instance, this will allow us to communicate with it through our pipeline. Hit the cog in the bottom left to open the project settings page and click on &#8216;Service connections&#8217; under &#8216;Pipelines&#8217;<figure class="wp-block-image size-large">

<img loading="lazy" width="629" height="371" src="http://tomaustin.xyz/wp-content/uploads/2020/03/image-2.png" alt="" class="wp-image-575" srcset="https://tomaustin.xyz/wp-content/uploads/2020/03/image-2.png 629w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-2-300x177.png 300w" sizes="(max-width: 629px) 100vw, 629px" /> </figure> 

Press &#8216;Create service connection and select &#8216;Azure Resource Manager&#8217; for the connection type before hitting &#8216;Next&#8217;.

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="489" height="121" src="http://tomaustin.xyz/wp-content/uploads/2020/03/image-3.png" alt="" class="wp-image-576" srcset="https://tomaustin.xyz/wp-content/uploads/2020/03/image-3.png 489w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-3-300x74.png 300w" sizes="(max-width: 489px) 100vw, 489px" /></figure>
</div>

Leave the authentication method at the recommended value of &#8216;Service principal&#8217; and press &#8216;Next&#8217; again. On the next page set the &#8216;Scope level&#8217; to &#8216;Subscription&#8217; and select the subscription that your AKS instance resides within. Now select your AKS resource group &#8211; you may get prompted to authenticate then just sign in to your Azure account. Finally give the service connection a meaningful name and press &#8216;Save&#8217;.

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="518" height="487" src="http://tomaustin.xyz/wp-content/uploads/2020/03/image-4.png" alt="" class="wp-image-577" srcset="https://tomaustin.xyz/wp-content/uploads/2020/03/image-4.png 518w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-4-300x282.png 300w" sizes="(max-width: 518px) 100vw, 518px" /></figure>
</div>

After a few seconds your service connection should be created and we can then move onto setting up the pipeline.

I&#8217;m going to be using yaml to declare my pipeline as I can just point a pipeline at a yaml file and all the configuration is already done. Yaml pipelines seem to be the preferred way of setting up pipelines now so if you aren&#8217;t familiar with them I suggest taking some time to investigate them further. Let&#8217;s start by navigating to Pipelines and pressing the &#8216;Create Pipeline&#8217; button.

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="506" height="292" src="http://tomaustin.xyz/wp-content/uploads/2020/03/image-5.png" alt="" class="wp-image-578" srcset="https://tomaustin.xyz/wp-content/uploads/2020/03/image-5.png 506w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-5-300x173.png 300w" sizes="(max-width: 506px) 100vw, 506px" /></figure>
</div>

I have written a complete yaml file for the pipeline so feel free to fork my project on GitHub, you can find it [here](https://github.com/tomaustin700/AKSIngress). My repository only contains two files: ingress.yaml which is the configuration for our ingress controller and azure-pipeline.yaml which is the configuration for our pipeline. When prompted where your code is select your repository location, in my case this will be &#8216;GitHub (YAML)&#8217;.

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="481" height="120" src="http://tomaustin.xyz/wp-content/uploads/2020/03/image-6.png" alt="" class="wp-image-579" srcset="https://tomaustin.xyz/wp-content/uploads/2020/03/image-6.png 481w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-6-300x75.png 300w" sizes="(max-width: 481px) 100vw, 481px" /></figure>
</div>

Select your GitHub repository and select &#8216;Existing Azure Pipelines YAML file&#8217; when prompted.

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="595" height="123" src="http://tomaustin.xyz/wp-content/uploads/2020/03/image-7.png" alt="" class="wp-image-580" srcset="https://tomaustin.xyz/wp-content/uploads/2020/03/image-7.png 595w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-7-300x62.png 300w" sizes="(max-width: 595px) 100vw, 595px" /></figure>
</div>

Now point the pipeline to the azure-pipeline.yaml file and press &#8216;Continue&#8217;.

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="505" height="301" src="http://tomaustin.xyz/wp-content/uploads/2020/03/image-8.png" alt="" class="wp-image-581" srcset="https://tomaustin.xyz/wp-content/uploads/2020/03/image-8.png 505w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-8-300x179.png 300w" sizes="(max-width: 505px) 100vw, 505px" /></figure>
</div>

The last step is to review the pipeline and make any changes we may need to make. The file is already configured perfectly for me but you may need to change the &#8216;azureSubscriptionEndpoint&#8217; parameters to match the name of the service connection you made earlier, you may also need to update &#8216;azureResourceGroup&#8217; and &#8216;kubernetesCluster&#8217; to match your resources. The &#8216;Release&#8217; stage of the pipeline simply runs two kubectl commands: &#8216;Login&#8217; which will authenticate with the instance and &#8216;Apply&#8217; which will apply our new ingress configuration. Once you have finished making any changes hit &#8216;Run&#8217; in the top right corner and your pipeline will start to execute.

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="582" height="517" src="http://tomaustin.xyz/wp-content/uploads/2020/03/image-9.png" alt="" class="wp-image-582" srcset="https://tomaustin.xyz/wp-content/uploads/2020/03/image-9.png 582w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-9-300x266.png 300w" sizes="(max-width: 582px) 100vw, 582px" /></figure>
</div>

The pipeline is split into two stages: &#8216;Build&#8217; which will take our ingress.yaml file and publish it and &#8216;Release&#8217; which will pick up that published file and update our ingress controller with that configuration. After a few minutes your pipeline should have completed.

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="558" height="208" src="http://tomaustin.xyz/wp-content/uploads/2020/03/image-10.png" alt="" class="wp-image-583" srcset="https://tomaustin.xyz/wp-content/uploads/2020/03/image-10.png 558w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-10-300x112.png 300w" sizes="(max-width: 558px) 100vw, 558px" /></figure>
</div>

That&#8217;s pretty much it! Now whenever you make commit changes to ingress.yaml the pipeline will automatically update your ingress controller for you. 

The final article will explore building a .NET Core application, creating a Helm chart for deployment and building a CI/CD pipeline to deploy to our AKS instance.