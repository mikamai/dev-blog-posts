---
ID: 200
post_title: 'Analyze ruby execution stack with #caller'
author: Andrea Longhi
post_date: 2014-10-08 08:11:17
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/10/08/analyze-ruby-execution-stack-with-caller/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/99472635014/analyze-ruby-execution-stack-with-caller
tumblr_mikamayhem_id:
  - "99472635014"
---
<p>Sometimes you may need (or just want) to see the current execution stack of your ruby code for inspection: for this very purpose Ruby core library provides a method named <strong>caller</strong>. Its output is similar to the one of error backtraces, so you should be able to get very familiar with it quite soon.</p>
<p><code>caller</code> provides some very useful debugging capability, is defined inside the Kernel module, which in turn is included into Object making it available anywhere. Let&rsquo;s start with the simplest example, create a file and save it with the following content:</p>
<pre>def a
  puts 'Your code execution stack:'
  puts caller(0)
end

def b
  a
end

b</pre>
<p>When executed, the file output is:</p>
<pre><code>
$ ruby example.rb 

Your code execution stack:
example.rb:3:in `a'
example.rb:7:in `b'
example.rb:10:in `'
</code></pre>
<p>As you can see this is pretty informative, as it shows every step involved in the code execution, listing the filename, the method and the line number where execution passed to each next method. When you&rsquo;re dealing with a complex application this can become a lifesaver. We&rsquo;re seeing the result printed on the screen, but actually <code>caller</code> returns an array of strings (i.e. the lines).</p>
<p>What about that <code>0</code> param? It tells how many initial entries from the stack are to be omitted, it is optional and the default is <code>1</code>. If you remove the zero then you will see that the first line <code>example.rb:3:in `a'</code> disappears from output.</p>
<p><code>caller</code> accepts a second parameter, which represents how many lines are to be collected in the array. If you change the code to <code>caller(0, 2)</code> then the last line of the output will disappear, listing only the first 2 lines.</p>
<p>Now, let&rsquo;s see a real use case when caller comes handy. I&rsquo;ve created a simple rails app with the only purpose of listing all the rails source files involved in a simple request, plus the ability of showing the files content directly in the browser. The application can be downloaded here: <a href="https://github.com/mikamai/read_rails">read_rails</a>.</p>
<p>Most of my code is in application_controller.rb, routes.rb and views/application/index.html.erb, you can look at those files if you&rsquo;re interested in how it works, but most of the interesting code is in the erb view listed below:</p>
<pre><code>&lt;pre&gt;
  &lt;% caller(0).each_with_index do |line, i| %&gt;
    &lt;% filename, line_number = line.split(':') %&gt;
    &lt;%= i+1 %&gt; &lt;%= link_to line, "#{filename}#n#{line_number}", target: '_blank' %&gt;
  &lt;% end %&gt;
&lt;/pre&gt;</code></pre>
<p>Bundle the gems, start the server and head to http://localhost:3000 to see the list of the files and methods involved in the request cycle. Each line, if clicked, will show the actual rails source code installed on your machine and used by the application.</p>
<p><img alt="image" src="https://raw.githubusercontent.com/mikamai/read_rails/master/public/screenshot.png" /></p>
<p>What can we learn from this backtrace?</p>
<p>First, rails is a complex beast: about 110 different method calls (and about the same size in files) are involved to complete the request.</p>
<p>Second: we hit the whole rails MVC stack, plus routing, rack and webrick.</p>
<p>Third: this simple request hits only one single file inside the real app (the index view), the rest is library code. Of course, when you add database queries, assets and javascripts the count increases.</p>
<p>At this point, you may want to pry into rails guts and see what&rsquo;s really happening at each point, so have fun!</p>