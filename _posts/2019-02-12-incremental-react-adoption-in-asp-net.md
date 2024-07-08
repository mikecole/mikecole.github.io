---
layout: post
title: Incremental React Adoption in ASP.NET
date: 2019-02-12
tags: react asp.net
meta-description: One of my clients wanted to start using React in a legacy ASP.NET WebForms application. Here's how we made it work.
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
```json
{
	"name": "the-project",
	"version": "1.0.0",
	"private": true,
	"dependencies": {
		"axios": "^0.18.0",
		"babel-polyfill": "^6.26.0",
		"evergreen-ui": "^4.6.0",
		"react": "^16.4.1",
		"react-bootstrap": "^0.32.4",
		"react-dom": "^16.4.1"
	},
	"optionalDependencies": {
		"fsevents": "*"
	},
	"devDependencies": {
		"babel-core": "^6.26.3",
		"babel-loader": "^7.1.5",
		"babel-preset-env": "^1.7.0",
		"babel-preset-react": "^6.24.1",
		"babel-preset-stage-2": "^6.24.1",
		"clean-webpack-plugin": "^0.1.19",
		"css-loader": "^1.0.0",
		"node-sass": "^4.9.2",
		"sass-loader": "^7.0.3",
		"style-loader": "^0.21.0",
		"uglifyjs-webpack-plugin": "^1.2.7",
		"webpack": "^4.16.1",
		"webpack-cli": "^3.1.0",
		"webpack-dev-server": "^3.1.5",
		"webpack-merge": "^4.1.3"
	},
	"scripts": {
		"dev-build": "webpack -d --config ./webpack.dev.js",
		"prod-build": "webpack --config ./webpack.prod.js"
	},
	"babel": {
		"presets": [
			"env",
			"react",
			"stage-2"
		]
	},
	"-vs-binding": {
		"ProjectOpened": [
			"dev-build"
		]
	}
}
```
<p>
This is the project.json file in its entirety. You can see a few dependencies that I like to use, and a bunch of development dependencies for our build. We'll cover some of those later. Two things to note: the build scripts (one for dev and one for prod) and the <strong>-vs-binding</strong> section which instructs Visual Studio through NPM Task Runner to run the <strong>dev-build</strong> script when the project is loaded.
</p>
<p>
Now we'll look at three webpack files: the dev setup, the prod setup, and the common setup that is shared with dev and prod. I'm using the <a href="https://www.npmjs.com/package/webpack-merge">webpack-merge</a> package to link them together.
</p>
```javascript
const path = require('path');
const CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = {
    entry: {
        login: './Scripts/apps/login.jsx',
        'mass-update': './Scripts/apps/mass-update/app.jsx',
    },
    module: {
        rules: [
            {
                test: /\.(js|jsx)$/,
                exclude: /node_modules/,
                use: ['babel-loader'],
            },
            {
                test:/\.(s*)css$/,
                use:['style-loader','css-loader', 'sass-loader'],
            },
        ]
    },
    resolve: {
        extensions: ['*', '.js', '.jsx'],
    },
    output: {
        path: path.join(__dirname, 'scripts', 'dist'),
        publicPath: '/',
        filename: '[name].bundle.js',
    },
    plugins: [
        new CleanWebpackPlugin([path.join(__dirname, 'scripts', 'dist')]),
    ]
};
```

```javascript
const merge = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common,
{
    mode: 'development',
    devtool: 'inline-source-map',
    devServer: {
        contentBase: './Scripts/dist'
    },
    watch: true,
    watchOptions: {
        aggregateTimeout: 300,
        poll: 1000,
        ignored: './node_modules/'
    }
});
```

```javascript
const webpack = require('webpack');
const merge = require('webpack-merge');
const common = require('./webpack.common.js');
const UglifyJSPlugin = require('uglifyjs-webpack-plugin');

module.exports = merge(common, {
    mode: 'production',
    devtool: 'source-map',
    plugins: [
        new UglifyJSPlugin({
            sourceMap: true
        }),
        new webpack.DefinePlugin({
            'process.env.NODE_ENV': JSON.stringify('production')
        })
    ]
});
```
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