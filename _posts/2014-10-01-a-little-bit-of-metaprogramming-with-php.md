---
ID: 202
post_title: A little bit of metaprogramming with PHP
author: Gianluca
post_date: 2014-10-01 10:20:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/10/01/a-little-bit-of-metaprogramming-with-php/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/98879497744/a-little-bit-of-metaprogramming-with-php
tumblr_mikamayhem_id:
  - "98879497744"
---
I should admit it. After writing more and more Ruby code and get into its beautiful Object Model it feels rather strange to go back to PHP.

What I miss mostly is the syntactic sugar and the less verbose syntax.

You don’t have to put dollar signs before variables names, semicolons at the end of statements and most of the times you can also omit method parentheses (i.e. when there are no ambiguities).

But beware! Omitted parentheses could lead to unexpected behaviors and bugs that could be hard to find! So you’d better indulge on this sugar only if you know what you’re doing.

<!--more-->

One thing you can also omit (and I always do) is the return keyword as methods in Ruby always return the last evaluated expression.

Beside these things, what I recently missed most is how easily you can get your hands dirty with metaprogramming.

At least this is what I miss most :P

Don’t worry! I know:

<img src="http://68.media.tumblr.com/8cec497acbbbdd6a5f1fefb35e2c2b0e/tumblr_inline_ncb8hse4pa1ss63kf.jpg" alt="image" />

Metaprogramming gives you power. A lot of power!

So (second) beware! If you are working on big projects with many colleagues you <strong>simply</strong> (<strong>absolutely</strong>) <strong>can’t </strong>expect to “metaprogram" like a cowboy! And if you do it without tests, then you’re asking for double trouble!

Anyway this post isn’t actually about Ruby! :P

It is indeed about PHP!

PHP is the first language I used at work and is still the one I use on a daily basis.

Recently I felt the need to "dry out” my code by reducing the duplication of many methods that shared some common features. As an example, I’m using some code that integrates a simple log infrastructure inside an e-commerce framework (PrestaShop).

So, in the context of a module (<a href="http://doc.prestashop.com/pages/viewpage.action?pageId=23626839" target="_blank">here’s</a> the documentation that explains what a module is about) I came up with the following solution:
<pre><code>
private function doAndLogErrors()
{
    $args = func_get_args();
    if ($return_value = call_user_func_array(
        array($this, $args[0]),
        array_slice($args, 2))) {
        return $return_value;
    }
    $this-&gt;_errors[] = $args[1];
    return $return_value;
}</code></pre>
&nbsp;

Here’s an example of its usage:
<pre><code>
$this-&gt;doAndLogErrors(
    'method_name',
    'Custom error message.',
    $first_parameter,
    $second_parameter,
    ...
)
</code></pre>
Now I can actually call every other method inside the module by passing as many parameters as I want, and I can log at the same time a custom error message if it fails.

The only requisite is that the called method should return a boolean value or something that actually tells if it succeed or not.

A possible candidate for this could be a method that validates a particular input case or that performs some persistence activities on simple data.

I’m sure that this is not a very clean solution for the problem I faced and that it could be solved in a more elegant way but my goal with this post was only  to demonstrate (as the title states) “A little bit of metaprogramming with PHP”.

I hope you enjoyed!

Cheers!