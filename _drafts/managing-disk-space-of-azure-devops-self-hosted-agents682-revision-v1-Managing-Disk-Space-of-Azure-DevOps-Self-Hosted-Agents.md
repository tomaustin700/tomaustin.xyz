---
id: 776
title: Managing Disk Space of Azure DevOps Self-Hosted Agents
date: 2021-03-26T15:23:56+00:00
author: tom
excerpt: How to manage disk space of Azure DevOps self-hosted agents using pipeline tasks and maintenance jobs.
layout: revision
guid: https://tomaustin.xyz/?p=776
permalink: /?p=776
---
Update: I&#8217;ve now released an Azure DevOps Extension to help with this, find out more [here](https://tomaustin.xyz/2021/03/14/clean-sources-directory-an-azure-devops-pipeline-extension-task/).

Part of my daily job involves managing [Azure DevOps](https://azure.microsoft.com/en-gb/services/devops/) pipelines and agents, something we have quite a lot of! I&#8217;d estimate nightly there are over a hundred pipelines that run and probably a further fifty run each day as part of pull request validation processes. Because all agents are currently self-hosted this can create a bit of an issue regarding disk space if you&#8217;re not careful and there are a couple of tips and tricks that can help with managing this.

Before getting into how to manage disk space it&#8217;s important to know how an Azure DevOps pipeline works. When a pipeline is queued it will execute a git fetch on the target branch, if you have quite a large git repository this can result in a large amount of files being pulled down onto the agent. These files will not be removed when the pipeline has completed which is why if you have a lot of pipelines you can run into disk issues.

Azure DevOps has a few ways to deal with this however it can also be confusing if you&#8217;re not overly familiar with pipeline settings and managing agents. 

The first thing you may come across is the &#8216;Clean&#8217; option on the pipeline which may sound exactly like what you want, you&#8217;d probably expect this to clean the working directory once the pipeline has completed &#8211; unfortunately this is not the case. Clean will only clean the working directory before the pipeline has ran meaning between pipeline runs your repository files will remain on the agent. Imagine having hundreds of different pipelines that rarely hit the same agent with a large git repository &#8211; it&#8217;s a recipe for your disk space to quickly vanish. 

If you&#8217;re using Classic Pipelines there is a marketplace task provided by Microsoft which can quickly resolve this called [Post Build Cleanup](https://marketplace.visualstudio.com/items?itemName=mspremier.PostBuildCleanup&targetId=8eefb3ec-c76a-4d9b-9ea7-f9cb7662c2e9). Post Build Cleanup looks at your pipelines clean settings and will clean the working directories when the task is ran meaning &#8211; excellent! Problem solved right? Not quite! Post Build Cleanup only support classic pipelines so if you are using YAML pipelines you&#8217;ve still got a problem. I&#8217;ve spoken to the the Microsoft Premier Services team about this and they have the following to say:

<div class="wp-block-group">
  <div class="wp-block-group__inner-container">
    <div class="wp-block-group">
      <div class="wp-block-group__inner-container">
        <div class="wp-block-group">
          <div class="wp-block-group__inner-container">
            <p>
              <em>&#8220;Thanks for contacting us. We recently received a similar report and ran a couple tests that revealed the following issues:</em>
            </p>
            
            <ol type="1">
              <li>
                <em>The workspace option of the job does not set the variable Build.Repostory.Clean to true. This is one of the predefined variables the task is looking for to check if clean is enabled. Thus, if you only use the workspace.clean parameter as in your definition below, the task simply assumes that clean is disabled and will not do anything.</em>
              </li>
              <li>
                <em>If you set the checkout.clean parameter to true, the agent does indeed set the Build.Repository.Clean variable to true so our task would run. However, in order to know how to clean the agent, our task looks for the cleanOptions property inside the repository properties of the build definition. Unfortunately, this property is not set for YAML pipelines, which is the reason that our task does not clean anything.</em>
              </li>
            </ol>
            
            <p>
              <em>We’d have to explicitly add code to handle YAML pipelines since there is not enough information in the build definition object returned by the API to figure out the correct cleanup strategy. The workspace option does not exist in the definition so we’d have to read the original YAML file and parse it in order to get the necessary information. Since YAML definitions can spread across multiple files, this isn’t as easy as it sounds. Thus, I’m not sure when we’ll be able to add proper YAML support for the task. I’m sorry I don’t have a better answer at the moment.&#8221;</em>
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

It looks like this isn&#8217;t something that will be resolved quickly then so we need to look at alternate solutions.

Luckily for us there is a built in Azure DevOps feature that can do what we need &#8211; just about! This feature is called [Maintenance Jobs](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/pools-queues?view=azure-devops&tabs=yaml%2Cbrowser). Maintenance Jobs run on your agents and will clean the directories for you, you can schedule these to run and even set how many days to keep unused working directories. Unfortunately Maintenance Jobs are hidden away in Azure DevOps settings and if you don&#8217;t know where they are they can be hard to find. Firstly navigate to your Organisation Settings and then select the &#8216;Agents pools&#8217; pane under pipelines, then select the &#8216;Settings&#8217; tab at the top &#8211; at the bottom of that page will be Maintenance jobs settings. 

I have my Maintenance Jobs set to run nightly and to only keep unused working directories for one day which is fairly aggressive but due to the large amount of pipelines and the large git repository size it is necessary. Unfortunately Maintenance Jobs can only be ran on a schedule and can&#8217;t be manually queued but with a bit of tweaking of the settings you should be able to find a good middle-ground.

I hope this article has helped you with managing your self hosted agents, ideally we&#8217;d all be using Microsoft hosted agents but due to the [large amount of agents my organisation needs and what we do with them](https://tomaustin.xyz/presentations/) it&#8217;s currently not feasible. Thanks.