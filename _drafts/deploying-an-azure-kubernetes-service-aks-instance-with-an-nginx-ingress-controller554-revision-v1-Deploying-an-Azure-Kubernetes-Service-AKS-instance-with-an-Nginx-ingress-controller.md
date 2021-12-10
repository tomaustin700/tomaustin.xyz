---
id: 571
title: Deploying an Azure Kubernetes Service (AKS) instance with an Nginx ingress controller
date: 2020-03-27T16:40:02+00:00
author: tom
layout: revision
guid: http://3.10.198.211/2020/03/27/554-revision-v1/
permalink: /?p=571
---
This is the first article in a series which will guide you through deploying an [Azure Kubernetes Service (AKS)](https://azure.microsoft.com/en-gb/services/kubernetes-service/) instance on Azure with an Nginx ingress controller, configuring a CI/CD pipeline to configure that ingress controller and then deploying to the AKS instance using Azure [DevOps](https://azure.microsoft.com/en-gb/services/devops/). We are going to use the command line to deploy everything which may be daunting if you&#8217;re not familiar with it but it makes things fairly easy and we don&#8217;t have to worry about navigating the Azure UI.

Let&#8217;s start by installing the latest version of the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest). Once done open the command line tool of your choice (I&#8217;ve been really enjoying using the new [Windows Terminal](https://github.com/microsoft/terminal)) and run the login command.

<pre class="wp-block-code"><code>az login</code></pre>

Your default browser will hopefully open and ask you to login to your Azure account. Next let&#8217;s install [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) which is the Kubernetes command-line tool.

<pre class="wp-block-code"><code>az aks install-cli</code></pre>

You&#8217;re also going to need to have [Helm](https://helm.sh/) installed. Helm is a package manager for kubernetes and will allow us to deploy applications to our instance, we will be using it to deploy [Nginx](https://www.nginx.com/). As of writing the latest version of Helm is [v3.1.2](https://github.com/helm/helm/releases/tag/v3.1.2) however get the latest version you can. To make things easier you can also add it to your Windows path, I&#8217;m not going to bother with this but if you want to there is a tutorial [here](https://medium.com/@JockDaRock/take-the-helm-with-kubernetes-on-windows-c2cd4373104b).

That should be everything we need to install locally so let&#8217;s get to deploying stuff to Azure. We will start with creating a resource group for our AKS instance.

<pre class="wp-block-code"><code>az group create --name AKS --location westeurope</code></pre>

You can call it what you like, I&#8217;ve named mine AKS just so it makes this tutorial fairly easy to follow. Also you can set the location to one of your choice also. To list all the Azure locations you can run the list-locations command.

<pre class="wp-block-code"><code>az account list-locations</code></pre>

Now that we have a resource group ready and waiting let&#8217;s deploy an AKS instance. I&#8217;m just going to be deploying a two node instance but you can tweak these parameters to your choosing.

<pre class="wp-block-code"><code>az aks create --resource-group AKS --name AKS --node-count 2 --generate-ssh-keys </code></pre>

If you get an error response mentioning ServicePrincipalProfile then wait a few minutes and re-run the command. This is a known issue and has been documented [here](https://github.com/Azure/azure-cli/issues/9585).

Once deployed we need to get the credentials for our instance so we can start to communicate with it using kubectl.

<pre class="wp-block-code"><code>az aks get-credentials --resource-group AKS --name AKS</code></pre>

We can now check everything is running correctly by running the get nodes command, this will return some data about our AKS nodes.

<pre class="wp-block-code"><code>kubectl get nodes</code></pre>

Now that our instance is deployed we can get to configuring it. We are going to start with creating a namespace. You can think of a namespace as a way to group pods, services and deployments together.

<pre class="wp-block-code"><code>kubectl create namespace ingress-basic</code></pre>

Let&#8217;s now deploy a container image to that namespace. I&#8217;ve put together a really basic .net core container which will simply show &#8216;hello world&#8217; when accessed.

<pre class="wp-block-code"><code>kubectl create deployment helloworld --image tomaustin/helloworldnetcore --namespace ingress-basic</code></pre>

This command will only deploy one pod but that should be fine for this tutorial, if you would like to learn more about deployments then the [official kubernetes documentation](https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/) is fairly in-depth. Once this command completes a pod will be running our image but there will be no way to access it. In order to be able to access it in the future we need to create a service for our deployment. Running the following command will create that service for us.

<pre class="wp-block-code"><code>kubectl expose deployment helloworld --name helloworld --namespace ingress-basic --port 80</code></pre>

Now that we have our service we need to deploy an ingress controller so we can access it. An ingress controller will direct external traffic to our service, you can find out more about ingress and kubernetes [here](https://kubernetes.io/docs/concepts/services-networking/ingress/). 

Firstly we are going to deploy a static public ip in Azure, we can then point a domain name at that ip address which is needed for our ingress controller. When we deployed our AKS instance it created a second resource group and deployed the nodes within it so we need to retrieve the name of that resource group, we can do that with the following command.

<pre class="wp-block-code"><code>az aks show --resource-group AKS --name AKS --query nodeResourceGroup -o tsv</code></pre>

Mine came out as &#8216;MC\_AKS\_AKS\_westeurope&#8217; so hopefully yours will be fairly similar. Let&#8217;s now deploy a public ip address to Azure (substitute MC\_AKS\_AKS\_westeurope with the resource group name returned by the previous command). This command will return the IP address so keep a note of that for future use.

<pre class="wp-block-code"><code>az network public-ip create --resource-group MC_AKS_AKS_westeurope --name AKSPublicIP --sku Standard --allocation-method static --query publicIp.ipAddress -o tsv</code></pre>

Now we&#8217;re going to run a few Helm commands. I&#8217;m going to cd into my Helm folder but if you added Helm to your Windows path variable then ignore the .\ at the start of the Helm commands.

Let&#8217;s start by adding the stable charts repo. This will allow us to pull Helm charts from that repository.

<pre class="wp-block-code"><code>.\helm repo add stable https://kubernetes-charts.storage.googleapis.com/</code></pre>

Run repo update to get the latest information about those charts.

<pre class="wp-block-code"><code>.\helm repo update</code></pre>

We can now deploy Nginx to our AKS instance using Helm. Make sure you set the namespace parameter to the same one you created earlier. Set the loadBalancerIP parameter the the IP address of your Azure public IP.

<pre class="wp-block-code"><code>.\helm install nginx-ingress stable/nginx-ingress --namespace ingress-basic --set controller.replicaCount=2 --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux --set controller.service.loadBalancerIP="51.105.209.106"</code></pre>

Once completed we can check Nginx has deployed successfully by running the following command:

<pre class="wp-block-code"><code>kubectl get service -l app=nginx-ingress --namespace ingress-basic</code></pre>

Now that we have a running ingress controller we need to assign a dns name to our Azure public IP &#8211; this is then used by Nginx to direct traffic to our deployment. Make sure you update the IP address in the following command to the one you created earlier, also take note of the &#8211;dns-name parameter and set it to something unique.

<pre class="wp-block-code"><code>az network public-ip update --ids (az network public-ip list --query "[?ipAddress!=null]|[?contains(ipAddress, '51.105.209.106')].[id]" --output tsv) --dns-name akstomaustin</code></pre>

This created me a DNS address of &#8216;akstomaustin.westeurope.cloudapp.azure.com&#8217; and yours should be fairly similar.

The last thing we need to do is apply configuration to our ingress controller. To do this we are going to use a local yaml file, a future article I will show you how to apply ingress configuration using an [Azure DevOps Pipeline](https://azure.microsoft.com/en-gb/services/devops/pipelines/) but applying it locally is fine for now. I have created a sample ingress.yaml file for you which you can find [here](https://gist.github.com/tomaustin700/16d99169a7792ccd239f648af3f29f2d), simply save this file locally as ingress.yaml and update the host parameter with the dns address you just created. Now all we need to do is apply that file using the kubectl apply command.

<pre class="wp-block-code"><code>kubectl apply -f C:\Users\Tom\Desktop\ingress.yaml --namespace ingress-basic</code></pre>

Once applied you should finally be able to access your deployment by putting the DNS address into your browsers address bar and hitting enter! Hopefully you get the following response:<figure class="wp-block-image size-large">

<img loading="lazy" width="845" height="283" src="http://tomaustin.xyz/wp-content/uploads/2020/03/image.png" alt="" class="wp-image-566" srcset="https://tomaustin.xyz/wp-content/uploads/2020/03/image.png 845w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-300x100.png 300w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-768x257.png 768w, https://tomaustin.xyz/wp-content/uploads/2020/03/image-720x241.png 720w" sizes="(max-width: 845px) 100vw, 845px" /> </figure> 

Congratulations!

This is all working over HTTP and really we want HTTPS. There are a few ways you can configure HTTPS; I normally use [Cloudflare](https://www.cloudflare.com/) which makes things nice and easy however you can also use [Let&#8217;s Encrypt](https://letsencrypt.org/) and run cert-manager within AKS. Both of these approaches are outside the scope of this tutorial and there are many well written guides for both approaches which can be found with a quick google search 🙂