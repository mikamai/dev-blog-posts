---
ID: 19
post_title: >
  The easiest way to add a Datepicker in
  Rails
author: Matteo Revelli
post_date: 2016-07-13 08:02:10
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/07/13/the-easiest-way-to-add-a-datepicker-in-rails/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/147332766824/the-easiest-way-to-add-a-datepicker-in-rails
tumblr_mikamayhem_id:
  - "147332766824"
---
A few days ago, I had to put in a registration form for a website, written in Ruby, a tool which gives you the possibility to enter the date, more easily, in the desired format. I decided to use a datepicker in order to allow users to choose the date rather than writing it (a more complex procedure). If a date is chosen, feedback is shown as the input’s value.

At the beginning, the implementation of the interactive calendar was a bit problematic and long, until I found a gem that did the whole work for me. The gem, called Bootstrap Datepicker for Rails, runs from the Rails version 3.1 up to the latest. As usual, to install it just add it to Gemfile with <code>gem 'bootstrap-datepicker-rails'</code> and launch the bundle install.<!--more-->

The configuration is as much as simple than before and it consists of adding two lines of code into the application.css and application.js, in order to assign the Bootstrap style and behaviour.
<pre><code>*= require bootstrap-datepicker // for css
//= require bootstrap-datepicker // for js</code></pre>
At this point, the datepicker is perfectly configured and installed: you must add it into the form where you need it. However, before doing this, we need to customize our interactive calendar to give us the correct date format as output. I found on Google that there is a great tool that allows you to customize your datepicker a lot, without having to waste too much time. In fact, this tool enables you to change many features such as the “start dates”, “end dates”, the type of the date separator, the disabled days of week, etc..

Once you do that, since it was necessary to add many datepickers in a view, I created a helper to manage it in a cleaner way.
<pre><code>
  def datepicker_input form, field
    content_tag :td, :data =&gt; {:provide =&gt; 'datepicker', 'date-format' =&gt; 'yyyy-mm-dd', 'date-autoclose' =&gt; 'true'} do
      form.text_field field, class: 'form-control', placeholder: 'YYYY-MM-DD'
    end
  end
 </code></pre>
After that, I put in the view the input fields for the dates and their related interactive calendars.
<pre><code>&lt;%= datepicker_input form, :accepted_at_start%&gt;
&lt;%= datepicker_input form, :accepted_at_end%&gt;
.
.</code></pre>
The result was very gratifying because in a very short time I was able to implement a fantastic feature within my project.
Again, all this has been possible thanks to one of the many gems of ruby!