---
layout: post
title: Running A GitHub Pages Dev Environment On Windows
date: 2018-02-04
tags: [GitHub Pages]
excerpt: |
  Running a GitHub Pages dev environment on Windows used to be prohibitively difficult for a person who did not have
  working knowledge of each piece involved. Now that Windows can run Bash, this is no longer the case. Here are
  step by step instructions on how to do exactly that, written for a primarily Windows-based user.
---
<p>
Running a GitHub Pages development environment on your machine is essential for troubleshooting compilation issues and previewing your changes before deploying. It's always been an absolute headache to set this environment up on Windows, since it's running on Ruby which generally favors Not Windows. However, running on Bash On Ubuntu On Windows (that's a mouthful), this is pretty easy.
</p>
<p>
These instructions apply to Windows 10 64-bit, completely updated and patched. I say this because on earlier versions of Windows 10 it's considered "beta" and the installation instructions are different.
</p>
<h3>Step 1: Installing Windows Subsystem For Linux</h3>
<p>
In Windows, search for "Turn Windows features on or off". In that dialog, select "Windows Subsystem For Linux" and click OK. You should get a prompt to restart your machine; do this now.
</p>
<h3>Step 2: Installing and Configuring Ubuntu</h3>
<p>
This is pretty easy. Once Windows restarts, open the Windows Store, search for Ubuntu, and choose the Ubuntu app. Install Ubuntu from the Windows Store. After installation is complete, click Launch. This will initialize Ubuntu and prompt you to create credentials. These credentials do not have to match your Windows credentials.
</p>
<h3>Step 3: Configuring GitHub Pages Environment</h3>
<p>
At this point you are left with an Ubuntu Bash screen. I'm weird, so I usually close this screen, launch PowerShell, type bash, and hit enter. This runs Bash through PowerShell, which is a bit more familiar to me.
</p>
<p>
From here I generally follow the published instructions from GitHub for <a href="https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/">Setting up your GitHub Pages site locally with Jekyll</a>. I'll summarize: I'm going to install Ruby with the following commands one-by-one:
<script src="https://gist.github.com/mikecole/f084f86a2afe834e27022f500a4942d7.js"></script>
</p>
<p>
And I'll verify with:
<script src="https://gist.github.com/mikecole/fbcc1b2e4f1322f3526f54d1e0e5231c.js"></script>
</p>
<p>
And then I'll install bundler:
<script src="https://gist.github.com/mikecole/766e2275898c79711840ddbcefbf21e3.js"></script>
</p>
<h4>Profit</h4>
<p>
We should have everything we need now to run the GitHub Pages dev environment on our machines. I already have a repo setup, so I'll clone that. I'll navigate to the root of my repo in Bash, and the first thing I'll do is install the dependencies using Bundler:
<script src="https://gist.github.com/mikecole/6186cbe4908de44c056e9b5542910379.js"></script>
</p>
<p>
Now we're ready to start Jekyll. We'll start it in watch mode, so when we change site files it will detect and automatically compile the site. This will output the url we'll use to navigate to the site, which is typically http://127.0.0.1 It will also output any compile errors:
<script src="https://gist.github.com/mikecole/6cc55122b6cb25f50cc2de7a83359f39.js"></script>
</p>
<p>
If you're working with simple sites, this might not even necessary since you can push right to a GitHub acceptance environment (blog post later on this...) and view the changes live. However, if you run into compilation issues or just want to view your changes before making public, this is the route you should take.
</p>