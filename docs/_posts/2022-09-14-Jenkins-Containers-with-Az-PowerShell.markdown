---
layout: post
title:  "Jenkins Containers with Az Powershell"
date:   2022-09-14 08:47:29 +0000
---
Recently I've been working on some Jenkins pipelines. I wanted to manipulate some Azure resources as part of the pipeline execution - the solution decided on was to use an Az Powershell - enabled container to execute the code.

The Jenkins app is hosted in Azure Kubernetes Services which is why the executing agent is Kubernetes - some improvements to the below gist could include:

- Capturing Kubernetes agent YAML in a separate file
- Capturing desired PowerShell code in a file and referencing that for the container to execute
- Creation of a PowerShell Secret object to provide the Connect-AzAccount command - pulling from environmental variables etc.

{% gist 08f93b0ff9fcccf2de36aa4007c5fcdf %}
