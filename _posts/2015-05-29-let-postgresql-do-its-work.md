---
ID: 117
post_title: Let PostgreSQL do its work
author: Nicola Racco
post_date: 2015-05-29 07:49:24
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/05/29/let-postgresql-do-its-work/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/120172273864/let-postgresql-do-its-work
tumblr_mikamayhem_id:
  - "120172273864"
---
Placing things in the right place is important. Rails devs tend to forget this, because Rails gives them some really good helpers.

For example, take a uniqueness validation. Doing the validation on the Rails side can produce duplicated records when there are two concurrent requests. So the best way is to handle this type of validation on the DB side.

Same thing with the counter cache column. Defining it on the Rails side may seem the right solution, but this choice will affect performances. Better to handle it via PostgreSQL.

I found a series of posts about these portings by Sean Huber:

- [Porting ActiveRecord soft delete behaviour to PostgreSQL](http://shuber.io/porting-activerecord-soft-delete-behavior-to-postgres/)
- [Porting ActiveRecord counter cache behaviour to PostgreSQL](http://shuber.io/porting-activerecord-counter-cache-behavior-to-postgres/)
- [Porting ActiveRecord validations to PostgreSQL](http://shuber.io/porting-activerecord-validations-to-postgres/)