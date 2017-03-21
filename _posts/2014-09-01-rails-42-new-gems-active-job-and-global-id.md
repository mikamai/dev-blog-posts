---
ID: 215
post_title: 'Rails 4.2 new gems: active job and global id'
author: Andrea Longhi
post_date: 2014-09-01 08:31:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/09/01/rails-42-new-gems-active-job-and-global-id/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/96343027199/rails-42-new-gems-active-job-and-global-id
tumblr_mikamayhem_id:
  - "96343027199"
---
<p>A few days ago the new rails 4.2 beta release was <a href="http://weblog.rubyonrails.org/releases/">announced</a> and the usual bag of goodies is going to make our life as developers easier and more interesting.</p>
<p>The most relevant new feature is the <strong>Active Job</strong> framework and its integration with ActionMailer: rails has now its own queue interface and you can swap the queuing gem (resque, delayed job, sidekiq&hellip;) without changing your application code. <br />You can even send email asyncronously with the new <code>deliver_later</code> method, while if you want sync delivery you should use instead <code>deliver_now</code> because the old <code>deliver</code> method is now deprecated.<br /> If you don&rsquo;t add any queue gem to the Gemfile then the default rails system will be used, which means that everything will be sent immediately (no async functionality).</p>
<p>This brilliant feature depends on a couple of new gems: <a href="https://github.com/rails/rails/tree/master/activejob">active job</a> and <a href="https://github.com/rails/globalid">global id</a>. Active Job usage is well documented in the <a href="http://edgeguides.rubyonrails.org/active_job_basics.html">new official guide</a>, so I will focus most of this article on Global Id.<br /> If you want to follow along you can download a demo app from <a href="https://github.com/mikamai/activejob_example">this repository</a>, bundle the gems and start the server with <code>rails s</code>. The example is basically a rails 4.2 app with a regular ActiveRecord Friend scaffold and a NameCapitalizerJob job class.</p>
<p>How do the new gems interact exactly? Let&rsquo;s enqueue a new job for the NameCapitalizerJob worker, which is located in the example application:</p>
<pre><code>  NameCapitalizerJob.perform_later Friend.first
  =&gt; #&lt;NameCapitalizerJob:0x007fb44acaff48 ...&gt;
</code></pre>
<p>If you used rails queuing systems in the past you already know that it was necessary to pass your ActiveRecord objects to the worker in the form of their id and manually reload the record at job execution. This is no more required, so in the example above we&rsquo;re simply passing the record itself, while in the job code the reload is automatic:</p>
<pre><code>NameCapitalizerJob &lt; ActiveJob::Base
  def perform(friend)
    name = friend.name.capitalize
    friend.update_attribute :name, name
  end
end
</code></pre>
<p>You can see the job has been correctly enqueued:</p>
<pre><code>Delayed::Job.count
 =&gt; 1
</code></pre>
<p>with the following params:</p>
<pre><code>YAML.load(Delayed::Job.first.handler).args
 =&gt; [NameCapitalizerJob, "4a33725b-35cf-4940-b1ca-d6fad84d410f", "gid://activejob-example/Friend/1"]
</code></pre>
<p>These params represent: the job class name, the job id, and the global id string representation for the <code>Friend.first</code> record.<br /> The format of the global id string is <code>gid://AppName/ModelClassName/record_id</code>.<br /> Given a gobal id, the worker will load the referenced record transparently. This is achieved by mixing into ActiveRecord::Base a module from the global id gem:</p>
<pre><code>puts ActiveRecord::Base.ancestors
...
GlobalID::Identification
...
</code></pre>
<p>The <code>GlobalID::Identification</code> module defines only a couple of methods: <code>#global_id</code>, <code>#signed_global_id</code> and their aliases <code>#gid</code> and <code>#sgid</code>, where the first is the record&rsquo;s globalid object, the second is the record&rsquo;s signed(encrypted) version of the same object:</p>
<pre><code>gid = Friend.first.gid
 =&gt; #&lt;GlobalID:0x007fa9add041f8 ...&gt;
