---
id: 694
title: Deploying Selenium Grid 4 to Azure Kubernetes Service (AKS) using Azure DevOps
date: 2021-01-13T18:00:00+00:00
author: tom
layout: post
guid: http://tomaustin.xyz/?p=694
permalink: /2021/01/13/deploying-selenium-grid-4-to-azure-kubernetes-service-aks-using-azure-devops/
image: /wp-content/uploads/2021/01/image-4.png
categories:
  - azure
  - Azure DevOps
  - azure kubernetes service
  - CI CD
  - devops
  - kubernetes
  - selenium
  - unit testing
tags:
  - azure devops
  - azure kubernetes service
  - kubernetes
  - selenium
  - selenium grid
---
One of the most popular articles I&#8217;ve written on this blog was [this tutorial](https://tomaustin.xyz/2020/02/02/deploying-selenium-grid-to-azure-kubernetes-service-aks-using-azure-devops/) showing how to deploy Selenium Grid 3 to Azure Kubernetes Service, Selenium Grid 4 is now in beta and has been rewritten from scratch so there are some big changes which you need to be aware of when deploying to Kubernetes. This article will explain some of the changes between Grid 3 and 4 and guide you through deploying Grid 4 to your AKS instance using Azure Pipelines.

Before we get started I am going to assume you have a running AKS instance with a functioning ingress controller, if you don&#8217;t then you can follow the following tutorials to get up and running &#8211; [Deploying an Azure Kubernetes Service (AKS) instance with an Nginx ingress controller](https://tomaustin.xyz/2020/03/27/deploying-an-azure-kubernetes-service-aks-instance-with-an-nginx-ingress-controller/) and [Updating an AKS Nginx ingress controller using Azure DevOps pipelines](https://tomaustin.xyz/2020/03/29/updating-an-aks-nginx-ingress-controller-using-azure-devops-pipelines/). It&#8217;s also beneficial to have kubectl [installed locally and configured](https://kubernetes.io/docs/tasks/tools/install-kubectl/). 

Let&#8217;s firstly look at some of the differences between Grid 3 and 4. Selenium Grid 3 had two types of components: Hub and Node. The hub would forward test jobs onto the nodes and the nodes would execute the tests, you would have one hub and then many nodes.<figure class="wp-block-image">

