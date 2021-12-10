---
id: 667
title: 'YamlPipelinesValidator.dev &#8211; Quickly and easily test Azure DevOps YAML pipelines'
date: 2020-06-30T10:26:27+01:00
author: tom
layout: revision
guid: https://tomaustin.xyz/2020/06/30/653-autosave-v1/
permalink: /?p=667
---
I&#8217;ve been slowly migrating all my [Azure DevOps](https://azure.microsoft.com/en-us/services/devops/) pipelines to the new [YAML](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema) format and really been enjoying the benefits however there is currently no way to test a YAML pipeline before committing it. Having to commit the YAML before seeing if it&#8217;s valid may be fine if you&#8217;re just pushing to master and have no branch policies but if you have branch policies that require the use of pull requests (as you probably should) it soon becomes tedious when the pipeline fails due to it being invalid in some way.

What I really wanted was a way to test if the pipeline was syntactically correct before committing it but there didn&#8217;t seem to be a way to do this. Microsoft had started building that functionality into their on-premise agents however that was quickly abandoned. Azure DevOps do provide an [API for testing a yaml pipeline](https://docs.microsoft.com/en-us/azure/devops/release-notes/2020/sprint-165-update#preview-fully-parsed-yaml-document-without-committing-or-running-the-pipeline) however I wanted something a bit more user friendly. 

After some searching I realised there wasn&#8217;t really anything that had the capability to do what I wanted so I decided to build it. I built [YamlPipelinesValidator.dev](https://yamlpipelinesvalidator.dev/)! Now you can test your Azure DevOps YAML pipelines without committing anything! Underneath the UI it is just calling the Azure DevOps API so has the same limitations such as not being able to check the contents of variables or checking for correct values however it will stop you needing to commit over and over again just to check if it has the correct syntax or your task properties are correct!

All the code is available on my [GitHub](https://github.com/tomaustin700/YAMLPipelineValidator) so if you have any issues let me know here or raise an issue (or raise a PR). Hopefully this will help you with your YAML pipelines and stop the endless cycle of commit &#8211; raise PR &#8211; run &#8211; fail. Enjoy!