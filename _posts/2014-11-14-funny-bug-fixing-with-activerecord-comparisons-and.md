---
ID: 186
post_title: >
  Funny bug fixing with ActiveRecord
  comparisons and Comparable
author: Andrea Longhi
post_date: 2014-11-14 15:59:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/11/14/funny-bug-fixing-with-activerecord-comparisons-and/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/102615202294/funny-bug-fixing-with-activerecord-comparisons-and
tumblr_mikamayhem_id:
  - "102615202294"
---
<p>Today I was facing a very annoying bug in some old code that demonstrated to be quite hard to track down.</p>
<p>There&rsquo;s a form where the user can set some dates, which have to be valid, meaning that each date must be greater than the previous one. Here&rsquo;s a screenshot of the relevant part of the form:</p>
<p><img alt="image" src="http://68.media.tumblr.com/e43ddf2886998026f56763aac602257d/tumblr_inline_nekr3wslr91s337n9.png" /></p>
<p>Each date in the form is actually part of an Event ActiveRecord object. Each event belongs to a calendar, which is the actual record that gets saved when the form is submittted.</p>
<p>In order to validate the correct order of the dates the Comparable module was included into the Event model. Here it is some pseudocode, a simplified version of the original one:</p>
<pre><code>
class Event &lt; ActiveRecord::Base
  include Comparable
  
  belongs_to :calendar

  def &lt;=&gt; other
    if calendar == other.calendar and calendar.present?
      date &lt;=&gt; other.date
    end
  end
  # ...
end

class Calendar
  has_many :events
  
  validate :validate_event_dates_sequence
  #....
end
</code></pre>
<p>The starship method (&lt;=&gt;) in the Event class is quite straightforward: it compares the records dates in order to decide which one is greater, given that they belong to the same calendar. Calendars have many events, and its the calendar responsibility to make sure all events have dates in the right order via the validate_event_dates_sequence method.</p>
<p>So, where&rsquo;s the problem? When you edit an existing calendar and try to change the first date value, validations somehow fail and an extra date coming out of the blue is added to the form. We used to have 5 events, now we have 6:<img alt="image" src="http://68.media.tumblr.com/ecb2e04758fc33b114bfefb3a6e830b5/tumblr_inline_neksv8O1TE1s337n9.png" /></p>
<p>Why is that happening? I used <a href="http://pryrepl.org/">pry</a> in order to inspect the issue and I noticed that <code>calendar.events</code> contained an extra record, which had the same id of the first one:</p>
<pre><code>
calendar.events.map &amp;:id
# =&gt; [16, 17, 18, 19, 20, 16]
</code></pre>
<p>For some reason ActiveRecord instantiated a new record instead of using the existing one. To confirm this oddity I did some more inspection. All records are saved in the database and that&rsquo;s expected since I&rsquo;m editing existing data:</p>
<pre><code>
events.map &amp;:persisted?
=&gt; [true, true, true, true, true, true]
</code></pre>
<p>The first record and the last one, while having the same id and referring to the same record in the database, are totally different objects for ruby:</p>
<pre><code>
events.first.object_id
=&gt; 2162200580
events.last.object_id
=&gt; 2214847000
</code></pre>
<p>Event dates are not ordered from smaller to greater (the last date, which should have replaced the first one, is in the wrong position as well):</p>
<pre><code>
[Thu, 06 Nov 2014 15:49:00 CET +01:00,
 Fri, 07 Nov 2014 15:49:00 CET +01:00,
 Sat, 08 Nov 2014 15:49:00 CET +01:00,
 Sun, 09 Nov 2014 15:49:00 CET +01:00,
 Mon, 10 Nov 2014 15:49:00 CET +01:00,
 Wed, 05 Nov 2014 17:49:00 CET +01:00]
 </code></pre>
<p>Finally, let&rsquo;s compare the first event to the last one, just to confirm that something is wrong:</p>
<pre><code>
 events.first &lt;=&gt; events.last
 # =&gt; 1
 </code></pre>
<p>It was supposed to return 0, because they refer to the same record on the database.</p>
<p>So, the first problem is that the old implementation was too naive: it doesn&rsquo;t check the records ID equality before comparing their dates. The first record has the same ID of the last one but its date is different, so the starship method &lt;=&gt; doesn&rsquo;t return 0 as it should. Lack of complete rspec tests allowed for this bug to happen. </p>
<p>But there&rsquo;s more: even though ActiveRecord::Base doesn&rsquo;t include the Comparable module, it definitely has the <code>&lt;=&gt;</code> method:</p>
<pre><code>
ActiveRecord::Base.methods.include? :&lt;=&gt;
# =&gt; true
ActiveRecord::Base.ancestors.include? Comparable<br />
# =&gt; false</code></pre>
<p>Here&rsquo;s the default ActiveRecord implementation:</p>
<pre><code><br />module ActiveRecord
  module Core
    def &lt;=&gt;(other_object)
      if other_object.is_a?(self.class)
        self.to_key &lt;=&gt; other_object.to_key
      else
        super
      end
    end
  end
end
</code></pre>
<p>The main issues were:</p>
<ul><li>The new implementation of the &lt;=&gt; method was not completely tested, leaving out one of the most important cases (record equality)</li>
<li>Comparable module was added but was useless</li>
<li>There&rsquo;s always a good explaination behind odd behaviors, it&rsquo;s just a matter of taking the time to find it out. </li>
</ul><p>By the way my final implementation was:</p>
<pre><code>
def &lt;=&gt; (other)
  unless super.zero? # they are not the same record
    if calendar == other.calendar and calendar.present?
      date &lt;=&gt; other.date
    end
  end
end<br /><br /></code></pre>