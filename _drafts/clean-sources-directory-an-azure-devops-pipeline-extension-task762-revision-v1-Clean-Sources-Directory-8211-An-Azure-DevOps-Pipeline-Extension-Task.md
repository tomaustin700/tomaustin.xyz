---
id: 810
title: 'Clean Sources Directory &#8211; An Azure DevOps Pipeline Extension Task'
date: 2021-06-07T16:14:50+01:00
author: tom
layout: revision
guid: https://tomaustin.xyz/?p=810
permalink: /?p=810
---
Update: I have now produced a new extension that cleans up all build directories and not just source, you can find this [here](https://marketplace.visualstudio.com/items?itemName=TomAustin.buildcleanup&targetId=8eefb3ec-c76a-4d9b-9ea7-f9cb7662c2e9&utm_source=vstsproduct&utm_medium=ExtHubManageList).

A while back I [wrote about the difficulty managing self-hosted Azure DevOps](https://tomaustin.xyz/2020/12/23/managing-disk-space-of-azure-devops-self-hosted-agents/) agents when it comes to disk space. By default when a pipeline runs it leaves the entire cloned git repository behind when it finishes and when you have large git repositories this can start to become a problem. Even after setting maintenance jobs to run every night you can still run into issues and it starts to become an uphill battle.

After not being 100% happy with the maintenance job solution I decided to take a look at writing my own extension task for Azure DevOps Pipelines, the extension would simply delete the content of the sources directory (the directory the git repository is cloned to). This would be set to run as the last task in the pipeline and remove any files that had no use anymore. I&#8217;d never written an extension for Azure DevOps before but I have written a [fairly popular one](https://marketplace.visualstudio.com/items?itemName=TomAustin.azure-devops-yaml-pipeline-validator) for VS Code and it was pretty easy, turns out Azure DevOps extensions are also pretty easy to write. What I came up with was [Clean Sources Directory](https://marketplace.visualstudio.com/items?itemName=TomAustin.cleansourceext).

Once you&#8217;ve installed the extension for the Azure DevOps extension marketplace into your organisation all you need to do is add it to your pipelines. For classic pipelines simply add the &#8216;Clean Sources Directory&#8217; task:

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="627" height="144" src="https://tomaustin.xyz/wp-content/uploads/2021/03/image.png" alt="" class="wp-image-764" srcset="https://tomaustin.xyz/wp-content/uploads/2021/03/image.png 627w, https://tomaustin.xyz/wp-content/uploads/2021/03/image-300x69.png 300w" sizes="(max-width: 627px) 100vw, 627px" /></figure>
</div>

And for YAML pipelines simply add the following YAML:

<pre class="wp-block-code"><code>steps:
- task: TomAustin.cleansourceext.cleansource-ta.CleanupSourcesDirectory@0
  displayName: 'Cleanup Sources Directory'
  condition: always()
</code></pre>

I&#8217;d recommend setting the always condition so even if your pipeline fails or you cancel it the sources directory will still be cleaned. That&#8217;s all there is to it!