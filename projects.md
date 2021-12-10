---
id: 822
title: Projects
date: 2021-09-12T15:40:07+01:00
author: tom
layout: page
guid: https://tomaustin.xyz/?page_id=822
---
## [XPathValidator.Dev](https://xpathvalidator.dev/)

I do a fair bit of day-to-day work with using XPath and found myself needing a quick and easy way to validate XPath expressions, I found a few already existing websites that allowed this but I wanted something quick and easy with no frills so I decided to build my own. The site is written using ASP.NET Core and runs within Azure. I want to rewrite it at some point to use Azure Static Web Apps and Azure functions for the API and bring it up to .NET 5.0 as I&#8217;ve learnt a lot about hosting in Azure since I built the site. The project code can be found on GitHub [here](https://github.com/tomaustin700/xpathvalidator.dev).

## [YamlPiplelinesValidator.Dev](https://yamlpipelinesvalidator.dev/)

After building XPathValidator.dev I realised I could build something similar to validate Azure DevOps YAML pipelines as they were also something I use in some way each day. I found there was no easy way to validate YAML pipelines before committing them and this could result in frustration and an endless cycle of commit -> run -> fail -> commit -> run -> fail so YamlPipelinesValidator.Dev solved this issue. The site allows a YAML pipeline to be pasted into it and it will validate it using the Azure DevOps API. This site is also written using ASP.NET Core and runs on Azure and I&#8217;d like to convert it to Azure Static Web Apps and Azure Functions also. The project code can be found on GitHub [here](https://github.com/tomaustin700/yamlpipelinesvalidator.dev).

## [Azure Pipelines YAML Validator VS Code Extension](https://marketplace.visualstudio.com/items?itemName=TomAustin.azure-devops-yaml-pipeline-validator)

Copying YAML pipelines to a website to validate them was much better than not being able to validate them but I wanted to improve things further. I mostly use VS Code for writing YAML pipelines so a VS Code extension seemed like a great addition. The extension has proved fairly popular and has improved my workflow when working with YAML pipelines and makes validation much more seamless. The project code can be found on GitHub [here](https://github.com/tomaustin700/YAML-Pipeline-Validator-VS-Code-Extension). 

## [PeepQuote](https://github.com/tomaustin700/PeepQuote)

I am a huge fan of the Channel 4 sitcom Peep Show and wanted to build something around that written using Azure Functions. I&#8217;d recently received a book containing some of the scripts from the show so thought about doing an API that allowed querying the scripts using various search parameters. Script data is stored within a JSON file within Azure Blob Storage which is then queried via the Function API. The project code can be found on GitHub [here](https://github.com/tomaustin700/PeepQuote). 

## [HowManyInfected.Uk](https://www.howmanyinfected.uk/)

During the pandemic I wanted a quick and easy way to see the latest statistics provided by the British Government. Their page was good but I wanted something that could be glanced at and provide up-to-date data. I also wanted to work with Azure Static Web Apps so I built a static website which calls the governments API. The project code can be found on GitHub [here](https://github.com/tomaustin700/howmanyinfected.uk). 

## [Wikipedia On This Day Twitter Bot](https://twitter.com/wikionthisday)

After working a fair bit with Azure Function Timer Triggers I thought a good use case would be a Twitter bot that could post something on a schedule. I noticed that there was an API to get the Wikipedia &#8216;On This Day&#8217; data so paired them together to build this bot. Every hour it posts an &#8216;on this day&#8217; entry provided by Wikipedia. The project code can be found on GitHub [here](https://github.com/tomaustin700/WikiOnThisDay). 

## [HyperDeploy](https://github.com/tomaustin700/HyperDeploy)

A few years ago I had the need to build many Hyper V virtual machines in one go with no interaction and I struggled to find a tool that could help with that, Terraform had a provider but it was very basic at the time so I decided to have a go at building my own Powershell module infrastructure as code tool for Hyper V. I&#8217;d never attempted anything this complex with Powershell before and it was a large learning curve to get it to where it is today. It takes a JSON file containing what needs to be provisioned and can do advanced features such as creating duplicates from one entry. It needs further enhancement to work with a state file like Terraform does but it is fully functional and I use it as part of my work. The project code can be found on GitHub [here](https://github.com/tomaustin700/HyperDeploy).