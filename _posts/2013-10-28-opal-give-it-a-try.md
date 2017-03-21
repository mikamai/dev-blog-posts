---
ID: 327
post_title: 'Opal: give it a try'
author: Nicola Racco
post_date: 2013-10-28 08:43:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/10/28/opal-give-it-a-try/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/65322179075/opal-give-it-a-try
tumblr_mikamayhem_id:
  - "65322179075"
---
If you are a Ruby developer, you're probably constantly complaining about Javascript's quirks :-)

Yes, [Coffeescript](http://coffeescript.org/) is a great abstraction. It gives you a nice syntax that is both easier to read and easier to write. Stuff like the [fat arrow](Fat Arrow explaination) and the [class](http://arcturo.github.io/library/coffeescript/03_classes.html) keyword are undoubtfully better than their vanilla JavaScript counterparts. In addition Coffeescript compiles to plain Javascript without the need of any framework or third party library. In a nutshell, Coffeescript is the perfect tool when you need to write all that javascript glue to work with jquery plugins, backbone and all that stuff that we know well.

But when you need to write your own framework or to write a more complex app, even Coffeescript could not be the right tool.
<!--more-->

Usually what I miss the most are features like [barewords](http://devblog.avdi.org/2012/10/01/barewords/) and [modules](http://ruby-doc.com/docs/ProgrammingRuby/html/tut_modules.html).

Luckily some guys decided to bring Ruby coolness in Javascript and started **[Opal](http://opalrb.org/)**, a Ruby to Javascript compiler. It doesn't give you only the Ruby nice syntax to work with, it gives you nearly all Ruby's power in all its shine. Here's a short example:

```ruby
# We have some elements that should be centered on the screen
# We create an object for each element. Each object should define an &#039;el&#039; method
# returning the tag and include the Centerable module
module Centerable
  def move_to_center
    el.css &#039;position&#039;, &#039;absolute&#039;
    el.css &#039;left&#039;, (el.parent.width - el.width) / 2
    el.css &#039;top&#039;,  (e.parent.height - el.height) / 2
  end
end
 
class OutputWindow
  include Centerable
  attr_reader :el
  
  def initialize selector
    @el = Element.find selector
  end
  
  def show
    el.show
    Window.on :resize, method(:move_to_center)
  end
end
    
Document.ready? do
  OutputWindow.new &#039;#output&#039;
end
```

I’ll give you a moment to imagine how your Javascript code could look like :-)

![Dreamy World](https://dev.mikamai.com/wp-content/uploads/2013/10/dreamy_world.jpg) {.aligncenter}

In addition to the plain syntax and core libraries, the Opal guys are constantly working on [opal-jquery](https://github.com/opal/opal-jquery) (jquery bindings), [opal-browser](https://github.com/opal/opal-browser) (browser support for opal: bindings for `Window` just to mention one), [vienna](https://github.com/opal/vienna#readme) (an MVC framework).

### Okay, let's talk of side effects (not problems, side effects)

If you're thinking that "with great shiny comes great side effects" you are right.

Opal isn't a simple syntax converter. It will embed in your code its library, used to give you the Ruby Core API. Just a thing to be aware of.

As I wrote above, it gives you **nearly** all the Ruby power. In fact some features are not exactly the same. An example is the String object. In ruby strings can be modified using bang methods, but in Opal this could be achieved only wrapping every javascript string in another object, and this would affect performances so the Opal guys decided to not include this support by default.

Naturally Opal doesn't want to break any other plain javascript code. For this reason it sandboxes all the code inside an Opal object and prefixes all the methods with a `$` sign. If we want to call the show method of the example described above using plain javascript we could type:

```ruby
Opal.OutputWindow.$new()
```

### But what if you need to call methods or read properties from native JavaScript objects?

As in Coffeescript code wrapped inside backticks (or x-strings `%x{}`) is preserved in the conversion. So, let's say you want to write a method that returns the context of a canvas element, you’ll write something like:

```ruby
def context(el) 
  `#{el}.getContext('2d');`
end
```

You can interpolate Ruby code that will be translated to JavaScript and mixed in the final JavaScript.

### Not so nice to see, right?

Luckily enough Opal provides also a `Native` class that can wrap any JavaScript object making it as Ruby as it can be. Let’s see an example with JavaScript's `window`:

```ruby
$window = Native(`window`)
puts $window[:location][:href]                         # => http://localhost/
puts $window.document.querySelector('title').innerText # => "Test page"
```

Naturally you'll come to this problem when you’ll need to work with third party libraries written in plain javascript. The best way to solve this problem is to write a proxy class that wraps all the calls done through Opal (a good example is the [Document definition](https://github.com/opal/opal-jquery/blob/master/opal/opal-jquery/document.rb) in opal-jquery). I tried this approach some days ago and I have to say that it's a good solution. I'll show you an example the next time.

In the end, if you are writing a framework or a new web application from scratch, maybe Opal can solve all your problems. If you're planning to embed it in your existing application be sure to have a good grasp on how it will interact with your existing JavaScript objects and libraries first. But it's surely a shiny product :-)