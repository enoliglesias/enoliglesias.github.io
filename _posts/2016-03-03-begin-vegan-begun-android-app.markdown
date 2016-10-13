---
layout: post
title: Begin Vegan Begun Android App (Cordova)
modified:
share: true
categories: blog 
excerpt: My very first try inside mobile app world :)
comments: true
tags: [begin, vegan, begun, android, app, mobile, cordova, hybrid]
date: 2016-03-03T19:26:59+01:00
---


Today, I would like to announce my first Android mobile application.

[Begin Vegan Begun vegan recipes](https://play.google.com/store/apps/details?id=com.enoliglesias.bvb){:target="_blank"}

This is the app for the vegan recipes blog [Begin Vegan Begun](http://beginveganbegun.es/){:target="_blank"}. And it's developed with [Apache Cordova](https://cordova.apache.org/){:target="_blank"}. It is a CLI wich allow you to build a HTML+CSS+JS project into a native mobile app for Android, iOS and some more platforms.

To make a simple project with Cordova is pretty easy. For this project I use:

* HTML
* CSS
* JS
* A Wordpress API. In order to get the blog data

You can develop the app like if you were developing a web app. Getting the API data via JS calls. Using all the HTML you want, CSS, etc. And once you build the project for some platform, those calls will work perfectly. And the HTML + CSS will show perfectly too. Cordova do the work of put them in a web view.

One important thing to take in account, it's that the responsive design is very important (in all apps is important, but in this case, a little more ;)). Because each mobile has a different screen size. And we want our application look awesome in all phones.

You can take a look at my [wpreader github repo](https://github.com/enoliglesias/wpreader){:target="_blank"} and see what I'm talking about.

I would like to talk about _pros_ and _cons_ of Cordova. Always IMHO. And based in my experience by developing this app.

# Main pros

* It's very fast to develop an application
* It's very simple if you come from web app development. You don't need to learn extra languages
* There are a lot of plugins and a big Cordova comunity
* You can use one development for all the platforms. Instead one per platform
* _Feel free to let me know your pros_


# Main cons
* It's hard (at least for me) to use native functionality like push notifications
* I couldn't add cloud storage via JS plugins
* There are some HTML+CSS that shows correctly in one platform but not in the other ones. So you have to do some dirty tricks }:)
* The same with brand/model of phone
* _Feel free to let me know your cons_

Please, take in account that I always talk about small-middle project. I don't know if for a huge project, Cordova it's a good or bad choice. But, you can tell me if you know it.

And that's all! I hope you enjoy reading this. The next step is to publish the iOS one. 

All comments are more than welcome. And feel free to tell me whatever you want. You can find me on twitter [@enoliglesias](https://twitter.com/enoliglesias){:target="_blank"} :)


