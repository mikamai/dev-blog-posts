---
ID: 182
post_title: Ruby, PHP and the Array Map method
author: Gianluca
post_date: 2014-11-24 10:27:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/11/24/ruby-php-and-the-array-map-method/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/103452743169/ruby-php-and-the-array-map-method
tumblr_mikamayhem_id:
  - "103452743169"
---
Due to my work and my academia commitments I find myself very often switching from Ruby to PHP and back again. This is making me more and more accustomed to the substantial differences between these two languages.

Among these differences the handling of collections is perhaps one of the biggest I found till now.

For example in Ruby there is a module (i.e. an object) called Enumerable that exposes a lot of useful helpers and utilities to work with collections (i.e. objects).

One of these utilities is <a href="http://ruby-doc.org/core-2.1.5/Enumerable.html#method-i-map" target="_blank">map</a>. The concepts underlined are pretty much the same across languages and are summarized <a href="http://en.wikipedia.org/wiki/Map_(higher-order_function)" target="_blank">here</a>.

While working on a collection of objects, in particular a <a href="http://www.rubydoc.info/github/sparklemotion/nokogiri/Nokogiri/XML/NodeSet" target="_blank">Nokogiri::XML::NodeSet</a>, I recently had the need to extend the map behavior to get a collection of non duplicated <a href="http://www.rubydoc.info/github/sparklemotion/nokogiri/Nokogiri/XML/Node" target="_blank">Nokogiri::XML::Node</a> elements.

<!--more-->

This is actually a pretty straightforward need that can be satisfied by calling the Array <a href="http://www.ruby-doc.org/core-2.1.5/Array.html#method-i-uniq" target="_blank">uniq</a> method and providing it with a block containing the logic regarding the uniqueness of the elements.

So I wrote the following little helper Module that I included inside the Nokogiri::XML::NodeSet class:
<pre><code>
module UniqueMapper
    def map_unique unique_id, &amp;block
        self.map(&amp;block).compact.uniq{ |x| x.send(unique_id) }
    end
end
</code></pre>
I simply defined a brand new map method that takes a parameter and a block as arguments. The first is used inside the uniq block to “describe” the uniqueness criteria while the latter is passed directly to the standard map method. The <a href="http://www.ruby-doc.org/core-2.1.5/Array.html#method-i-compact" target="_blank">compact</a> method is used in between to remove nil values from the array.

Thanks to Ruby “awesomeness” this was pretty easy to accomplish.

Now, luckily for me I don’t have to satisfy a similar need in PHP yet.

What I needed instead was to pass to the callback of the PHP map function (i.e. <a href="http://php.net/manual/en/function.array-map.php" target="_blank">array_map</a>) some additional arguments.

While doing this I discovered something awkward. You need to prepare an array for every additional argument composed by the argument value with a length equal to the length of the array that should be “mapped”.

Here’s an example:
<pre><code>
array_map(
'callback', 
$array_to_map, 
array_fill(0, count($array_to_map), $first_additional_argument),
array_fill(0, count($array_to_map), $second_additional_argument),
...
);
</code>
</pre>
The first argument (i.e. “callback”) represents the name of the function used to map the second argument (i.e. $array_to_map). The others arguments are the ones taken by the “callback” function.

This solution is everything but pretty and probably (I hope!) there’s a more elegant one out there.

With this article I wanted to show my personal (and limited) experience with the map utility in Ruby and PHP and how these languages differs profoundly from each other.

Cheers!