---
ID: 63
post_title: 'ActiveRecord tips: when to use select in place of pluck'
author: Nicola Racco
post_date: 2015-11-19 11:28:58
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/11/19/activerecord-tips-when-to-use-select-in-place-of/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/133523948449/activerecord-tips-when-to-use-select-in-place-of
tumblr_mikamayhem_id:
  - "133523948449"
---
As you probably know ActiveRecord allows arrays as arguments of conditions. I remember this feature at least since Rails 2...
<!--more-->


So if you write:

```ruby
Product.where tag_code: [1,2,3]
```

It produces:

```sql
SELECT * FROM products WHERE tag_code IN (1,2,3);
```

It's ok, but it's not a wonderful solution if you have a lot of arguments. You can even have an error because you're passing too many arguments.

Then Rails 4 added the `.pluck` method, and since then I'm seeing a lot of code like:

```sql
Product.where tag_code: Tag.pluck(:code)
```

But `pluck` returns an array, so it executes the query right away. In the end you are doing two queries:

```sql
SELECT code FROM tags; # =&gt; [1,2,3]
SELECT * FROM products WHERE tag_code IN (1,2,3);
```

**DON’T DO THIS**. It’s bad. In this case you want to use `.select`:

```ruby
Product.where tag_code: Tag.select(:code)
```

`.select` returns an ActiveRecord relation object, that can be included in other queries. In fact the above code will produce:

```sql
SELECT * FROM products WHERE tag_code IN (SELECT code FROM tags);
```