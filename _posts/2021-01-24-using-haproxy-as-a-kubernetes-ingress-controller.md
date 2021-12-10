---
id: 714
title: Using HAProxy as a Kubernetes Ingress Controller
date: 2021-01-24T16:38:51+00:00
author: tom
layout: post
guid: http://tomaustin.xyz/?p=714
permalink: /2021/01/24/using-haproxy-as-a-kubernetes-ingress-controller/
spay_email:
  - ""
image: /wp-content/uploads/2021/01/haproxy_logo-2.png
categories:
  - aks ingress tutorial
  - azure kubernetes service
  - kubernetes
tags:
  - azure kubernetes service
  - haproxy
  - ingress
  - kubernetes
---
Whenever I&#8217;ve needed a Kubernetes Ingress controller in the past I&#8217;ve always used Nginx, not because it&#8217;s better than everything else but just because I was familiar with it. Outside of Kubernetes I&#8217;ve always used [HAProxy](http://www.haproxy.org/) as my reverse-proxy of choice and been very happy with it, it&#8217;s always performed well and been easy to configure. Recently HAProxy released [v1.5 of their Ingress controller](https://www.haproxy.com/blog/announcing-haproxy-kubernetes-ingress-controller-1-5/) so I thought it was time to try HAProxy instead of Nginx.

In this article I&#8217;m going to show the steps to get the HAProxy Ingress running within Kubernetes and configure it to allow access. I&#8217;m going to be using [Azure Kubernetes Service](https://azure.microsoft.com/en-gb/services/kubernetes-service/) but these steps should work for any Kubernetes setup. 

I&#8217;ve already got Kubernetes setup (Azure makes this easy) however if you don&#8217;t you can quickly get a running instance by running the [following script](https://github.com/tomaustin700/AKS-Nginx-Deploy-Script/blob/master/AKSDeploy.ps1) (make sure to remove the installation of Nginx and it&#8217;s configuration as that&#8217;s not needed). We&#8217;re going to be using [Helm](https://helm.sh/) for HAProxy deployment as it makes it really quick and easy to get it running with minimal configuration required, I&#8217;m going to presume you have Helm already setup but if not you can follow the steps [here](https://helm.sh/docs/intro/install/). You will also need Kubectl installed and configured.

Let&#8217;s start by adding the HAProxy Helm repository by running the following Powershell:

<pre class="wp-block-code"><code>helm repo add haproxytech https://haproxytech.github.io/helm-charts</code></pre>

We can then update the Helm chats

<pre class="wp-block-code"><code>helm repo update</code></pre>

Now we can use Helm to install the HAProxy Ingress Controller. Make sure you update $publicIP with an actual public IP address.

<pre class="wp-block-code"><code>helm install kubernetes-ingress haproxytech/kubernetes-ingress --set controller.service.type=LoadBalancer --set controller.service.loadBalancerIP="$publicIP"</code></pre>

We can check the deployment was successful by running the following kubectl command:

<pre class="wp-block-code"><code>kubectl get deployments</code></pre>

and we should see the following:<figure class="wp-block-image size-large">

<img loading="lazy" width="681" height="63" src="https://tomaustin.xyz/wp-content/uploads/2021/01/image-5.png" alt="" class="wp-image-715" srcset="https://tomaustin.xyz/wp-content/uploads/2021/01/image-5.png 681w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-5-300x28.png 300w" sizes="(max-width: 681px) 100vw, 681px" /> </figure> 

We can also check the public IP address was set correctly by getting the services and checking the external IP.

<pre class="wp-block-code"><code>kubectl get services</code></pre><figure class="wp-block-image size-large">

<img loading="lazy" width="719" height="79" src="https://tomaustin.xyz/wp-content/uploads/2021/01/image-7.png" alt="" class="wp-image-717" srcset="https://tomaustin.xyz/wp-content/uploads/2021/01/image-7.png 719w, https://tomaustin.xyz/wp-content/uploads/2021/01/image-7-300x33.png 300w" sizes="(max-width: 719px) 100vw, 719px" /> </figure> 

That all looks good to me!

Now that the HAProxy Ingress Controller is running we can create another deployment before configuring HAProxy to route traffic to it. In this example I&#8217;m just going to deploy my basic Helloworld image that I normally use for tutorials

<pre class="wp-block-code"><code>kubectl create deployment helloworld --image tomaustin/helloworldnetcore</code></pre>

Once created expose the deployment:

<pre class="wp-block-code"><code>kubectl expose deployment helloworld --name helloworld --port 80</code></pre>

We can now just configure the Ingress Controller using Kubectl and that should be pretty much it. Save the following to a file and apply it using Kubectl, make sure to update &#8216;yourdnsnamegoeshere&#8217; with your dns address.

<pre class="wp-block-code"><code>apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress
  namespace: default
  annotations:
    haproxy.org/check: "true"
spec:
  rules:
  - host: yourdnsnamegoeshere
    http:
      paths:
      - path: /
        backend:
          serviceName: helloworld
          servicePort: 80</code></pre>

<pre class="wp-block-code"><code>kubectl apply -f ingress.yml</code></pre>

That should be it! If all went to plan you should be able to navigate to your dns address and see your service<figure class="wp-block-image size-large">

<img loading="lazy" width="845" height="283" src="http://tomaustin.xyz/wp-content/uploads/2020/03/image.png" alt="" class="wp-image-566" srcset="https://tomaustin.xyz/wp-content/uploads/2020/03/image.png 845w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-300x100.png 300w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-768x257.png 768w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-720x241.png 720w" sizes="(max-width: 845px) 100vw, 845px" /> </figure> 

The HAProxy Ingress Controller can do a fair bit so make sure to check out the documentation [here](https://www.haproxy.com/documentation/kubernetes/latest/configuration/ingress/). If you encountered any issues then don&#8217;t hesitate to get in touch by either leaving a comment or contacting me on [Twitter](https://twitter.com/tomaustin700). Thanks!