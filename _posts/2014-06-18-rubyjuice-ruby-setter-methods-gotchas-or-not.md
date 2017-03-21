---
ID: 237
post_title: '[RubyJuice] Ruby setter methods gotchas&#8230; or not?'
author: Andrea Longhi
post_date: 2014-06-18 08:38:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/06/18/rubyjuice-ruby-setter-methods-gotchas-or-not/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/89143504484/rubyjuice-ruby-setter-methods-gotchas-or-not
tumblr_mikamayhem_id:
  - "89143504484"
---
<p>This article is more a curiosity than something that will prove useful in the near future, still I think that knowing the corner cases of programming languages is fun and healthy, so let&rsquo;s start right now!</p>
<p>You were taught that ruby methods always return the last sentence evaluated, so</p>
<pre><code>def hello
  "hello world!"
end</code></pre>
<p>will unsurprisingly return the string <code>"hello world!"</code>. This makes totally sense and it&rsquo;s very practical, discouraging the use of explicit <code>return</code> statements when not strictly required.</p>
<p>So what&rsquo;s up with writer methods?</p>
<p>You probably already know that you can get them for free using class macros such as <code>attr_writer</code> or <code>attr_accessor</code>. These code snippets are totally equivalent when it comes to their behavior (the first implementation being slightly faster because writtern in C):</p>
<pre><code>class Cat
  attr_writer :name
end

class Cat
  def name=(name)
    @name = name
  end
end</code></pre>
<p>It should come with no surprise that the return value of the above defined method is the <code>name</code> parameter value:</p>
<pre><code>Cat.new.name = 'Tom'
# =&gt; "Tom"</code></pre>
<p>What if for some odd reason we want to return something different than the name? Let&rsquo;s try to achieve that:</p>
<pre>class Cat
  def name=(name)
    @name = name
    "dear cat, I dub you #{name}"
  end
end

Cat.new.name = 'Felix'
# =&gt; "Felix"</pre>
<p>What&rsquo;s going on? Where&rsquo;s the string that was supposed to be returned? Well, it happens you cannot change the return value of a setter method, it will always return the parameter. That&rsquo;s the rule. Still, you can have some fun with that parameter:</p>
<pre><code>class Cat
  def name=(name)
    @name = name &lt;&lt; ' the Cat'
  end
end

Cat.new.name = 'Felix'
# =&gt; "Felix the Cat"</code></pre>
<p>Eventually we managed to have a different return value ;-) This actually works because we&rsquo;re just adding some characters to the original string, not returning a totally different object.</p>
<p>Errors are a totally different case, and they behave just like you&rsquo;d expect:</p>
<pre><code>class Cat
  def name=(name)
    raise "I'm an exceptional cat, I get angry when I get a name!"
    @name = name
  end
end

Cat.new.name = 'Felix'
# RuntimeError: I'm an exceptional cat, I get angry when I get a name!</code></pre>
<p>Anyway surprises are not over yet. What about multiple parameters? You can define such method, but you won&rsquo;t be able to put it to good use:</p>
<pre><code>class Cat
  def name=(name, surname)
    @name    = name
    @surname = surname
  end
end

Cat.new.name = 'Felix', 'the Cat'
# ArgumentError: wrong number of arguments (1 for 2)</code></pre>
<p>What? the interpreter is expecting 2 arguments, but only one was found&hellip; hey we clearly provided 2 arguments! Let&rsquo;s add parentheses, maybe that&rsquo;s going to fix it:</p>
<pre>  <code>Cat.new.name=('Felix', 'the Cat')
# SyntaxError: (irb):25: syntax error, unexpected ',', expecting ')'
# Cat.new.name=('Felix', 'the Cat')
#                       ^</code></pre>
<p>That&rsquo;s even worse, and at this point I&rsquo;m out of ideas&hellip; I can&rsquo;t make it work.</p>
<p>What we&rsquo;ve learned so far about custom setter methods:</p>
<ul><li>they accept only one parameter</li>
<li>they always return the parameter, except when an error is raised</li>
<li>you can modify the return value of the method only if you mutate the original object</li>
</ul><p>There&rsquo;s still one use case I&rsquo;d like to address, multiple args using a splat:</p>
<pre><code>class Cat
  attr_reader :name, :surname
  
  def name=(*args)
    @name    = args.first
    @surname = args.last
  end
end

felix = Cat.new
felix.name = 'Felix', 'the Cat'
# =&gt; ["Felix", "the Cat"]</code></pre>
<p>So, finally we found a way to pass multiple params without getting an exception. This actually does work because <code>*args</code> is converted to a single argument, an array, and that&rsquo;s exactly what is eventually returned by the method call.</p>
<p>Are we done? No, because that method didn&rsquo;t set the instance variables as you probably expected:</p>
<pre><code>felix.name
# =&gt; ["Felix", "the Cat"]
felix.surname
=&gt; ["Felix", "the Cat"]</code></pre>
<p>Ok, let&rsquo;s try another time:</p>
<pre>  <code>class Cat
  attr_reader :name, :surname
  
  def name=(*args)
    @name    = args.first.first
    @surname = args.first.last
  end
end

felix = Cat.new
felix.name = 'Felix', 'the Cat'
felix.name
# =&gt; "Felix"
felix.surname
# =&gt; "the Cat"</code></pre>
<p>And now it works as expected. Inside the setter method it happens that <code>args</code> gets wrapped with another array; this is confirmed by inspecting <code>args</code> with <a href="http://pryrepl.org/">pry</a> inside the method definition:</p>
<pre><code>[1] pry(main)&gt; args
=&gt; [["Felix", "the Cat"]]</code></pre>
<p>I hope that finding out some quirk behavior was fun for you as it was fun for me, this post ends now but the natural next step would be to explore Ruby C source code and see why things happen this way, and I encourage you to take that step.</p>