---
ID: 516
post_title: >
  How requirements shaped my code, AKA
  Rails 5 and ActiveRecord before_destroy
  callbacks
author: Andrea Longhi
post_date: 2017-01-12 13:54:32
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/01/12/how-requirements-shaped-my-code-aka-rails-5-and/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/155762886624/how-requirements-shaped-my-code-aka-rails-5-and
tumblr_mikamayhem_id:
  - "155762886624"
---
I recently started a new project with Rails 5, and at some point I found out the code was not behaving as expected.
The initial scenario was quite simple, Admins can have many Phones, as this code suggests:

<!--more--> 

<pre><code>
class Admin  &lt; ApplicationRecord
  has_many :phones
end

class Phone &lt; ApplicationRecord
  belongs_to :admin
end
</code></pre>
<h3>New requirements add new issues</h3>
Then application requirements changed as follows:

<cite>“Admins can add/remove phones, but are not allowed to destroy the last remaining one.”</cite>

After reading the requirements, my first guess was to use a <code>before_destroy</code> callback in the Phone model. If the callback returns false, the destroy operation should be halted:
<pre><code>
class Phone  &lt; ApplicationRecord
  belongs_to :admin

  before_destroy :dont_destroy_the_last_admin_phone

  private

  def dont_destroy_the_last_admin_phone
    return false if admin.phones.count &lt; 2
  end
end
</code></pre>
But much to my surprise this doesn’t work anymore. There’s not much information on this ActiveRecord new feature on the web, but according to the <a href="http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html">current documentation</a> the behavior has changed:

<cite>“If the before_validation callback throws :abort, the process will be aborted and ActiveRecord::Base#save will return false. If ActiveRecord::Base#save! is called it will raise an ActiveRecord::RecordInvalid exception. Nothing will be appended to the errors object.”</cite>

So, the working implementation became as follows:
<pre><code>
class Phone  &lt; ApplicationRecord
  belongs_to :admin

  before_destroy :dont_destroy_the_last_admin_phone

  private

  def dont_destroy_the_last_admin_phone
    throw(:abort) if admin.phones.count &lt; 2
  end
end
</code></pre>
This is the first time in 10 years using Ruby that I see real usage for a <code>throw</code> statement, so my face is still a bit frowned.
But the reason behind this change is very sound: there are situations where a callback may return false but the intended behavior is not to halt the events chain. In the past if you experienced that issue you needed to explicitly return <code>true</code> in the callback method, which was a little quirky.
<h3>Upgrading old applications</h3>
So what happens if you update your old rails 4 app to rails 5? That should not be a problem, since this new behavior is active by default only on freshly new created apps, see <code>config/initializers/new_framework_defaults.rb</code> file:
<pre><code>
# Do not halt callback chains when a callback returns false. Previous versions had true.
ActiveSupport.halt_callback_chains_on_return_false = false
</code></pre>
<h3>Final implementation</h3>
By the way, my final implementation was completely different. I was not much satisfied with my callback code, because I felt the behavior was not very visible within the <code>before_destroy</code> callback, and, as I wrote before, that <code>throw :abort</code> still makes me frown ;-)

The requirement could be stated as follows, which drives a totally different and clearer implementation, in my opinion:

<cite>“Admins must always have at least one phone”</cite>

So my final code was:
<pre><code>
class Admin  &lt; ApplicationRecord
  has_many :phones

  validates :phones, presence: true
end

class Phone &lt; ApplicationRecord
  belongs_to :admin
end
</code></pre>
Much better, isn’t it?

There is one slight difference with the previous implementation here: now you can actually destroy a Phone record even if it’s the last one for the given admin, for example via rails console. The enforcement is only at the Admin model level. This is not real a problem in the application, since in the web interface you always access phones in the context of the Admin.
<h3>TL;DR</h3>
My giveaways with this article are two:
<ul>
 	<li>be aware of the recent changes with ActiveRecord callbacks</li>
 	<li>the requirements wording may be as strong as to drive your actual implementation, at least for me</li>
</ul>