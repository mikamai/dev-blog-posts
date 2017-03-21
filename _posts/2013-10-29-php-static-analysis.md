---
ID: 326
post_title: PHP Static Analysis
author: Nicola Racco
post_date: 2013-10-29 11:04:05
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/10/29/php-static-analysis/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/65423870501/php-static-analysis
tumblr_mikamayhem_id:
  - "65423870501"
---
If you want to statically analyze the source code you surely need to first parse it and obtain a form of Abstract Syntax Tree (AST). [PHP-Parser](https://github.com/nikic/PHP-Parser) lets you do that!

You just need to pass the parser the code, then it will return a code representation that you can use to implement several types of analysis.

We're relying on it to create an in-memory model of an entire source base using Redis to do, afterwards, some type inference analysis. It works pretty well!