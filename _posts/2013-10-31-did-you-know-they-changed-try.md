---
ID: 324
post_title: 'Did you know they changed #try ?'
author: Nicola Racco
post_date: 2013-10-31 14:40:05
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/10/31/did-you-know-they-changed-try/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/65614353244/did-you-know-they-changed-try
tumblr_mikamayhem_id:
  - "65614353244"
---
Did you know `#try` has been changed since Rails 4? I didn’t.

Look at the implementation for [Rails 4](https://github.com/rails/rails/blob/master/activesupport/lib/active_support/core_ext/object/try.rb#L45) and [Rails 3](https://github.com/rails/rails/blob/3-2-stable/activesupport/lib/active_support/core_ext/object/try.rb#L36). Rails 4 implementation uses `respond_to?` to check the method presence before calling it via send. It's a subtle change, but it's enough to become a problem: In Rails 3 you could use try even when you are dealing with method_missing. Now you have to override the `respond_to?` to make it work. E.g.:

```ruby
class A
  def method_missing name, *args, &amp;block
    if name.to_s == &#039;foo&#039;
      &#039;bar&#039;
    else
      super
    end
  end
      
  def respond_to? name
    name.to_s == &#039;foo&#039; || super
  end
end
    
A.new.try :foo # this returns nil if we don&#039;t override #respond_to?
```