---
ID: 175
post_title: The Volt framework
author: Nicola Racco
post_date: 2014-12-12 11:40:01
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/12/12/the-volt-framework/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/105000921849/the-volt-framework
tumblr_mikamayhem_id:
  - "105000921849"
---
This morning I stumbled upon [Volt](http://voltframework.com) and it impressed me.

It’s like Meteor but on Ruby, its opinionated directory structure is clearly inspired by Rails while its template system is very similar to the one used by Angular (and I’m referring to Angular 1.x, not that mess called 2.0 they are working to).
<!--more-->

And of course, as you might guess, it comes with all the syntactic sugar, rich api set and code beauty that ruby developers are so used to.

To give you a more complete overview about Volt, here’s the author's response to the question "_What's the problem that this framework is trying to solve?_" made on [HN](https://news.ycombinator.com/item?id=8316837).

> _The goal here isn't to keep people from learning JS. I've been doing JS development since long before I found ruby. Just some thoughts on it I had been working on for a blog post:_

> _In web development today, JavaScript gets to be the default language by virtue of being in the browser. JavaScript is a very good language, but it has a lot of warts. (See [http://wtfjs.com/](http://wtfjs.com/) for some great examples) Some of these can introduce bugs, others are just difficult to deal with. JavaScript was rushed to market quickly and standardized very quickly. Ruby was used by a small community for years while most of the kinks were worked out. Ruby also has some great concepts such as uniform access, mixin's, duck typing, and blocks to name a few. While many of these features can be implemented in JavaScript in userland, few are standardardized and the solutions are seldom eloquent._

> _Uniform access and duck typing provide us with the ability to make reactive objects that have the exact same interface as a normal object. This is a big win, nothing new to learn to do reactive programming._

Now, I will not write a tutorial because [Volt documentation already contains a good tutorial](http://voltframework.com/docs). And there is also a complete [presentation on slideshare](http://www.slideshare.net/ryanstout/isomorphic-app-development-with-ruby-and-volt-rubyconf2014) about it. So I want only to highlight some cool features.

### Websockets

Instead of syncing data between client and server via HTTP, Volt uses a persistent connection between them. When data is updated on one client, it is updated in the database and any other listening clients.

In addition the server notifies the clients whenever the code change, forcing a reload, like meteor.

These things may seem obvious for a Meteor competitor, but keep in mind that Volt is a one-man work and it's relatively new.

### Ruby

Obvious, but it's better to highlight this. Thanks to Opal you write ruby code for both server and client in a way that is a joy for a ruby dev.

### File Weight

![Volt File Size](https://dev.mikamai.com/wp-content/uploads/2014/12/volt_file_size.jpg) {.aligncenter}

No words

### Components

As soon as you'll start to read the tutorial you'll notice that paths have a weird structure. For example the first file you’ll edit is `app/main/views/main/todos.html`

What's that `main` inside `/app`? Do I need it **really**?

It will become clear when you'll read about [Components](http://docs.voltframework.com/en/docs/components.html). Inside `/app` you place components. And now it's clear: `main` is the main component.

This way you'll naturally tend to treat your app as a set of components, each one at the same level, and not as a giant monolithic structure.

### Conclusions

So... I'm **really** impressed. It's a one-man work and it's a relatively new project; I'm not sure if it could be a great idea to use it in production **right now**. But I'll surely look at it deeper and I'll give it a chance.