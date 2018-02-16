---
title: Simple Angular and React Hybrid App
tags:
---

If you have worked with legacy apps, you know doing a big bang migration of one framework to another is not easy peasy!

A bit of history:

Once upon a time I had worked on a project to migrate our legacy Angular app to React.js. Basically we had 2 separate applications, one for our mobile website, built using angular, angular-router. And our desktop app, on React. After duplicating our estimates for each piece of work, having to do the same feature on both Angular and React, we decided to stick with one of them and make the whole site responsive. Our choice was to responsify our Desktop app built in React.
The idea is to somehow, get angular and React playing "nicely" together, so that you can start stripping off small sections from your legacy app and move it to React, rather than trying to migrate everything in one go, which is normally the recipe for disaster and an unhappy business.

In this post I will give you a starting point for you to do this, using the power of webpack and ngReact.

Show me the code!

Let's start with our legacy angular app. Suppose this is the html page you're serving:

{% gist b1eb5e1d4916104a58200aa268224b81 %}

Note we have the script tags for `angular` and `angular-router` and our app `angular-app`. 
Let's have a look now at the angular app:

{% gist b5c59b2294e6314935f596234e6e9f67 %}

There is nothing particularly interesting so far. Just a simple app which will render `/legacy/templates/home.html` for the route `/#home`. This is the home template:

{% gist 63c3a76d4837f9ba3ad3429865c96f3e %}

If we serve what we have so far :

![index.html](https://res.cloudinary.com/danielcondemarin/image/upload/v1518813469/angular-react-hybrid-indexhtml_n6f3vm.png)

Now, let's suppose we want to render a React component in the home page template. For that I'm going to use `ngReact`, which uses `react` and `react-dom` to render your React components. 
Before we configure `ngReact` in our angular app, let's say we have the following React component which we want to render:

{% gist 7cd09185724dc418b824809c1955d719 %}

Now, we will bundle our react app using webpack. The most IMPORTANT thing to note here is we need to EXPOSE in the window the React component, so that ngReact can actually see it and use it.

This is the webpack.config:

{% gist eed36be54b42e6799f5e67989d830749 %}

First, on our `entry`, note we are **explicitly** requiring `react-dom` and `ngReact`. If we don't do this, webpack won't include it in the bundle as these libraries are not used in the React app. Note we don't do `ReactDOM.render ...` anywhere.
The second important part is the `output` where we have asked webpack: Can you expose in the **root scope**(that's what `libraryTarget: "var"` means) a variable called `ReactEntry`?  This will allow you to access `DummyComponent` from the window: `window.ReactEntry.DummyComponent`.
Lastly, we use the plugin `expose-loader` to have the library ngReact on the window as well. Alternatively you could have ngReact on a script tag separately from your webpack bundle, but I prefer to have everything in one webpack config.

Now, let's configure ngReact. First, we need to add an extra dependency called 'react' to the angular module:

`var myApp = angular.module('myApp', ['ngRoute', 'react']);`

Then set the component for ngReact to use like this:

`myApp.value('DummyComponent', window.ReactEntry.DummyComponent);`

With that done we can render DummyComponent in home.html using a custom directive provided by ngReact:

<react-component name="DummyComponent"></react-component>

And this is what the final product looks like:

![index.html](https://res.cloudinary.com/danielcondemarin/image/upload/v1518816097/react-hybrid-final_bu4uoq.png)

There is a repo on github with all the code, https://github.com/danielcondemarin/angular-react-hybrid.

Feel free to comment below if you have any questions :)

















