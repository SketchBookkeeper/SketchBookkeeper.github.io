---
layout: post
title:  "How to setup BroswerSync in Lando"
date:   2018-11-27 20:00:00 -0600
---

Moving to Docker(via [Lando](https://docs.devwithlando.io/)) has helped remove a lot of the issues I've faced working with different local environments in a team. Most of us don't want to lose access to [BrowserSync](https://browsersync.io/) and hot reloading when moving over to a Docker environment.

## What is Lando?
I realize this post is fairly specific to an issue, but let take a minute to talk about why I think Lando is so great. Lando is a Console Application that is really just a helper for creating and maintaining Docker images. The idea is that anyone can pickup a project without needing any additional local setup or tooling.

## Exposing the port
In this example I'm working with a WordPress recipe, but the solution is helpful for any setup. In Lando we define the Docker image using a `.lando.yml` file.
``` yml
name: my-project
recipe: wordpress
services:
  node-cli:
    type: node:8.9
  node:
    type: node:8.9
    overrides:
      services:
        ports: [3003:3003]
tooling:
  npm:
    service: node
  node:
    service: node
```

The important part here is the the services. When you create a Lando project, you can select what services will be needed to serve your application. If you use a recipe, it will pre-configure some things for you. The WordPress recipe will include things you need to get started such as PHP and MySQL. Additional services can be added in the `.lando.yml`. As you can see I've added the services and tooling needed to have a specif version of node in my build process.

Now the tricky part is that because Docker is basically another computer running on your machine. The Docker image will not have access to all the ports unless we tell it to use one. Under `node` I've added the `ports: [3003:3003]` so that this Docker's node server will be available at port 3003.

Now in your build tool of choice for BrowserSync, configure it to use the port. In this case 3003 for me. I'm using Webpack and here is what that looks like. The nice thing is that the BrowserSync API is consistent across build tools.
``` js
const BrowserSyncPlugin = require('browser-sync-webpack-plugin');
// ... webpack stuff ...
plugins: [
  new BrowserSyncPlugin(
    {
      proxy: 'http://myproject.lndo.site',
      host: 'myproject.lndo.site',
      port: 3003,
      files: ['**/*.js', '**/*.css', '**/*.php'],
      injectChanges: true,
    },
  ),
]
```

## Port Madness
Notice my project's url does not have a port appended to the end. If ports 80 and/or 443 are being used by another service, Lando will open up another port for your application and append that port to the end of the url. You'll get something like this `http://myproject.lndo.site:8000`. I was __not__ able to get BrowserSync to work if this was my url. If you are like most PHP developers, you have something else that wants to use ports 80 and 443 (MAMP, Laravel Valet, etc.).

You can check if the ports are free with these commands.
``` bash
sudo lsof -n -i :80 | grep LISTEN
sudo lsof -n -i :443 | grep LISTEN
```

In my case it was Laravel Valet. To remedy this, I make sure Valet is stopped and the ports are unused when I run `lando start` the first time in a project. I even go as far as not letting the Docker app start on startup. I do have to switch between stopping Docker and Valet in order to work on different projects, but it's not too big of a pain and totally worth to have live reloading.

## Links
[GitHub Issue](https://github.com/lando/lando/issues/144)

[Lando Proxy Guide](https://docs.devwithlando.io/config/proxy.html)
