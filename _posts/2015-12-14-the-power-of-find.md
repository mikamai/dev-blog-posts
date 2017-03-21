---
ID: 59
post_title: The power of find
author: Diomede Tripicchio
post_date: 2015-12-14 14:38:30
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/12/14/the-power-of-find/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/135186209964/the-power-of-find
tumblr_mikamayhem_id:
  - "135186209964"
---
Today I scratch the surface of a “little” useful command: `find`

First of all, some context. I have a Wordpress site that manages all media with the awesome plugin <a href="https://it.wordpress.org/plugins/amazon-s3-and-cloudfront/">WP Offload S3</a>, but for some reason I need to disable this plugin and return back to the default Wordpress media management.

<!--more-->

So I downloaded the *uploads* folder from the S3 bucket and then uploaded it on the server that host the site.

“Unfortunately” the *Object Versioning* options is enabled in the plugin (WP Offload S3).

Now on the S3 bucket I have a file structure like this:

```bash
|—uploads
  |—2015
    |—01
      |—12011023
        |—example.jpg
```

But Wordpress expects:

```
|—uploads
  |—2015
    |—01
      |—example.jpg
```

I have to move all the files up of one dir. Something like 20.000 files divided across hundreds of dirs. How I can do this without wasting a lot of time?

After some googling here is the answer… `find`!!

From the *uploads* dir:

`find . -mindepth 4 -maxdepth 4 -type f -execdir mv {} .. ;`

With this command I get all children files at the fourth level of the tree (`-mindepth 4 -maxdepth 4 -type f`) and then I move the file to the parent dir (`-execdir mv {} .. ;`)

One command, upload (a little wait), all done!

Maybe not so special at the end, but very useful in a case like this!