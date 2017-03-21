---
ID: 172
post_title: 'Opal under a microscope: Object creation in Ruby and JavaScript'
author: Elia Schito
post_date: 2014-12-17 14:36:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/12/17/opal-under-a-microscope-object-creation-in-ruby/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/105439378224/opal-under-a-microscope-object-creation-in-ruby
tumblr_mikamayhem_id:
  - "105439378224"
---
<p><em>In this post we&rsquo;ll learn how Ruby objects are mapped in JavaScript-land by the Opal compiler, how to call methods on them and how object instantiation works for both Ruby and JavaScript.</em></p>

<h2>The basics: constants, methods and instance variables</h2>

<p>The rule by which Ruby objects are compiled into JavaScript by Opal is quite simple. Constants are registered with their regular name under the <code>window.Opal</code> (i.e. the <code>Opal</code> property on the JavaScript <code>window</code> object). Methods are mapped to properties prefixed by a <code>$</code> (dollar sign). Instance variables are just regular properties.</p>

<h3>Example:</h3>

<p>The following Ruby code:</p>
    <pre><code class="ruby">class Foo
  def initialize
    @bar = 'BAR!'
  end

  def bar
    @bar
  end

  def baz
    'BAZ!'
  end
end
</code></pre>

<p>can be used from JavaScript in this way:</p>
    <pre><code class="js">var Foo = Opal.Foo; // Constant lookup
obj = Foo.$new();   // calling method on class Foo

obj.bar             // accessing @bar      =&gt; 'BAR!
obj.$bar();         // another method call =&gt; 'BAR!'

obj.baz;            // a js property =&gt; undefined
obj.$baz();         // another method call =&gt; 'BAZ!'
</code></pre>

<p>As a Ruby developer you may be surprised that <code>obj.baz</code> returns <code>undefined</code> and not <code>nil</code> (actually <code>window.Opal.nil</code>) as that&rsquo;s the value that you would expect to find while reading an instance variable for the first time.</p>

<p>What happens is that the Opal compiler will do just that, as long as you access those instance variables from Ruby code. In fact it statically analyzes the class&rsquo; code and pre-initializes to <code>nil</code> any instance variable for which it can find a reference.</p>

<p>Now that we have the basics let&rsquo;s go further and put <code>Foo.new</code> under the microscope for both Opal and CRuby.</p>

<h2>The birth of an object in 3 steps</h2>

<p>By looking in detail how object instantiation is done in JavaScript by the Opal compiled code we&rsquo;ll have a chance to learn how it actually works in CRuby.</p>

<h3>The implementation of .new</h3>

<p>At the cost of oversimplifying here&rsquo;s the rough implementation of the <code>#new</code> method available to all classes in Ruby:</p>
    <pre><code class="ruby">class Class
  def new(*args, &amp;block)
    obj = allocate # allocates the memory in CRuby
    obj.send(:initialize, *args, &amp;block) # forwards all arguments to #initialize
    return obj
  end
end
</code></pre>

<p>Breaking it down we see that:</p>

<ol><li>it calls <code>#allocate</code> in order to setup a raw object (in C it will also <em>allocate</em> the necessary memory)</li>
<li>it forwards all arguments and block to <code>#initialize</code></li>
<li>it returns the object</li>
</ol><p>Here&rsquo;s how it looks in Opal (the code inside <code>%x{}</code> is JavaScript):</p>
    <pre><code class="ruby">class Class
  def new(*args, &amp;block)
    %x{
      var obj = self.$allocate();
      obj.$initialize.$$p = block;
      obj.$initialize.apply(obj, args);
      return obj;
    }
  end
end
</code></pre>

<p>(<a href="https://github.com/opal/opal/blob/master/opal/corelib/class.rb#L42-L50"><code>Class#new</code> source</a>)</p>

<p>As you can see the the only notable difference is in how the block is passed. Opal will store any block call on the method itself under the <code>$$p</code> property. This way blocks can&rsquo;t be confused with regular arguments. Apart from that it&rsquo;s clear that it&rsquo;s fundamentally the same code.</p>

<h3>Uncovering Class#allocate</h3>

<p>The curious reader at this point is wondering what the allocate method does in JavaScript because surely it can&rsquo;t manage memory. Let&rsquo;s see the implementation:</p>
    <pre><code class="ruby">class Class
  def allocate
    %x{
      var obj = new self.$$alloc;
      obj.$$id = Opal.uid();
      return obj;
    }
  end
end
</code></pre>

<p>(<a href="https://github.com/opal/opal/blob/master/opal/corelib/class.rb#L31-L37"><code>Class#allocate</code> source</a>)</p>

<p>Let&rsquo;s break it down by line, but this time we&rsquo;ll go in reverse order:</p>

<ol><li>The last line is the easiest one in which the code just returns the object: <code>return obj</code>.</li>
<li>The middle one is still quite clear and seems to just assign a unique identifier to the object. That value will be the one returned by <code>#object_id</code>.</li>
<li>The first and most important one is where stuff actually happens. The JavaScript <code>new</code> keyword is used to create an object whose constructor seem to be stored in <code>$$alloc</code>.</li>
</ol><p>For those not very familiar with JavaScript I&rsquo;ll show how objects are usually created:</p>
    <pre><code class="javascript">// this function acts as the MyClass constructor
function MyClass() { this.foo = 'bar'; }

var obj = new MyClass;
obj.foo         // =&gt; 'bar'
obj.constructor // =&gt; MyClass
</code></pre>

<p>The awesome thing to me is that Opal manages to have an implementation that is really idiomatic in both Ruby and JavaScript.</p>