---
ID: 207
post_title: Initialize Ruby Objects with hashes
author: Gianluca
post_date: 2014-09-19 15:06:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/09/19/initialize-ruby-objects-with-hashes/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/97890966689/initialize-ruby-objects-with-hashes
tumblr_mikamayhem_id:
  - "97890966689"
---
Ruby is a wonderful language (personal opinion! :P) and one of its main features is that it strongly adopts the concepts of Object Oriented Paradigm (OOP).

More than a few times I found it compared to Smalltalk for how strongly it aligns with the over mentioned concepts.

One of the basic good practices on which the OOP funds its roots is to minimize dependencies between objects.

<!--more-->

Actually there are a lot of different types of dependencies that can be classified on the base of their level of abstraction and on the subjects to which they refer.

Regarding methods parameters, for example, there is the order in which they should be supplied when they’re invoked (or more precisely when they’re used in the context of message sending) and their default values.

A straightforward way to deal with this kind of dependencies is to encapsulate the over mentioned parameters into hashes data structures.

A very good explanation about the problem and the relative solution can be found inside the third chapter (i.e. “Use Hashes for Initialization Arguments”) of the wonderful <a href="http://www.poodr.com/" target="_blank">POODR</a> (i.e. “Practical Object-Oriented Design in Ruby” by Sandi Metz).

Trying to dry and clean up a project of mine I came up (with the help of a senior <a href="https://twitter.com/elia" target="_blank">colleague</a> ;) ) with a module whose aim is to minimize the dependencies related to objects initialization.

Here it is:
<pre><code>
module Initializer
  def initialize_with default_attributes
    attr_accessor *default_attributes.keys
    define_method :default_attributes do
      default_attributes
    end
    include InstanceMethods
  end

  module InstanceMethods
    def initialize args = {}
      set_instance_variables(default_attributes.merge(args))
    end

    def set_instance_variables attributes
      attributes.each do |name, value|
        public_send("#{name}=", value)
      end
    end

    def default_attributes
      {}
    end
  end
end
</code></pre>
The module deals in particular with the set up of the initial state of an object. The state is specified through the <code>default_attributes</code>.

These can be overwritten by supplying custom ones inside an hash and for each one of them the module takes care to create the proper <code>attr_accessor</code>.

Together with these, the default attributes gets also stored inside the homonymous property (i.e. <code>default_attributes</code>).

Here’s an example of possible usage:
<pre><code>
class SampleClass

  extend Initializer
  initialize_with ({
    a_property: AClass.new,
    another_property: AnotherClass.new,
    ...
  })
  
end
</code></pre>
If there is the need to perform some particular initialization steps nothing stops you from overriding the classic initialize by simply call <code>super</code> and then the over mentioned steps.

I hope this can be useful to everyone pursuing the concept of dependency minimization!

To the next article! ;)

Cheers!