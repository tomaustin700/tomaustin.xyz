---
id: 742
title: Deploying Selenium Grid to Azure Kubernetes Service (AKS) using Azure DevOps
date: 2021-01-29T16:28:40+00:00
author: tom
layout: revision
guid: https://tomaustin.xyz/2021/01/29/516-revision-v1/
permalink: /?p=742
---
**This article focuses on deploying Selenium Grid 3 which is now obsolete please see the [following article](https://tomaustin.xyz/?p=694) which shows the same deployment process for Selenium Grid 4.**

<https://tomaustin.xyz/2021/01/13/deploying-selenium-grid-4-to-azure-kubernetes-service-aks-using-azure-devops/>

I&#8217;ve recently started using [Selenium](https://selenium.dev/) for UI testing some web applications and I kept seeing [Selenium Grid](https://selenium.dev/documentation/en/grid/) being mentioned so decided to take a look at it. For those of you who don&#8217;t know about Selenium Grid it allows you to distribute your tests over several machines by sending your tests to a &#8216;hub&#8217; which will then distribute your tests for you over your &#8216;nodes&#8217;. The thing that really interested me about this was that everything could be run from Docker containers so within a few minutes I had all the infrastructure running locally via a compose file. This is all well and good but what if we wanted to have more than a handful nodes? AKS seemed like a good fit.

Before we start I&#8217;m going to presume you already have an AKS instance running with an ingress controller deployed, it would also be good if you were also familiar with kubectl (for debugging more than anything). One thing to be aware of is that each node container requires about 1GB of RAM so if you intend on running a lot you&#8217;re going to need a pretty powerful AKS instance. I have vm size set to B2ms for the instances within the scale set but even this won&#8217;t be enough if you want to run more than a few nodes.

Luckily for us Selenium already have [Docker containers](https://github.com/SeleniumHQ/docker-selenium) for the hub and the nodes so all we really need to do is deploy them to AKS. For this I&#8217;m going to be using Helm 3 as it&#8217;s quick and easy. We are going to need to [Helm](https://helm.sh/) charts to deploy but I&#8217;ve already done the hard work and built those, you can find them on my GitHub [here](https://github.com/tomaustin700/AKSSeleniumGrid). If you do intend on using my charts then you&#8217;re probably going to want to Fork the repo so you can tweak the replica count for yourself. 

Let&#8217;s go to [Azure DevOps](https://azure.microsoft.com/en-gb/services/devops/) and create a new project. Once completed we are going to want to build a pipeline and a release process. The pipeline will publish the charts and the release process will deploy the charts to AKS. Go to Pipelines and select the &#8216;New pipeline&#8217; button in the top right. I&#8217;m going to be using the classic pipeline editor as it&#8217;s a bit easier to follow along with so select &#8216;Use the classic editor&#8217; at the bottom.

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2020/02/image.png" alt="" class="wp-image-517" width="482" height="500" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image.png 494w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-289x300.png 289w" sizes="(max-width: 482px) 100vw, 482px" /></figure>
</div>

On the next page select GitHub as your source and select the Repository you cloned earlier.

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2020/02/image-1.png" alt="" class="wp-image-518" width="431" height="502" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image-1.png 505w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-1-258x300.png 258w" sizes="(max-width: 431px) 100vw, 431px" /></figure>
</div>

When prompted to select a template just click &#8216;Empty job&#8217;. We are only going to need one build task so no templates are required! Add the &#8216;Publish build artifacts&#8217; task and configure it to match the screenshot below.

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2020/02/image-2.png" alt="" class="wp-image-519" width="440" height="481" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image-2.png 510w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-2-275x300.png 275w" sizes="(max-width: 440px) 100vw, 440px" /></figure>
</div>

That&#8217;s pretty much it for the pipeline. You may want to turn on continuous integration if you want any changes to auto deploy (Triggers > Enable continuous integration). Lets hit &#8216;Save & queue&#8217; and hopefully after a few seconds the pipeline will complete with a nice green tick.

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2020/02/image-3.png" alt="" class="wp-image-521" width="657" height="137" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image-3.png 822w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-3-300x63.png 300w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-3-768x161.png 768w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-3-720x151.png 720w" sizes="(max-width: 657px) 100vw, 657px" /></figure>
</div>

Now let&#8217;s add a Release pipeline to deploy our charts. Click Releases and select click the + New button, then select &#8216;New release pipeline&#8217;. When prompted select &#8216;Empty job&#8217; like we did before. To start let&#8217;s add an artifact so click &#8216;Add an artifact&#8217; and select your build pipeline for the Source, it should auto-populate the rest of the fields and then click Add.

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2020/02/image-4.png" alt="" class="wp-image-522" width="522" height="568" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image-4.png 612w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-4-276x300.png 276w" sizes="(max-width: 522px) 100vw, 522px" /></figure>
</div>

At this point I always enable continuous deployment so click the trigger button (lightning bolt) and Enable the deployment. This will automatically deploy our Helm charts whenever new build artifacts are published.

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="396" height="356" src="http://tomaustin.xyz/wp-content/uploads/2020/02/image-5.png" alt="" class="wp-image-523" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image-5.png 396w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-5-300x270.png 300w" sizes="(max-width: 396px) 100vw, 396px" /></figure>
</div>

All we have to do now if add a few tasks to do the deployment. Start with clicking &#8216;1 job&#8217; under Stage 1, this should open the stage tasks page. The first task we are going to add is &#8216;Helm tool installer&#8217;, this will install Helm onto the build agent and allow it to deploy our charts. When the task is added it will default the Helm Version Spec to &#8216;2.14.1&#8217;, we are going to want to change this as we are going to want to use Helm 3 (Helm 3 has been released for a few months now and makes things a lot easier) so set it to latest.

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2020/02/image-6.png" alt="" class="wp-image-524" width="477" height="436" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image-6.png 516w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-6-300x274.png 300w" sizes="(max-width: 477px) 100vw, 477px" /></figure>
</div>

Next add a Kubectl task, this will deal with the authentication between the pipeline and our AKS instance. Set the &#8216;Service connection type&#8217; to &#8216;Azure Resource Manager&#8217; and select your Azure subscription (you may need to authorise it), once selected you should be able to select your Resource group and your cluster. The last thing to do is set the &#8216;Command&#8217; box to login.

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2020/02/image-7.png" alt="" class="wp-image-525" width="441" height="639" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image-7.png 493w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-7-207x300.png 207w" sizes="(max-width: 441px) 100vw, 441px" /></figure>
</div>

Next add the &#8216;Package and deploy Helm charts&#8217; task, we are going to start by deploying the Selenium hub. Select your Azure subscription, Resource group and Kubernetes cluster as you did before. In the &#8216;Namespace&#8217; box enter the namespace of your ingress controller (I&#8217;ve set mine to a pipeline variable but you can hard-code it if you want). Next set the &#8216;Command&#8217; to upgrade, &#8216;Chart Type&#8217; to &#8216;File Path&#8217; and select the Hub Charts folder, set &#8216;Release Name&#8217; to &#8216;selenium-hub&#8217; and tick &#8216;Install if release is not present&#8217; Your task config should look like the screenshot below.

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="493" height="625" src="http://tomaustin.xyz/wp-content/uploads/2020/02/image-8.png" alt="" class="wp-image-526" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image-8.png 493w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-8-237x300.png 237w" sizes="(max-width: 493px) 100vw, 493px" /></figure>
</div>

Right click the Helm task you just added and clone it twice. We are going to be deploying Chrome and Firefox nodes so we need one task each. One each task change the &#8216;Chart path&#8217; to point to the chart for the node you are deploying and change the &#8216;Release Name&#8217;. My Chrome deploy task looks like this:

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2020/02/image-9.png" alt="" class="wp-image-528" width="452" height="268" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image-9.png 469w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-9-300x178.png 300w" sizes="(max-width: 452px) 100vw, 452px" /></figure>
</div>

And Firefox is like this:

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2020/02/image-10.png" alt="" class="wp-image-529" width="424" height="264" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image-10.png 475w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-10-300x187.png 300w" sizes="(max-width: 424px) 100vw, 424px" /></figure>
</div>

If you chose to set the namespace from a variable like I did then make sure to set that in the variables tab and that should be it. Your Release pipeline should now look like the following screenshot.

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="525" height="369" src="http://tomaustin.xyz/wp-content/uploads/2020/02/image-14.png" alt="" class="wp-image-533" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image-14.png 525w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-14-300x211.png 300w" sizes="(max-width: 525px) 100vw, 525px" /></figure>
</div>

Save and then hit &#8216;Create release&#8217; in the top right. After a few moments the deployment should have succeeded. 

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="350" height="257" src="http://tomaustin.xyz/wp-content/uploads/2020/02/image-11.png" alt="" class="wp-image-530" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image-11.png 350w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-11-300x220.png 300w" sizes="(max-width: 350px) 100vw, 350px" /></figure>
</div>

Before running tests on our Grid we need to update the ingress controller to allow us to connect to the Hub. Before that though let&#8217;s double check everything deployed correctly. Open a terminal and use kubectl to get the running pods for your namespace, with any luck you&#8217;ll see your hub pod along with two Chrome pods and two Firefox pods (my namespace is ingress-basic so substitute for your namespace name) .

<pre class="wp-block-code"><code>kubectl get pods -n ingress-basic</code></pre>

We can see all our pods are successfully running.

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="http://tomaustin.xyz/wp-content/uploads/2020/02/image-12.png" alt="" class="wp-image-531" width="618" height="82" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image-12.png 670w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-12-300x40.png 300w" sizes="(max-width: 618px) 100vw, 618px" /></figure>
</div>

If you see that some of your pods aren&#8217;t running and are in a &#8216;pending&#8217; state. I&#8217;d guess that your Kubernetes nodes either don&#8217;t have enough memory resources or cpu resources free. You can quickly see this by running:

<pre class="wp-block-code"><code>kubectl describe node</code></pre>

Scroll to the &#8216;Allocated resouces&#8217; section and you will be able to see if you have overcommited resources, if so then your limits percentages will be greater than 100%. If this occurs you either have to increase the vm size for the instances within the scale set or deploy less replicas. If everything is within limits it should look like this:

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="451" height="142" src="http://tomaustin.xyz/wp-content/uploads/2020/02/image-15.png" alt="" class="wp-image-534" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image-15.png 451w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-15-300x94.png 300w" sizes="(max-width: 451px) 100vw, 451px" /></figure>
</div>

Now just configure your ingress controller by adding a rule for the hub service. [Here is a sample ingress.yam](https://gist.github.com/tomaustin700/b78d07137dcc92d3bf57c274e14c5139)l file which should point you in the right direction. Once configured you should be able to access the grid console by using the url specified in your ingress.yaml file followed by /grid/console. Hopefully it brings up the following page. I wouldn&#8217;t recommend having a publicly accessible Grid so if you&#8217;re running this in production make sure this isn&#8217;t available to the outside world.

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="https://i1.wp.com/tomaustin.xyz/wp-content/uploads/2020/02/image-13.png?fit=640%2C210&ssl=1" alt="" class="wp-image-532" width="739" height="242" srcset="https://tomaustin.xyz/wp-content/uploads/2020/02/image-13.png 1181w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-13-300x98.png 300w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-13-1024x336.png 1024w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-13-768x252.png 768w, https://tomaustin.xyz/wp-content/uploads/2020/02/image-13-720x236.png 720w" sizes="(max-width: 739px) 100vw, 739px" /></figure>
</div>

To run tests on your Grid simply use your url followed by /wd/hub and pass this to your RemoteWebDriver instance. If you want a really simple example of running tests on Selenium Grid there is an example c# repo [on my GitHub here](https://github.com/tomaustin700/SeleniumGridTest).