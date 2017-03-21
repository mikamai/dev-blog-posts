---
ID: 617
post_title: Ruby and the forgotten thrown symbol
author: Nicola Racco
post_date: 2017-01-18 15:00:22
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/01/18/ruby-and-the-forgotten-thrown-symbol/
published: true
---
As usual Rails 5 brought many changes. Among these, there's the new callbacks behaviour for ActiveRecord that uncovered an old and forgotten ruby construct: `throw`.
<!--more-->

In case you don't know what I'm talking about, let's say I don't want _captains_ to be removed from my app, but plain users only. In Rails 4 I can write the following:

```ruby
class User &lt; ActiveRecord::Base 
  scope :plain, -&gt; { where captain: false }
  scope :captain, -&gt; { where captain: true }

  before_destroy :ensure_not_captain

  private

  def ensure_not_captain
    return true unless captain?
    errors.add :base, &#039;Captains cannot be destroyed&#039;
    return false
  end
end
```

This results in the following:

```ruby
plain_user = User.plain.first
plain_user.destroy # user is destroyed
captain = User.captain.first
captain.destroy # =&gt; false
captain.errors.full_messages.first # =&gt; &#039;Captains cannot be destroyed&#039;
```

But Rails 5 changed this, and now the above code wouldn't work because returning `false` inside a callback will not halt the flow anymore. Instead you have to do the following:

```ruby
def ensure_not_captain
  return unless captain?
  errors.add :base, &#039;Captains cannot be destroyed&#039;
  throw :abort
end
```

**Okay, what's that throw?** At the very moment I looked at this code I thought it could be a synonym of `raise`. Why not? Our `raise` is after all a `throw` in Java.

But in that code I read `throw :abort`. I'm sending a symbol to `throw`, while everyone knows that `raise` accepts only strings, exception classes or exception instances:

```ruby
raise Exception # works
raise Exception.new # works
raise &#039;asd&#039; # works
raise :asd # raises a TypeError
raise Object # raises a TypeError
raise Object.new # raises a TypeError
```

I had to go back to my first ruby studies to find this `throw` keyword I had completely forgotten about, because since I switched to Ruby (it was the end of the 2010) I never ever used or read (not a single time) the **throw** keyword. Never.

**Again, what's that `throw` then?** Differently from the `raise` keyword, the `throw` is not used for errors but for control flow. It's used in pair with `catch` and its usage is similar to `raise`:

```ruby
songs = []
catch :done do
  while typed = gets.strip
    throw :done if typed == &quot;!&quot;
    songs &lt;&lt; typed
  end
end
puts &quot;You typed the following songs: #{songs}&quot;
```

The above code will continue asking for song names until I type a "!". In that case it will throw `:done` existing from the `catch` block. Let's see a more complex example:

```ruby
def generate_random_numbers
  catch :random_numbers do
    result = []
    10.times do
      number = rand 100
      throw :random_numbers, result if number &lt; 10
      result &lt;&lt; number
    end
    result
  end
end
generate_random_numbers # =&gt; [10, 27]
generate_random_numbers # =&gt; [55, 75]
generate_random_numbers # =&gt; [28, 53, 40, 13, 76, 22, 41, 20, 15, 44]
```

This block will generate up to 10 random numbers. But it will exit with the current ones as soon as it gets a number below 10. This can be done because by default a `catch` block will return the last statement but you can ovverride it specifying the second argument in `throw`.

And last but not least, in case you are thinking you could anyway live without `throw` and using the `raise` also for control flow, there's the performance thing. Exceptions are slow, really really slow, because raising an exception forces ruby to build and dump the stack trace. The `throw/catch` is instead blazingly fast because no stack trace needs to be built.

So, thank you Rails for having unconvered the `throw/catch` construct. I had completely forgotten about it, and I promise I'll try to use it more because I think it can improve my code.