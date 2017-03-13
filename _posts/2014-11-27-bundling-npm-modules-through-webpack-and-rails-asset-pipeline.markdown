---
layout: post
title: Bundling NPM modules through Webpack and Rails' Asset Pipeline
---

This post was inspired by an [excellent post over at Rails on Maui](http://forum.railsonmaui.com/t/fast-rich-client-rails-development-with-webpack-and-the-es6-transpiler/82){:.link.dim.blue} detailing how Webpack and the
Rails Asset Pipeline can work together with ES6 modules. To keep it
simple, I decided to try and work with the bare minimum Rails and Webpack setup skip
some of the additional work that Justin has described around implementing the Webpack dev server and using ES6 modules.

Over the past few years using Gemified assets has and become
the de facto solution for including JavaScript and CSS modules in a
Rails application. This solution is simple and the usage is elegant from the
perspective of the Rails developer, all you need to do is specify the
gem as a dependency, install with Bundler, require the asset name
through the appropriate Sprockets manifest and you're set to go!

But it's not all that simple. First we have to assume that the JS or CSS module/component you want
to use has been gemified and is up to date with the latest version of
the module it packages. The maintainers gemified libraries are
required to manually publish new versions every time the corresponding
JS or CSS modules are updated and Github becomes littered with numerous
repositories all prefixed with "rails-".

As Justin described in his post, using a package manager
that's designed to manage JavaScript and/or SCSS assets (like
NPM or Bower) means that we aren't creating a sub-ecosystem of Rails
specific components.

Right, so onto how I've implemented this. I'm not going to go into a lot
of detail as far as how Webpack works and the motiviations for using it,
but Pete Hunt has an excellent [how-to](https://github.com/petehunt/webpack-howto){:.link.dim.blue} that is worth reading. To get started, Webpack needs to be
installed globally in your environment:

{% highlight sh %}
npm install webpack -g
{% endhighlight %}

Next add a file called `package.json`, you can do this manually or by
running `npm init` and answering the questions it offers. Here is my
demo `package.json` for reference:

{% highlight json %}
{
  "name": "react-demo",
  "version": "0.0.0",
  "description": "Simple demo built with React",
  "main": "index.js",
  "directories": {
    "test": "test"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "James McCann <jmccnz@gmail.com>",
  "license": "ISC",
  "dependencies": { },
  "private": true,
  "devDependencies": { }
}
{% endhighlight %}

At the moment we have no dependencies in the package, but this will
change shortly. I'm going to build this demo out using React, so we'll
need to install that via NPM. It would be nice to use JSX as well, so
I'll install the Webpack [jsx-loader](https://www.npmjs.org/package/jsx-loader){:.link.dim.blue} as a development dependency:

{% highlight sh %}
npm install react --save
npm install jsx-loader --save-dev
{% endhighlight %}

And NPM will create a `node_modules` directory within our Rails
project, now we are onto something! Following the directory
structure suggested by Justin in his post, we need to make a `webpack`
directory within our project, and create a place to store our JS code:

{% highlight sh %}
mkdir -p webpack/assets/javascripts
{% endhighlight %}

Inside the webpack directory we'll need to add a config file that tells
Webpack how to bundle all of our modules. I won't go into too much
detail on the format of this file as others on the web can do a far
better job than I can, but the key takeaway here is that we are telling
Webpack to do it's thing and then output the bundled JS to
`app/assets/javascripts/bundle.js` so we can include it into our JS
manifest via Sprockets. Here is my config file:

{% highlight js %}
var path = require('path');

module.exports = {
  context: __dirname,
  entry: './assets/javascripts/app.jsx',
  output: {
    filename: 'bundle.js',
    path: '../app/assets/javascripts'
  },
  resolve: {
    root: [path.join(__dirname, "assets/javascripts")],
    extensions: ["", ".js", ".jsx"]
  },
  module: {
    loaders: [
      { test: /\.jsx$/, loader: 'jsx-loader' },
    ]
  }
}
{% endhighlight %}

In the new javascripts directory add a file called app.jsx (I've used
the React "Hello World" example):

{% highlight js %}
/** @jsx React.DOM */

var React = require('react');

document.addEventListener('DOMContentLoaded', function() {
  React.render(
    <h1>Hello, world!</h1>,
    document.body
  );
});
{% endhighlight %}

Running `webpack -w` inside the `webpack/` directory will watch our JS
files for changes and recompile the `bundle.js` file that Rails is
looking for.

{% highlight sh %}
➜  webpack  webpack -w
Hash: 8a13e1b363201b21091b
Version: webpack 1.4.13
Time: 746ms
    Asset    Size  Chunks             Chunk Names
bundle.js  589019       0  [emitted]  main
+ 148 hidden modules
{% endhighlight %}

The final piece of this puzzle is to tell Rails that we want to serve
the bundled JavaScript, this should look pretty familiar:

{% highlight js %}
//= require bundle.js
{% endhighlight %}

And now, running `rails s` should give us what we are looking for:

![Hello World](/images/2014/11/28/react-rails.png)

I hope this was a useful, albeit brief, introduction to how we can start
to utilise the NPM/Bower ecosystems in Rails using Webpack. I think that Webpack offers the potential for a better way to manage JS
modules and their dependencies than Gemified assets, and allows us to
leverage the success of NPM and Bower for managing web packages. Not
only does this lead to less Rails specific asset packages out in the
wild, it also supports an ecosystem for publishing re-usable
components that can be used anywhere regardless of the environment.

