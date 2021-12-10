---
id: 633
title: 'How to fix &#8220;app-name has no deployed releases&#8221; with Helm 3'
date: 2020-04-13T17:30:22+01:00
author: tom
layout: revision
guid: http://3.10.198.211/2020/04/13/627-autosave-v1/
permalink: /?p=633
---
Helm 3 has been out for a while now and for the most part it&#8217;s a good step up from Helm 2 however it does have one issue which if you don&#8217;t know how to deal with can be frustrating and difficult to resolve. The issue I&#8217;m referring to occurs when the first deployment of an app fails for some reason, subsequent deployments will then throw a failure message with the text &#8220;app-name has no deployed releases&#8221;.

This issue can easily occur if you&#8217;re using Helm to deploy from a pipeline and make a small mistake somewhere in the pipeline configuration process. This issue is well know and has a very active GitHub issue which can be found [here](https://github.com/helm/helm/issues/5595#issuecomment-612611077), there is also a pull request which attempts to resolve the issue however at time of writing it is yet to merge (see [PR7653](https://github.com/helm/helm/pull/7653)). If you read the GitHub issue and comments you&#8217;ll see there are a few solutions out there, I&#8217;ve not had much luck with any of them so I came up with my own. You&#8217;re going to need to have kubectl installed locally and configured for your Kubernetes instance. 

This fix is pretty much a clean-up operation and we are going to remove all traces of the broken deployment. For this example my app is called &#8220;akshelm3issue&#8221; and my namespace is &#8220;ingress-basic&#8221;. We are going to start by deleting the secret associated against our app so let&#8217;s list all secrets so we can find the one we want.

<pre class="wp-block-code"><code>kubectl get secrets --show-labels -n ingress-basic</code></pre>

This will list all the secrets for the namespace, the one we are after will contain our app name within it. <figure class="wp-block-image size-large">

<img loading="lazy" width="774" height="147" src="http://tomaustin.xyz/wp-content/uploads/2020/04/image-6.png" alt="" class="wp-image-629" srcset="https://tomaustin.xyz/wp-content/uploads/2020/04/image-6.png 774w, https://tomaustin.xyz/wp-content/uploads/2020/04/image-6-300x57.png 300w, https://tomaustin.xyz/wp-content/uploads/2020/04/image-6-768x146.png 768w, https://tomaustin.xyz/wp-content/uploads/2020/04/image-6-720x137.png 720w" sizes="(max-width: 774px) 100vw, 774px" /> </figure> 

Now we&#8217;ve got that we can start the clean-up. Let&#8217;s delete that secret&#8230;

<pre class="wp-block-code"><code>kubectl delete secret sh.helm.release.v1.akshelm3issue.v1 -n ingress-basic</code></pre>

The rest of the clean-up is pretty straight forward and all we need is the app name and the namespace so let&#8217;s get on with it.

<pre class="wp-block-code"><code>kubectl delete service akshelm3issue -n ingress-basic
kubectl delete deployment akshelm3issue -n ingress-basic
kubectl delete ingress akshelm3issue -n ingress-basic</code></pre>

You may get some messages stating that what you&#8217;ve attempted to delete does not exist, this will vary depending on how far your initial deployment got before it failed. That should be it! Fix whatever issue caused the initial deployment to fail and redeploy, hopefully you&#8217;re greeted with a nice green tick.

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="740" height="188" src="http://tomaustin.xyz/wp-content/uploads/2020/04/image-7.png" alt="" class="wp-image-630" srcset="https://tomaustin.xyz/wp-content/uploads/2020/04/image-7.png 740w, https://tomaustin.xyz/wp-content/uploads/2020/04/image-7-300x76.png 300w, https://tomaustin.xyz/wp-content/uploads/2020/04/image-7-720x183.png 720w" sizes="(max-width: 740px) 100vw, 740px" /></figure>
</div>