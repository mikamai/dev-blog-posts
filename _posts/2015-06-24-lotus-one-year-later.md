---
ID: 107
post_title: 'Lotus &#8211; one year later'
author: Nicola Racco
post_date: 2015-06-24 12:08:58
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/06/24/lotus-one-year-later/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/122331210664/lotus-one-year-later
tumblr_mikamayhem_id:
  - "122331210664"
---
One year ago [@jodosha](https://github.com/jodosha) released [Lotus](http://lotusrb.org) 0.1.0, a small and minimalistic web ruby framework. Even at that early stage it looked like a very interesting product because of its solid MVC approach.
<!--more-->

One year later the freshly released Lotus 0.4.0 is a solid alternative to Rails and it has all the features that you can desire: from migrations to view helpers, from CSRF protection to a good-looking MVC structure.

The best way to see it in action is to follow its [Getting Started](http://lotusrb.org/guides/getting-started/) guide: it requires less than one hour and it will guide you through all the main features.

Among all the features, even considering the latest ones (released with the [0.4.0 version](http://lotusrb.org/blog/2015/06/23/announcing-lotus-040.html)), what I really **love** of this framework is its clean MVC structure.

In lotus the controller is a module and the action is a class. And **it just makes sense!** If you think about it, a controller is only a group of actions related to the same resource.

View and template are two separate entities in lotus. The view is a class while the template is... well, a template. This means that the action handles only the business logic, while the view handles the presentation logic, and the template presents data with the proper response format.

Lotus has many other good key features. Let’s consider the models structure. The behavior that a model expresses is the Entity object, while the persistence layer is handled by the Repository. The practical implications are that changing the persistence layer under the model, or sharing the same model between different projects becomes very easy.

Still, I’ve had no occasion to use Lotus in a production-ready app, but with the 0.4.0 release I will definitely do it.