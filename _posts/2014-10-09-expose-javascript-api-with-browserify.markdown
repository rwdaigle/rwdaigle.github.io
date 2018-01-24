---
title: "Exposing a Javascript API in a Web Page with Browserify"
layout: post
date: 2014-10-09 13:07:56 -0500
author: ryan
tags:
  - browserify
category: blog
blog: true
---

When using Browserify to build and resolve your javascript library dependencies, it's very easy to get a resulting `bundle.js` file that you can include in a browser with a simple script tag.

```html
<script src="/assets/bundle.js"></script>
```

Including the bundle in this way will execute the code in your entrypoint (often `main.js`) which is where most online tutorials end and which might be all you need. However, how do you create a bundle that will expose an API to the including page? So you can do something like:

```html
<script src="/assets/bundle.js"></script>
<script>
var lib = new MyLibrary();
</script>
```

You need to specify the `standalone` option to the browserify command (or API), which will export the API you expose in your entrypoint file in the given namespace. Here's an example.

Given a library file where your main library functionality is defined:

```js
var _ = require('underscore');

module.exports = MyLibrary;

function MyLibrary() {
  this.aSetting = true;
};

MyLibrary.prototype.doWork = function() {
  console.log(this.aSetting);
}
```

And your browserify entrypoint which exposes your library API:

```js
module.exports = require('./lib')
```

At this point, if you were to build a bundle with browserify and include it in a web page, you wouldn't be able to access `MyLibrary`. Since browserify makes sure everything is local scoped, and the web page doesn't know how to deal with your top level export, it's effectively hidden. The solution is to tell browserify to expose your exports with the `standalone` option.

As a command it looks like this:

```bash
$ browserify main.js --standalone MyLibrary > bundle.js
```

And if you're using Gulp or similar the API options looks like:

```js
browserify('./main.js', {
  standalone: 'MyLibrary'
})
```

With this resulting bundle you can now reference `MyLibrary` from the including html page.

```html
<script src="/assets/bundle.js"></script>
<script>
var lib = new MyLibrary();
lib.doWork();
</script>
```