gid.app
 =&gt; "activejob-example" 
gid.model_name
 =&gt; "Friend" 
gid.model_class
 =&gt; Friend(id: integer, name: string, email: string...)
gid.model_id
 =&gt; "1"   
gid.to_s
 =&gt; "gid://activejob-example/Friend/1"

sgid = Friend.first.sgid
 =&gt; #&lt;SignedGlobalID:0x007fa9add15e58 ...&gt;
sgid.to_s
 =&gt; "BAh7CEkiCGdpZAY6BkVUSSIlZ2lkOi8vYWN0aXZl..."
</code></pre>
<p>The most important thing here is the string representation of the gid, as it contains enough information to retrieve its original record:</p>
<pre><code>GlobalID::Locator.locate "gid://activejob-example/Friend/1"
 =&gt; #&lt;Friend id: 1, name: "John smith" ...&gt;
</code></pre>
<p>The actual source code used for locating records is rather simple and self explanatory:</p>
<pre><code>class ActiveRecordFinder
  def locate(gid)
    gid.model_class.find gid.model_id
  end
end  
</code></pre>
<p>Regarding the signed object, we can inspect the original data hidden into its string representation using the following code:</p>
<pre><code>SignedGlobalID.verifier.verify sgid.to_s
 =&gt; {"gid"=&gt;"gid://activejob-example/Friend/1", "purpose"=&gt;"default", "expires_at"=&gt;Mon, 29 Sep 2014 08:25:31 UTC +00:00}
</code></pre>
<p>That&rsquo;s how the global id gem works inside rails. By the way, it&rsquo;s rather easy to use it with your own classes. Let&rsquo;s see how to do it.<br /> Let&rsquo;s start a new irb session with <code>bundle exec irb</code> from the app folder, and require it with <code>require 'globalid'</code>.<br /> The needs an <code>app</code> namespace in order to generate ids, so we need to set it manually:</p>
<pre>GlobalID.app = 'TestApp'</pre>
<p>Now let&rsquo;s build a PORO class that defines globalid required methods (<code>::find </code> and <code>id</code>) and includes the <code>GlobalID::Identification</code> module:</p>
<pre><code>class Item
  include GlobalID::Identification
  
  @all = []
  def self.all; @all end
  
  def self.find(id)
    all.detect {|item| item.id.to_s == id.to_s }
  end
  
  def initialize
    self.class.all &lt;&lt; self
  end
    
  def id
    object_id
  end
end</code></pre>
<p>As you might guess, the <code>::find</code> method retrieves an item from its id code, while the <code>#id</code> method is simply an identifier. It works like this:</p>
<pre><code>  item = Item.new
   =&gt; #&lt;Item:0x007fdb4b05da10&gt;
  id = item.id
   =&gt; 70289916620040 
  Item.find(id)
   =&gt; #&lt;Item:0x007fdb4b05da10&gt;
</code></pre>
<p>Time to get the item global id:</p>
<pre><code>  gid = item.gid
   =&gt; #&lt;GlobalID:0x007fdb4b026358 ...&gt;
  gid.app
   =&gt; "TestApp"
  gid.model_name
   =&gt; "Item"
  gid.model_id
   =&gt; "70289916620040"
  gid_uri = gid.to_s
   =&gt; "gid://TestApp/Item/70289916620040"
</code></pre>
<p>We can now retrieve the original Item object from the gid_uri:</p>
<pre><code>found = GlobalID.locate gid_uri
 =&gt; #&lt;Item:0x007fdb4b05da10&gt;
found == item
 =&gt; true
</code></pre>
<p>That&rsquo;s it! I encourage you to check out the global id gem code on github, it&rsquo;s just a few files so it&rsquo;s very readable.</p>