---
id: 839
title: Setting Runtime Demands on Azure DevOps YAML Pipelines
date: 2021-10-27T15:21:48+01:00
author: tom
layout: revision
guid: https://tomaustin.xyz/?p=839
permalink: /?p=839
---
After using Azure DevOps YAML Pipelines for a few years it took me an embarrassingly long time to realise that it wasn&#8217;t possible to set demands when manually queueing a pipeline like it was when queueing a classic pipeline, this change kind-of made sense to me as everything should be declared in code right? Well variables can be declared at runtime and declared within YAML just like they could with classic pipelines so the change to demands was confusing and somewhat frustrating. After scratching my head for a few hours I came up with a solution that restored runtime demands and thought it was worth a share as it&#8217;s not straightforward.

We&#8217;re going to need to make a few changes to our YAML to put runtime demands back, firstly we need to add a parameter. If you&#8217;ve never used a parameter before it allows control of what values can be passed to a pipeline, they work similarly to variables however you can predefined values. More info on parameters can be found [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/runtime-parameters?view=azure-devops&tabs=script). Here is our starting pipeline:

<pre class="wp-block-code"><code>name: Pipeline_$(Build.SourceBranchName).$(Year:yy)$(DayOfYear)$(Rev:.r)

trigger: none

stages:
- stage: Build
  jobs:
  - job: "Build"
    workspace:
      clean: all
    pool:
      name: PoolName
    steps:
    - checkout: self
    - powershell: 'write-host "TEST"'
      displayName: 'PowerShell Script'
   
</code></pre>

Let&#8217;s add a parameters block and declare a &#8216;Demands&#8217; parameter.

<pre class="wp-block-code"><code>name: Pipeline_$(Build.SourceBranchName).$(Year:yy)$(DayOfYear)$(Rev:.r)

trigger: none

parameters:
- name: Demands
  displayName: 'Demands'
  type: object
  default: ''

stages:
- stage: Build
  jobs:
  - job: "Build"
    workspace:
      clean: all
    pool:
      name: PoolName

    steps:
    - checkout: self
    - powershell: 'write-host "TEST"'
      displayName: 'PowerShell Script'
   
</code></pre>

The one thing to be aware of here is that type is set to object, it looks like it would make more sense to be a string parameter however a string parameter cannot use &#8221; as a default value. We want &#8221; as a default value as we may not want to supply any runtime demands and we still want the pipeline to run correctly. Next we need to add a variable that links to our parameter:

<pre class="wp-block-code"><code>name: Pipeline_$(Build.SourceBranchName).$(Year:yy)$(DayOfYear)$(Rev:.r)

trigger: none

parameters:
- name: Demands
  displayName: 'Demands'
  type: object
  default: ''

variables:
  Demands: '${{ parameters.Demands }}'
  
stages:
- stage: Build
  jobs:
  - job: "Build"
    workspace:
      clean: all
    pool:
      name: PoolName
    steps:
    - checkout: self
    - powershell: 'write-host "TEST"'
      displayName: 'PowerShell Script'
   
</code></pre>

For some reason you have to use a variable to set the demand but that variable has to be linked to a parameter, I tried many ways of getting this to work and this was the only way that worked. Finally we need to use an if statement to conditionally set demands if our Demands variable has a value set.

<pre class="wp-block-code"><code>name: Pipeline_$(Build.SourceBranchName).$(Year:yy)$(DayOfYear)$(Rev:.r)

trigger: none

parameters:
- name: Demands
  displayName: 'Demands'
  type: object
  default: ''

variables:
  Demands: '${{ parameters.Demands }}'
  

stages:
- stage: Build
  jobs:
  - job: "Build"
    workspace:
      clean: all
    pool:
      name: PoolName
      ${{ if ne(variables&#91;'Demands'], '') }}:
        demands: ${{variables.Demands}}
    steps:
    - checkout: self
    - powershell: 'write-host "TEST"'
      displayName: 'PowerShell Script'
   
</code></pre>

That&#8217;s all there is to it, commit the YAML and manually queue the pipeline. You will now see a &#8216;Demands&#8217; box you can use to set runtime demands:

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="https://tomaustin.xyz/wp-content/uploads/2021/10/image-1024x663.png" alt="" class="wp-image-838" width="548" height="354" srcset="https://tomaustin.xyz/wp-content/uploads/2021/10/image-1024x663.png 1024w, https://tomaustin.xyz/wp-content/uploads/2021/10/image-300x194.png 300w, https://tomaustin.xyz/wp-content/uploads/2021/10/image-768x498.png 768w, https://tomaustin.xyz/wp-content/uploads/2021/10/image.png 1244w" sizes="(max-width: 548px) 100vw, 548px" /></figure>
</div>

It&#8217;s unfortunate that Azure DevOps prevents setting of runtime demands for YAML pipelines but at least there is a workaround so if you do need to use them it can be done. If you have any questions or comments then please leave a comment or contact me on [Twitter](https://twitter.com/tomaustin700). Thanks.