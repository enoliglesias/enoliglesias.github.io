---
layout: post
title: Storing Compressed JSON Objects in Local Storage With Lz-string
modified:
share: true
categories: blog 
excerpt: A simple tiny way to store JSON compressed objects in the local storage
comments: true
tags: [json, javascript, local storage, compress, lz string]
date: 2016-02-08T19:26:59+01:00
---


In my first post I would like to talk about a simple way to store JSON compressed objects in local storage. This is a very tiny, but useful feature.

The local storage keys are limited to 5 Mb. With this feature, we'll compress a **2 Mb JSON into a 0.22 Mb**. Sounds good, huh?

Let's start! The first thing we need to ease this way of storage is to extend the localStorage by adding a couple of functions. In order to save and restore the JSON objects. Take a look:

~~~ javascript
Storage.prototype.setObj = function(key, obj) {
  return this.setItem(key, JSON.stringify(obj));
};
Storage.prototype.getObj = function(key) {
  return JSON.parse(this.getItem(key));
};
~~~
Now, using `setObj` and `getObj` instead `setItem` and `getItem` we're able to do things like that:

~~~ javascript
localStorage.setObj("test", {a: "1", b: "2"})
var test = localStorage.getObj("test")
test.a 
> "1"
test.b
> "2"
~~~

The next step is to add the [LZ-string](http://pieroxy.net/blog/pages/lz-string/index.html){:target="_blank"} library. In order to compress the data. Download the library and set the corresponding script tag in your `<head>` 

~~~ html
<script language="javascript" src="route/to/lz-string.js"></script>
~~~

Come on! it's almost done. The last step is to use the LZ string to compress the objects. It's as simple as adding in our custom functions the following code:

~~~ javascript
Storage.prototype.setObj = function(key, obj) {
  return this.setItem(key, LZString.compress(JSON.stringify(obj)));
};
Storage.prototype.getObj = function(key) {
  return JSON.parse(LZString.decompress(this.getItem(key)));
};
~~~

And that's all! I hope you find it useful. You can find me on twitter [@enoliglesias](https://twitter.com/enoliglesias){:target="_blank"} for whatever you want to ask.

I want to thank [@putuko](https://twitter.com/putuko){:target="_blank"} for discover me the LZ string :)


