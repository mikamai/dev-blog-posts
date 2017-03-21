---
ID: 306
post_title: 'PrestaShop 1.5 &#8230; hooks, hooks everywhere!'
author: Gianluca
post_date: 2013-12-10 11:09:02
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/12/10/prestashop-15-hooks-hooks-everywhere/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/69579529086/prestashop-15-hooks-hooks-everywhere
tumblr_mikamayhem_id:
  - "69579529086"
---
Ever faced the need to put something in a PS theme but you don’t have an hook that you can use?

If the answer is yes don’t despair!

<!--more-->

You can create your custom hooks (in your custom modules) and then call them directly anywhere you want in your theme.

Just like this:
<pre class="brush:php">{hook h='hookName'}</pre>
And here’s the definition of your hook (i.e. a <strong>public</strong> method) in your custom module (i.e. a class extending the abstract ModuleCore class):
<pre class="brush:php">public function hookName($params)
{
  return 'hi there!';
}</pre>
Simple isn’t it? :)

Just remember to register the new custom hook during the module installation:
<pre class="brush:php">public function install()
{
  return parent::install() &amp;&amp; $this-&gt;registerHook('hookName');
}</pre>
<a href="http://nemops.com/adding-hooks-to-prestashop-1-5/">Here</a> you can find a more detailed explanation by Nemo which owns the credits for what I presented here!