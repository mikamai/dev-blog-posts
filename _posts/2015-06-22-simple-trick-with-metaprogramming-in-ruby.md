---
ID: 108
post_title: >
  Simple trick with Metaprogramming in
  Ruby
author: Diomede Tripicchio
post_date: 2015-06-22 12:15:15
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/06/22/simple-trick-with-metaprogramming-in-ruby/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/122161310604/simple-trick-with-metaprogramming-in-ruby
tumblr_mikamayhem_id:
  - "122161310604"
---
<p>Today i will show you a little trick I’ve used in a recent project. Nothing so special, just a little bit of Metaprogramming with Ruby</p>
<!--more-->

<p>I have a constant variable like this:</p>

<pre><code class="“ruby”">
class Foo
    DEFAULT_VALUES = {
        foo: 1,
        bar: 2,
        baz: 3
    }
end
</code></pre>

<p>Ok, great. Now every time I want one value from this hash I need use a syntax like this:  <strong>Foo::DEFAULT_VALUES[:foo]</strong>. That rare beauty!</p>

<p>However I’m lazy, I don’t want <strong>::</strong> and <strong>[]</strong> spread around in my code and I don&rsquo;t want to use CAPITAL_CASE if I can avoid it.</p>

<p>Hence, let&rsquo;s define some methods!! (I am going to use class methods)</p>

<pre><code class="“ruby”">
class Foo
    DEFAULT_VALUES = {
        foo: 1,
        bar: 2,
        baz: 3
    }

    class &lt;&lt; self
        def default_foo
            DEFAULT_VALUES[:foo]
        end
        def default_bar
            DEFAULT_VALUES[:bar]
        end
        def default_baz
            DEFAULT_VALUES[:baz]
        end
    end
end
</code></pre>

<p>Much much better, but it can become even better. I can use a cleaner syntax to retrieve what I need writing <strong>Foo.default_foo</strong> but I have violated the DRY principle.</p>

<p>So, let’s fix this code with some refactoring:</p>

<pre><code class="“ruby”">
class Foo
  DEFAULT_VALUES = {
    foo: 1,
    bar: 2,
    baz: 3
  }

  class &lt;&lt; self
    DEFAULT_VALUES.each do |k, v|
      define_method "default_#{k}" do
        v
      end
    end
  end
end
</code></pre>

<p>In this way, using <strong>metaprogramming</strong> we can DRY up our code. Thanks to <strong>define_method</strong> we can declare methods at runtime based on the values of <em>DEFAULT_VALUES</em>.</p>

<p>In this way we can edit the <em>DEFAULT_VALUES</em> hash in just one place and the metaprogramming will do the rest. For example:</p>

<pre><code class="“ruby”">
class Foo
    DEFAULT_VALUES = {
        new_value: "string",
        hello: "world"
    }

    …
end

Foo.default_new_value # string
Foo.default_hello # world
</code></pre>

<p>I hope this post can help you solve some boring problems.</p>

<p>Bye and see you soon!</p>