![Grid](https://www.selenium.dev/documentation/images/grid.png) <figcaption>Image from https://www.selenium.dev/documentation/en/grid/grid\_3/components\_of\_a\_grid/</figcaption></figure> 

With Selenium Grid 4 things are a bit different, the hub has now gone and in it&#8217;s place we have the following components:

  * Router
  * Distributor
  * Session Map
  * Session Queue
  * Event Bus<figure class="wp-block-image">

![Grid](https://www.selenium.dev/documentation/images/grid_4.png) <figcaption>Image from https://www.selenium.dev/documentation/en/grid/grid\_4/components\_of\_a\_grid/</figcaption></figure> 

I&#8217;m not going to go into detail about how Grid 4 works with the new components but if you are interested it&#8217;s all documented [here](https://www.selenium.dev/documentation/en/grid/grid_4/components_of_a_grid/). Due to the additional components this makes it a little trickier to deploy but it&#8217;s not too bad once you get your head around it. Let&#8217;s get onto deployment!

I&#8217;m going to be using [Helm](https://helm.sh/) to deploy the Grid 4 components using Azure DevOps. I&#8217;m only using Azure DevOps as it&#8217;s what I&#8217;m familiar with however the concept is the same if you are using GitHub Actions or any other CI/CD system. 

To deploy the components using Helm we need Helm charts for each component, if you navigate to the [following GitHub repo](https://github.com/tomaustin700/AKSSeleniumGrid4) I have already constructed all the charts we are going to need. That repo also contains the pipeline for deploying so it should just be a case of configuring Azure DevOps and pointing it at our files and pipeline. I&#8217;d recommend forking that GitHub repo if you want to follow along easily.

Let&#8217;s head over to Azure DevOps and make a new project, once your project has initialised press the Project Settings cog in the bottom left and navigate to &#8216;Service connections&#8217;, then press &#8216;New service connection&#8217; in the top right. 

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="https://tomaustin.xyz/wp-content/uploads/2021/01/image-1024x680.png" alt="" class="wp-image-697" width="416" height="276" srcset="https://tomaustin.xyz/wp-content/uploads/2021/01/image-1024x680.png 1024w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-300x199.png 300w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-768x510.png 768w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-720x478.png 720w, https://tomaustin.xyz/wp-content/uploads/2021/01/image.png 1333w" sizes="(max-width: 416px) 100vw, 416px" /></figure>
</div>

Select &#8216;Azure Resource Manager&#8217; and press &#8216;Next&#8217;. Leave authentication method at &#8216;Service principal&#8217; and press &#8216;Next&#8217; again.

We now need to connect to our Azure subscription so leave &#8216;Scope level&#8217; at &#8216;Subscription and select your Azure Subscription from the drop down. Leave &#8216;Resource Group&#8217; blank and set the service connection name. Finally press &#8216;Save&#8217;

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="https://tomaustin.xyz/wp-content/uploads/2021/01/image-1-806x1024.png" alt="" class="wp-image-698" width="431" height="547" srcset="https://tomaustin.xyz/wp-content/uploads/2021/01/image-1-806x1024.png 806w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-1-236x300.png 236w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-1-768x975.png 768w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-1-1209x1536.png 1209w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-1-720x914.png 720w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-1.png 1270w" sizes="(max-width: 431px) 100vw, 431px" /></figure>
</div>

Now we just need to add the pipeline so head over to Azure Pipelines by pressing the Pipeline button on the left. Once you&#8217;re on Pipelines press &#8216;New pipeline&#8217; in the top right. Select the location of your code (in my case GitHub) and select the repository. When prompted to configure the pipeline select &#8216;Existing Azure Pipelines YAML file&#8217; and select your pipeline file before pressing &#8216;Continue&#8217;.

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="https://tomaustin.xyz/wp-content/uploads/2021/01/image-2-1024x624.png" alt="" class="wp-image-699" width="434" height="264" srcset="https://tomaustin.xyz/wp-content/uploads/2021/01/image-2-1024x624.png 1024w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-2-300x183.png 300w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-2-768x468.png 768w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-2-720x439.png 720w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-2.png 1273w" sizes="(max-width: 434px) 100vw, 434px" /></figure>
</div>

You should now see your YAML pipeline file. You may need to make a few changes to this depending on your Azure service connection name you set earlier, Azure Resource Group name and AKS instance name. Update the following variables on each task to match yours:

<pre class="wp-block-code"><code>        azureSubscriptionEndpoint: 'Azure'
        azureResourceGroup: aksselgrig4test
        kubernetesCluster: aksselgrid4</code></pre>

Make sure to update these variables on each task, you could potentially declare them in the &#8216;variables&#8217; section to make things slightly easier but I haven&#8217;t. Also make sure to update the namespace variable to match the namespace you want to deploy Selenium Grid 4 to within your AKS instance. Once you&#8217;ve finished configuring your pipeline you should be able to run it and everything should deploy.

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="https://tomaustin.xyz/wp-content/uploads/2021/01/image-3-1024x812.png" alt="" class="wp-image-700" width="422" height="334" srcset="https://tomaustin.xyz/wp-content/uploads/2021/01/image-3-1024x812.png 1024w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-3-300x238.png 300w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-3-768x609.png 768w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-3-720x571.png 720w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-3.png 1230w" sizes="(max-width: 422px) 100vw, 422px" /></figure>
</div>

If you have kubectl installed locally and configured you can run the following command to list all the components that we just deployed.

<pre class="wp-block-code"><code>kubectl get all -l component=selenium-grid-4 -n ingress-basic</code></pre>

With any luck you should be able to see everything running correctly.<figure class="wp-block-image size-large">

<img loading="lazy" width="749" height="199" src="https://tomaustin.xyz/wp-content/uploads/2021/01/image-4.png" alt="" class="wp-image-701" srcset="https://tomaustin.xyz/wp-content/uploads/2021/01/image-4.png 749w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-4-300x80.png 300w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-4-720x191.png 720w" sizes="(max-width: 749px) 100vw, 749px" /> </figure> 

Now the only thing left to do is configure your ingress controller to allow access to selenium-router. I&#8217;m using an nginx ingress controller so I applied the following config using kubectl, you could also do with with an Azure DevOps pipeline as seen [here](https://tomaustin.xyz/2020/03/29/updating-an-aks-nginx-ingress-controller-using-azure-devops-pipelines/) &#8211; I&#8217;d recommend you do it this way if running in prod.

<pre class="wp-block-code"><code>apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress
  namespace: ingress-basic
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: youraksinstancehostnamegoeshere
    http:
      paths:
      - path: /grid(/|$)(.*)
        backend:
          serviceName: selenium-router
          servicePort: 4444
      - path: /
        backend:
          serviceName: helloworld
          servicePort: 80</code></pre>

There are a few things to note in the config. Firstly make sure to update the host to match the hostname of your AKS instance. Also I already had a service running in my AKS instance that had a rule so I needed to add another for Selenium Grid, I used an [ingress rule rewrite annotation](https://kubernetes.github.io/ingress-nginx/examples/rewrite/) in the path which you may also need to do if you have other services on your AKS instance. 

Once you&#8217;ve applied that configuration you can use [Postman](https://www.postman.com/product/api-client/) to check everything is running correctly. Simply do an HTTP GET request to the hostname of your AKS instance followed by /status. You should get a response with &#8216;ready: true&#8217; and info about your nodes. You should be able to start running tests on Grid. Something to note is that the url for running tests has changed between Grid 3 and Grid 4: the Grid 3 used to be hostname/wd/hub whereas now you don&#8217;t need the /wd/hub portion.

That concludes this tutorial and you should now have Grid 4 deployed to AKS using an Azure DevOps pipeline. If you have any issues or queries please leave a comment or contact me on [Twitter](https://twitter.com/tomaustin700). Thanks