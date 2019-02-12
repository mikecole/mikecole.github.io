---
layout: post
title: Incremental React Adoption in ASP.NET
date: 2019-02-12
tags: [ASP.NET, React]
excerpt: |
  One of my clients wanted to start using React in a legacy ASP.NET WebForms application. Here's how we did it.
---
<p>
In a few projects I've taken on lately, the client has stated their desire to modernize their system. In both cases we were dealing with ASP.NET WebForms applications that have been around for years. Both systems worked great and there was no justification for a rewrite, although the architectural parts of the systems left something to be desired. We wanted to minimize the creation of new WebForms code and focus on React.
</p>
<p>
In addition to using WebForms, the client also had dated DevOps practices. No build server, no build scripts – just Right Click => Publish. Although this is something we will focus on later, at this time they had a feature they wanted implemented, so that took precedence. I was worried about the client's ability to consistently handle the webpack part of things since they were generally pretty new to JavaScript in general. I wanted to make it as pain free as possible and not force them into practices with which they were not yet comfortable.
</p>
<p>
I'll summarize the steps taken that are outside the scope of this post: I upgraded to the latest version of .NET Framework, configured WebAPI in the project, added SimpleInjector for DI, added some test projects, created a Cake build script to properly package up the project for deployment including running unit tests, publishing to disk, managing npm, and building the React application. I quickly realized the WebForms Master Page had layers and layers of logic that I didn't want to unravel and apply to an MVC Layout, so new pages are going to be basic ASPX pages that contain the React targets. 
</p>
<p>
My main goal was to enable them to easily adopt React without necessarily having to understand the complexities of npm or webpack. I wanted to automate that stuff as much as possible. Step 1 was to install the <a href="https://marketplace.visualstudio.com/items?itemName=MadsKristensen.NPMTaskRunner">NPM Task Runner</a> extension into Visual Studio. This would give us the ability to bind npm scripts to Visual Studio events. The goal was to bind our webpack dev build to run in watch mode when the project was loaded.
</p>
<script src="https://gist.github.com/mikecole/267ef5c5b04accb41b3b071e2128faa0.js"></script>
<p>
This is the project.json file in its entirety. You can see a few dependencies that I like to use, and a bunch of development dependencies for our build. We'll cover some of those later. Two things to note: the build scripts (one for dev and one for prod) and the <strong>-vs-binding</strong> section which instructs Visual Studio through NPM Task Runner to run the <strong>dev-build</strong> script when the project is loaded.
</p>
<p>
Now we'll look at three webpack files: the dev setup, the prod setup, and the common setup that is shared with dev and prod. I'm using the <a href="https://www.npmjs.com/package/webpack-merge">webpack-merge</a> package to link them together.
</p>
<script src="https://gist.github.com/mikecole/21a7df01287a960bebde9b6f2f7f5be0.js"></script>
<p>
The webpack.common.js file contains the base settings I want to use in both dev and prod. I have two SPAs defined: <strong>login</strong> and <strong>mass-update</strong>. It's a fairly generic webpack script other than the merging. The webpack.dev.js file merges in the webpack.common.js file, sets my development settings, and specifies to watch the content for changes which it will rebuild. The webpack.prod.js file is the production version which parses/compresses the compiles JS and sets an environment flag to <strong>production</strong> which I can test later in the project.
</p>
<p>
I am using both SPAs differently in the site. The mass-update app was a brand new feature, so it gets its own barebones aspx page and runs the show. The login app was an addition to an existing feature and is more of an applet that's bound to an existing page.
</p>
<p>
So far this setup has worked very well. We're able to develop new features using modern technologies in the existing codebase (albeit with some architectural improvements). We didn't have to do a costly rewrite which would have been completely unreasonable, and we're able to feel pretty good about working with React and WebAPI. We expect most of the new development will be done using our new platform, and we'll port over other sections when it makes sense. 
</p>
<p>
Most companies have similar systems sitting around that they feel like they need to re-write to stay modern. If it works, it works, and if it's in no danger of becoming unsupported or incompatible then you don't need to rewrite it. Don't let consulting companies pressure you into a rewrite. I am available for hire as a strategic consultant, an architect, and/or a developer and I can help you make sense of your situation and plot a course of action. You can find my info at <a href="https://cole-consulting.net/">https://cole-consulting.net</a>. Feel free to reach out if you find yourself in a similar situation and feel like you could use my services.
</p>