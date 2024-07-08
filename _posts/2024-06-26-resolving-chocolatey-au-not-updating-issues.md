---
layout: post
title: Resolving chocolatey-au Not Updating Issues
date: 2024-07-05
tags: chocolatey appveyor
meta-description: I ran into an issue with chocolatey-au not being able to automatically update Chocolatey packages when running on AppVeyor. Here is how I was able to troubleshoot and solve the issue.
---
I maintain several [Chocolatey](https://chocolatey.org/) community packages on [GitHub](https://github.com/mikecole/chocolatey-packages). I am using the [chocolatey-au](https://github.com/chocolatey-community/chocolatey-au) tool to automatically update new versions of software, running in an [AppVeyor scheduled build](https://ci.appveyor.com/project/mikecole/chocolatey-packages). A few months ago the Zoom package stopped automatically updating, and it appeared that the AppVeyor build agent was not seeing the latest version of the [Zoom version feed](https://zoom.us/rest/download?os=win`). After troubleshooting what I thought was a caching issue, I realized that the build agent was still running on a Windows Server 2012 OS. It appears the Zoom version feed observes the user agent HTTP request header, and serves the latest version that your operating system can support. Once I updated the build agent to use the `Visual Studio 2022` image, the chocolatey-au tool worked as expected.

#TIL that you can RDP into an AppVeyor build agent to help troubleshoot issues. First you need to add the following to the `on_finish` block in your .yml file:

```powershell
on_finish:    
- ps: $blockRdp = $true; iex ((new-object net.webclient).DownloadString('https://raw.githubusercontent.com/appveyor/ci/master/scripts/enable-rdp.ps1'))
```

This will block the job from succeeding and output the RDP info in the console. Before re-running the job, you need to add an environment variable named `APPVEYOR_RDP_PASSWORD` to your build settings. The value of this will be the password of your RDP connection.