---
ID: 167
post_title: Ruby and self
author: Andrea Longhi
post_date: 2015-01-19 11:05:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/01/19/ruby-and-self/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/108536863609/ruby-and-self
tumblr_mikamayhem_id:
  - "108536863609"
---
<h3>What&rsquo;s self?</h3>
<p>Ruby newcomers usually are confused about the use of <code>self</code>, whether it&rsquo;s a completely new concept for them or not.</p>
<p>First, let&rsquo;s define self:</p>
<blockquote>
<div>self is the current object</div>
</blockquote>
<p>The definition may seem a little too dry, but that&rsquo;s exactly what self is. In your code, at any time, there&rsquo;s always a <code>self</code> object, and its value depends on the code context.</p>
<h3>IRB and self</h3>
<p>When practicing with ruby, <a href="http://en.wikipedia.org/wiki/Interactive_Ruby_Shell">irb</a> is your best friend. For those who want to get fancy you can replace it with <a href="http://pryrepl.org/">pry</a>. Let&rsquo;s see some examples, first start the irb session with the <code>irb</code> command, then let&rsquo;s see what irb has to say about <code>self</code>:</p>
<pre><code>$ irb

2.1.0 :001 &gt; self
 =&gt; main 

&gt; self.inspect
 =&gt; "main" 

self.class
 =&gt; Object 
</code></pre>
<p>There&rsquo;s always a <code>self</code> when executing ruby code: at startup irb creates a <strong>top level object</strong> called main, which is actually just an <strong>instance of the Object class</strong>. That&rsquo;s your default <code>self</code>.</p>
<h3>self at class definition</h3>
<p>Time to move on. Let&rsquo;s see what <code>self</code> is when defining a class:</p>
<pre><code>class Person
  self
end

 =&gt; Person
</code></pre>
<p>It&rsquo;s the class itself, and it makes perfect sense. Do you know about class methods? When you&rsquo;re out of the context of the class, you can define them like this:</p>
<pre><code>def Person.show_self
  self.inspect
end

Person.show_self
 =&gt; "Person" 
</code></pre>
<p>But when you&rsquo;re in the context of the class, you can define class methods in both these ways:</p>
<pre><code>class Person<br />  def Person.show_self<br />    self.inspect<br />  end<br />
  def self.show_self
    self.inspect
  end
end

Person.show_self
 =&gt; "Person"
</code></pre>
<p>They are equivalent, but the latter is preferable. Why? Because if you decide to change the name of the class, you don&rsquo;t have to change the name in the signature method as well. </p>
<h3>self in instance methods</h3>
<p>What about instance methods? Inside instance method definitions, <code>self</code> is the actual instance:</p>
<pre><code>class Person
  def show_self
    self.inspect
  end
end

Person.new.show_self
 =&gt; #&lt;Person:0x00000101939098&gt;
</code></pre>
<p>Most of the times, inside method body definitions, <code>self</code> is redundant and we can avoid it:</p>
<pre><code>class Person
  def self.show_self # class method, self is the class
    inspect
  end

  def show_self # instance method, self is the instance
    inspect
  end
end<br /></code></pre>
<pre><code>Person.show_self
 =&gt; "Person"<br /><code><br />Person.new.show_self</code></code></pre>
<pre><code> =&gt; #&lt;Person:0x00000101939098&gt;</code></pre>
<h3>When self is required</h3>
<p>There are some cases when you need an explicit <code>self</code>, namely when inside the method body there&rsquo;s a local variable with the same name of the method you&rsquo;re willing to call on <code>self</code>. If you omit <code>self</code> you&rsquo;re going get some unexpected results for sure:</p>
<pre><code>class Person
  def show_self
    inspect = 'boo!'
    inspect
  end
end

Person.new.show_self
 =&gt; "boo!"

class Person
  def show_self
    inspect = 'boo!'
    self.inspect
  end
end

Person.new.show_self
 =&gt; #&lt;Person:0x00000101939098&gt;
</code></pre>
<p>Another very common scenario when <code>self</code> is always required is when using writer methods:</p>
<pre><code>class Person
  attr_accessor :name

  def set_name(person_name)
    self.name = person_name
  end
end

me = Person.new
me.set_name 'Andrea'

me.name
 =&gt; "Andrea"
</code></pre>
<p>If we had written</p>
<pre><code>def set_name(person_name)
  name = person_name
end
</code></pre>
<p>then the code would have created a local variable named name.</p>
<p>The last important case when <code>self</code> is required is when you call the <code>class</code> method of an instance:</p>
<pre><code>class Person
  def show_class
    self.class
  end
end

Person.new.show_class 
 =&gt; Person
</code></pre>
<p>In this case self is required because class is a reserved keyword, so we need to make sure the parser undestands we&rsquo;re referencing to the instance class object.</p>
<h3>When self cannot be used</h3>
<p>Sometimes on the other hand you cannot use the explicit <code>self</code>: that&rsquo;s the case with private methods:</p>
<pre><code>class Person
  attr_accessor :name

  private :name

  def show_name
    name
  end

  def buggy_show_name
    self.name
  end
end

me.show_name 
 =&gt; "Andrea"

me.name
 NoMethodError: private method `name' called for #&lt;Person:0x00000101939098&amp; @name="Andrea"&gt;

me.buggy_show_name
 NoMethodError: private method `name' called for #&lt;Person:0x00000101939098&amp; @name="Andrea"&gt;
</code></pre>
<h3>Wrapping up</h3>
<p>In order to write idiomatic ruby, the rule of thumb is to avoid the explicit self whenever it&rsquo;s not required.</p>