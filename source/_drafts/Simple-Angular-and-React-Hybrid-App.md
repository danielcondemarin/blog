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

Let's start off with our legacy angular app, which will be a single file, setting up our app, routing and controllers:

 







