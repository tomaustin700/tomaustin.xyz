---
id: 359
title: Deploying a website to Azure Kubernetes Service (AKS) using Azure DevOps
date: 2019-05-18T18:27:45+01:00
author: tom
layout: post
guid: http://3.10.198.211/?p=359
permalink: /2019/05/18/deploying-a-website-to-azure-kubernetes-service-aks-using-azure-devops/
image: /wp-content/uploads/2019/05/aks.png
categories:
  - azure
  - devops
  - kubernetes
---
 

In this series we will be exploring how to set up a CI/CD pipeline to deploy websites and services to [Azure Kubernetes Service](https://azure.microsoft.com/en-gb/services/kubernetes-service/) using [Azure DevOps](https://azure.microsoft.com/en-gb/services/devops/). I&#8217;m going to be using [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) with the Azure Development workload, you&#8217;re also going to need the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest) and [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows) for future articles in this series. This first article will guide you through deploying a basic website to AKS using Azure DevOps pipelines. You won&#8217;t need to be overly familiar with Kubernetes to follow this however a basic understanding might help. 

Let&#8217;s start by navigating to the Azure portal and creating a new AKS cluster (Kubernetes services > Add). Create a new Resource Group for your cluster, fill in the mandatory fields and set the node size and count.<figure class="wp-block-image">

<img loading="lazy" width="774" height="629" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-18.png" alt="" class="wp-image-366" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-18.png 774w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-18-300x244.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-18-768x624.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-18-720x585.png 720w" sizes="(max-width: 774px) 100vw, 774px" /> </figure> 

Hit &#8216;Next&#8217; until you are at the Networking page. I&#8217;m going to turn on HTTP application routing as it makes things slightly easier to get setup and test (a future article will replace this setting with our own ingress controller).<figure class="wp-block-image">

<img loading="lazy" width="511" height="278" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-19.png" alt="" class="wp-image-367" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-19.png 511w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-19-300x163.png 300w" sizes="(max-width: 511px) 100vw, 511px" /> </figure> 

Hit &#8216;Review and create&#8217; and let Azure run its validation (sometimes validation will fail for no reason so if this happens just press &#8216;Previous&#8217; and then &#8216;Review and create&#8217; again). Once validation has passed finally press &#8216;Create&#8217;. It will take about 10 mins for Azure to provision everything so whilst that is going on let&#8217;s get started with Visual Studio.

Open up Visual Studio, select &#8216;Create a new project&#8217; and then select the &#8216;ASP.NET Core Web Application&#8217; template.<figure class="wp-block-image">

<img loading="lazy" width="615" height="307" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-20.png" alt="" class="wp-image-369" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-20.png 615w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-20-300x150.png 300w" sizes="(max-width: 615px) 100vw, 615px" /> </figure> 

Give your project a name and press &#8216;Create&#8217;. On the next page select &#8216;Web Application&#8217; and also enable docker support for Linux.<figure class="wp-block-image">

<img loading="lazy" width="978" height="178" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-21.png" alt="" class="wp-image-370" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-21.png 978w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-21-300x55.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-21-768x140.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-21-720x131.png 720w" sizes="(max-width: 978px) 100vw, 978px" /> </figure> 

After the project has been created we will have a very basic web application. Feel free to press F5 and see what it looks like. One thing to note is that Visual Studio has generated us a Dockerfile which will be used to run our website in a containerised form.

Next right click on the application, press &#8216;Add&#8217; and select &#8216;Container Orchestration Support&#8217;.

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-22.png" alt="" class="wp-image-372" width="339" height="351" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-22.png 602w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-22-290x300.png 290w" sizes="(max-width: 339px) 100vw, 339px" /></figure>
</div>

When prompted make sure the container orchestrator is set to &#8216;Kubernetes/Helm&#8217;. Visual Studio will now generate us a helm chart which will allow us to deploy our application to our Kubernetes cluster. We should now have a folder in the solution called &#8216;charts&#8217;, this contains a collection of yaml files which describe our intended deployment. Let&#8217;s tweak a few of these!

Open up values.yaml and set replicaCount to 2 (this will create two pods for our website) and set the image, imagePullSecrets, service and ingress sections to the following.

<pre class="wp-block-code"><code>image:
  repository: VALUE_TO_BE_OVERRIDDEN
  tag: latest

imagePullSecrets: []
  # Optionally specify an array of imagePullSecrets.
  # Secrets must be manually created in the namespace.
  # ref: https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod
  #
  # This uses credentials from secret "myRegistryKeySecretName".
  # - name: myRegistryKeySecretName
service:
  port: 80

ingress:
  enabled: false
  annotations:
    kubernetes.io/ingress.class: addon-http-application-routing
  path: /
  hostname: VALUE_TO_BE_OVERRIDDEN</code></pre>

A full copy of my values.yaml file can be found [here](https://gist.github.com/tomaustin700/d75cb6b3746a160b8570886e54b97333). We are going to be overriding a lot of these values as part of our CI/CD pipeline which is why some are set to &#8216;VALUE\_TO\_BE_OVERRIDDEN&#8217;. Next open up deployment.yaml and update the contents to match what I have [here](https://gist.github.com/tomaustin700/bb18dbf1142b8b8b977b59bfecda8f7c) &#8211; remember to replace &#8216;kuberwebsite&#8217; with your website name.

ingress.yaml is pretty close to how we want it, I&#8217;d recommend changing the rules section to the following as it will pull the servicePort value from our values file (full copy [here](https://gist.github.com/tomaustin700/66544ccc002b7c2a2b314b9309180b41)).

<pre class="wp-block-code"><code>rules:
  {{- range .Values.ingress.hosts }}
    - host: {{ . }}
      http:
        paths:
          - path: {{ $ingressPath }}
            backend:
              serviceName: {{ $fullName }}
              servicePort: http
  {{- end }}</code></pre>

We are now ready to push this code to Azure DevOps. At the bottom right of Visual Studio select &#8216;Add to Source Control&#8217; and then &#8216;Git&#8217;. When prompted press &#8216;Publish Git Repo&#8217; under &#8216;Azure DevOps&#8217;.

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="380" height="142" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-23.png" alt="" class="wp-image-374" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-23.png 380w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-23-300x112.png 300w" sizes="(max-width: 380px) 100vw, 380px" /></figure>
</div>

Select your organisation and give your repository a name, then press &#8216;Publish Repository&#8217;.

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="385" height="237" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-24.png" alt="" class="wp-image-375" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-24.png 385w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-24-300x185.png 300w" sizes="(max-width: 385px) 100vw, 385px" /></figure>
</div>

Once pushed we should be able to open up Azure DevOps and see our new project along with the code we just pushed.<figure class="wp-block-image">

<img loading="lazy" width="639" height="327" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-25.png" alt="" class="wp-image-376" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-25.png 639w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-25-300x154.png 300w" sizes="(max-width: 639px) 100vw, 639px" /> </figure> 

Before going any further let&#8217;s go back to Azure and check on our deployment. Hopefully the cluster will have been provisioned! <figure class="wp-block-image">

<img loading="lazy" width="818" height="318" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-26.png" alt="" class="wp-image-377" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-26.png 818w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-26-300x117.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-26-768x299.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-26-720x280.png 720w" sizes="(max-width: 818px) 100vw, 818px" /> </figure> 

Whilst we are in the Azure Portal let&#8217;s quickly make a &#8216;Container registry&#8217; to store our images. Give your registry a unique name and add it to our previously created resource group (I&#8217;ve also set my SKU to basic) before pressing &#8216;Create&#8217;.

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-27.png" alt="" class="wp-image-378" width="351" height="353" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-27.png 397w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-27-150x150.png 150w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-27-298x300.png 298w" sizes="(max-width: 351px) 100vw, 351px" /></figure>
</div>

Back to Azure DevOps now and let&#8217;s navigate to the Pipelines page. When you are there press the &#8216;Create Pipeline&#8217; button. We are going to be using the classic pipelines editor for this tutorial as its slightly easier to follow so let&#8217;s press &#8216;Use the classic editor&#8217; at the bottom.

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="574" height="129" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-28.png" alt="" class="wp-image-380" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-28.png 574w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-28-300x67.png 300w" sizes="(max-width: 574px) 100vw, 574px" /></figure>
</div>

On the next page leave everything at their default values and press &#8216;Continue&#8217;. When prompted to select a template let&#8217;s go with &#8216;Docker container&#8217;.<figure class="wp-block-image">

<img loading="lazy" width="602" height="128" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-30.png" alt="" class="wp-image-382" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-30.png 602w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-30-300x64.png 300w" sizes="(max-width: 602px) 100vw, 602px" /> </figure> 

Once at the Task page select &#8216;Build an image&#8217; &#8211; we need to set some configuration here! Start by setting the Task version to 1.* and then set your Azure subscription (if it&#8217;s not listed press &#8216;Manage&#8217;), authorise it if you need to. Then select the Azure Container Registry we previously created, select the Dockerfile by pressing the &#8230; button and then set the &#8216;Image name&#8217; field to your container login server (this can be seen when you select the container registry in the Azure Portal) followed by a &#8216;/&#8217;, your website name followed by :$(Build.BuildId). Finally un-tick &#8216;use default build context&#8217;. You should have something which looks like this:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-33.png" alt="" class="wp-image-386" width="372" height="540" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-33.png 486w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-33-207x300.png 207w" sizes="(max-width: 372px) 100vw, 372px" /></figure>
</div>

Now let&#8217;s configure the &#8216;Push an image&#8217; task. A lot of the fields can be set to the same values as the fields in the &#8216;Build an image&#8217; task so replicate the Azure subscription, Azure Container Registry and the Image Name. Make sure the command is set to push and you should be set.

The final step is to add a new task to publish our Helm chart. Press the + button to add a new task and select &#8216;Publish Build Artifacts&#8217;. Once added select the task and change &#8216;Path to publish&#8217; to your charts directory and set the &#8216;Artifact name&#8217; field to &#8216;Helm&#8217;.

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-34.png" alt="" class="wp-image-388" width="407" height="309" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-34.png 475w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-34-300x228.png 300w" sizes="(max-width: 407px) 100vw, 407px" /></figure>
</div>

Let&#8217;s quickly turn on continuous integration by navigating to the &#8216;Triggers&#8217; tab and toggling &#8216;Enable continuous integration&#8217;. This will instruct Azure DevOps to build and push our container whenever we push code changes to the repo.

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-35.png" alt="" class="wp-image-390" width="504" height="270" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-35.png 574w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-35-300x160.png 300w" sizes="(max-width: 504px) 100vw, 504px" /></figure>
</div>

We can now give our pipeline a name and press &#8216;Save & queue&#8217; to give it a test. Azure DevOps will now build our container and push it to our container registry. Hopefully we get a nice green tick showing everything has worked!<figure class="wp-block-image">

<img loading="lazy" width="608" height="221" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-36.png" alt="" class="wp-image-392" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-36.png 608w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-36-300x109.png 300w" sizes="(max-width: 608px) 100vw, 608px" /> </figure> 

Now we need to configure a release pipeline to deploy our container to the Kubernetes cluster. Navigate to &#8216;Releases&#8217; and press &#8216;New pipeline&#8217;, when prompted to select a template just press &#8216;Empty job&#8217;. 

Let&#8217;s start by adding an artifact. Set the &#8216;Source&#8217; field to the build pipeline we previously created and press &#8216;Add&#8217;.

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-37.png" alt="" class="wp-image-393" width="344" height="429" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-37.png 496w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-37-240x300.png 240w" sizes="(max-width: 344px) 100vw, 344px" /></figure>
</div>

Before we add any tasks let&#8217;s quickly set a continuous deployment trigger by clicking the lightning bolt symbol on our artifact and enabling the trigger.

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="433" height="194" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-45.png" alt="" class="wp-image-408" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-45.png 433w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-45-300x134.png 300w" sizes="(max-width: 433px) 100vw, 433px" /></figure>
</div>

Under &#8216;Stage 1&#8217; press &#8216;1 job, 0 task&#8217; and then we can start adding some tasks. First we need to setup the Tiller environment so add the &#8216;Deploy to Kubernetes task&#8217;, set the &#8216;Service connection type&#8217; to &#8216;Azure Resource Manager&#8217; and select the correct subscription, resource group and Kubernetes cluster. Set namespace to &#8216;$(namespace)&#8217; &#8211; we will add a variable for it later on and set &#8216;Command&#8217; to apply. Now select &#8216;Use configuration&#8217; and set &#8216;Configuration type&#8217; to inline. Copy and paste [this text](https://gist.github.com/tomaustin700/c04a6fb95d548a935c771e1b9cdbc8c9) into the &#8216;Inline configuration&#8217; field. Hopefully you should have something which looks like this:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-38.png" alt="" class="wp-image-395" width="421" height="648" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-38.png 471w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-38-195x300.png 195w" sizes="(max-width: 421px) 100vw, 421px" /></figure>
</div>

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-39.png" alt="" class="wp-image-396" width="397" height="368" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-39.png 477w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-39-300x278.png 300w" sizes="(max-width: 397px) 100vw, 397px" /></figure>
</div>

Now we need to pull some secrets from Azure so add another &#8216;Deploy to Kubernetes&#8217; task and set the Kubernetes values to the same ones we set on the previous task. In the &#8216;Command&#8217; field set it to &#8216;get&#8217; and set &#8216;Arguments&#8217; to &#8216;service&#8217;.

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-40.png" alt="" class="wp-image-398" width="411" height="241" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-40.png 492w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-40-300x176.png 300w" sizes="(max-width: 411px) 100vw, 411px" /></figure>
</div>

Now expand the &#8216;Secrets&#8217; section and select the subscription and registry, in the &#8216;Secret name&#8217; field type &#8216;$(dockerAuthSecretName)&#8217;.

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-41.png" alt="" class="wp-image-400" width="391" height="425" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-41.png 466w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-41-276x300.png 276w" sizes="(max-width: 391px) 100vw, 391px" /></figure>
</div>

Before we add anymore tasks let&#8217;s quickly add the variables we have specified. Navigate to the &#8216;Variables&#8217; tab and add two variables; one for dockerAuthSecretName and one for namespace. Set &#8216;dockerAuthSecretName&#8217; to the name of your Kubernetes cluser followed by docker auth and set namespace to the name of your website.<figure class="wp-block-image is-resized">

<img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-42.png" alt="" class="wp-image-401" width="565" height="152" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-42.png 589w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-42-300x81.png 300w" sizes="(max-width: 565px) 100vw, 565px" /> </figure> 

Now let&#8217;s go back to the &#8216;Tasks&#8217; tab and the &#8216;Helm tool installer&#8217; task, leave everything at their default values. Now add two &#8216;Package and deploy helm charts&#8217; tasks.

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-43.png" alt="" class="wp-image-404" width="488" height="322" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-43.png 508w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-43-300x198.png 300w" sizes="(max-width: 488px) 100vw, 488px" /></figure>
</div>

In the first task select your Azure subscription, resource group and Kubernetes cluster. Set the &#8216;Command&#8217; value to &#8216;init&#8217;, un-tick &#8216;Upgrade Tiller&#8217; and type the following in the Arguments box.

<pre class="wp-block-code"><code>--service-account tiller</code></pre>

You should have something which looks like this:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-44.png" alt="" class="wp-image-405" width="366" height="441" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-44.png 482w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-44-249x300.png 249w" sizes="(max-width: 366px) 100vw, 366px" /></figure>
</div>

Now expand the &#8216;Advanced group and set the &#8216;Tiller namespace&#8217; field to $(namespace).

On the second Helm task set the subscription, resource group and cluster fields like we have previously set. Set Namespace to $(namespace), &#8216;Command&#8217; to &#8216;upgrade&#8217; and &#8216;Chart Type&#8217; to &#8216;File Path&#8217;. Now select your chart artifacts using the &#8230; button. Set &#8216;Release Name&#8217; to your website name. Now select &#8216;Install if release not present&#8217; and &#8216;Force&#8217; before typing the following into the &#8216;Arguments&#8217; box. Make sure you substitute the image.repository value with your own and set ingress.hostname to the HTTP application routing domain of your cluster (this can be found when you select your cluster in the Azure Portal).

<pre class="wp-block-code"><code>--set image.repository=kubertutorial.azurecr.io/kuberwebsite --set image.tag=$(Build.BuildId) --set service.port=80 --set ingress.enabled=true --set ingress.hostname=$(namespace).3bab45c18a7547e99d6c.westeurope.aksapp.io --set imagePullSecrets={$(dockerAuthSecretName)} --timeout 900</code></pre>

These arguments are overriding some of the values we have set previously in our values.yaml file. Now expand the &#8216;Advanced&#8217; group and set &#8216;Tiller namespace&#8217; to $(namespace). Now give your pipeline a name (I chose &#8216;CD&#8217;) and press &#8216;Release&#8217; &#8211; &#8216;Create release&#8217;. Now we wait patiently. Hopefully after a few minutes the release completes and we have a lot of green ticks!

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="568" height="547" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-46.png" alt="" class="wp-image-414" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-46.png 568w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-46-300x289.png 300w" sizes="(max-width: 568px) 100vw, 568px" /></figure>
</div>

If we expand the &#8216;helm upgrade&#8217; section we should be able to see that we have two pods running with our container in and also the ingress hostname.<figure class="wp-block-image">

<img loading="lazy" width="653" height="315" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-47.png" alt="" class="wp-image-416" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-47.png 653w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-47-300x145.png 300w" sizes="(max-width: 653px) 100vw, 653px" /> </figure> 

It will take a few minutes for DNS to start working but once it does you should be able to access your website by navigating to the ingress hostname using your browser.<figure class="wp-block-image">

<img loading="lazy" width="962" height="573" src="http://tomaustin.xyz/wp-content/uploads/2019/05/image-48.png" alt="" class="wp-image-420" srcset="https://tomaustin.xyz/wp-content/uploads/2019/05/image-48.png 962w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-48-300x179.png 300w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-48-768x457.png 768w, https://tomaustin.xyz/wp-content/uploads/2019/05/image-48-720x429.png 720w" sizes="(max-width: 962px) 100vw, 962px" /> </figure> 

All of the source code for this article can be found [here](https://dev.azure.com/tomaustin700/_git/kubertutorial). The next article in this series will guide you through deploying an api for your website and using kubectl to manage your cluster.