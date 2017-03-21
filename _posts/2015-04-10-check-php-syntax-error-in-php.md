---
ID: 137
post_title: Check PHP syntax error in PHP.
author: Gianluca
post_date: 2015-04-10 15:04:18
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/04/10/check-php-syntax-error-in-php/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/116031296309/check-php-syntax-error-in-php
tumblr_mikamayhem_id:
  - "116031296309"
---
This sounds much like “Inception” right?

Well a week ago I had the need to implement a PHP syntax check in PHP, to be more precise inside a PrestaShop (PS) module.

After a little bit of “googling” I stumbled upon a StackOverflow comment showing the solution I needed. Actually I missed the original link so I’m posting this pearl as a “bump” just to help spread the solution. I don’t own any credits ;)

<!--more-->

Here’s it is the magic statement:
<pre><code>exec(sprintf('echo %s | php -l', escapeshellarg($string)), $output, $exit);</code></pre>
All this instruction does is to invoke the PHP “lint“ functionality and collect its results.

Just for the record here there is the method of the over mentioned module:
<pre><code>private function checkPhpSyntaxErrors($string)
{
   exec(sprintf('echo %s | php -l', escapeshellarg($string)), $output, $exit);
    if ($exit !== 0) {
      $this-&gt;_errors[] = $this-&gt;l("Syntax error: {$this-&gt;outputVarDump($output)}");
      return false;
    }
    return true;
 }</code></pre>
I hope this can be helpful to everyone facing the same problem as mine.

To the next time!

Cheers!