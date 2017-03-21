---
ID: 264
post_title: >
  Preview emails for any user with Rails
  4.1
author: Elia Schito
post_date: 2014-04-02 14:13:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/04/02/preview-emails-for-any-user-with-rails-41/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/81487222497/preview-emails-for-any-user-with-rails-41
tumblr_mikamayhem_id:
  - "81487222497"
---
<p>Last night I was preparing a weekly email to remind my dear <a href="https://timesampler.com">TimeSampler</a> users about the work they have done in their previous week.</p>

<p>I started by adding a couple of previews one for the user that has actual stuff done (see an example below):</p>

<p><img src="http://cl.ly/image/2w2B093j4644/Screen%20Shot%202014-04-02%20at%2011.33.31%20am.png" alt="Weekly email for user with registered bursts" /></p>

<h3>Setup a mailer (you can skip this)</h3>

<p>The first thing you need to preview an email is to have a Mailer (yuk!), here&rsquo;s a simple one:</p>
    <pre><code class="ruby"># app/mailers/user_notifier

class UserNotifier &lt; ActionMailer::Base
  default from: 'weekly-report@timesampler.com', reply_to: 'elia+timesampler-weekly-report@schito.me'
  helper DayHelper

  def weekly_report user
    @user         = user
    @project_days = project_days
    @days         = project_days.group_by(&amp;:day).to_a.sort
    @end_time     = end_time
    @start_time   = start_time

    mail to: user.email, subject: "TimeSampler: Weekly activity report · week #{start_time.cweek}"
  end
end
</code></pre>

<p>And its view:</p>
    <pre><code class="haml">/ app/views/user_notifier
= render 'style'

%h1 TimeSampler: Weekly activity report · week #{@start_time.cweek}

%p
  Hi #{@user.some_name}!

- if @project_days.any?
  %p
    Here's what you've done during the past week (#{link_to "from #{@start_time} to #{@end_time}", journal_path}):
  .days
    - @days.sort.reverse.each do |(day, project_days)|
      = render 'day', day: day, project_days: project_days
- else
  = render 'blank_slate'

= render 'footer'
</code></pre>

<h3>Setup a previewer</h3>

<p>Then we need to add our previewer. Problem is that the default location for the previewers is <code>test/mailers/previews</code><br />
and as you may have guessed already <a href="http://words.steveklabnik.com/rails-has-two-default-stacks">we have no <code>test</code> dir in this app</a>, instead we&rsquo;ll put our<br /><code>ActionMailer::Preview</code> classes inside <code>app/mailer_previews</code>.</p>

<p>To do that let&rsquo;s add the following line to <code>config/environments/development</code>:</p>
    <pre><code class="ruby">Rails.application.configure do
  # …
  config.action_mailer.preview_path = "#{Rails.root}/app/mailer_previews"
end
</code></pre>

<p>Your previewer could be something like this:</p>
    <pre><code class="ruby">class UserNotifierPreview &lt; ActionMailer::Preview
  def weekly_report_empty
    UserNotifier.weekly_report(andrea)
  end

  def weekly_report_full
    UserNotifier.weekly_report(elia)
  end

  # …s
end
</code></pre>

<p>And have beautyful previews for your emails. For example, this is how the blankslate <a href="https://timesampler.com">TimeSampler</a> mail looks like:</p>

<p><img src="http://cl.ly/image/0q0I1j251I43/Screen%20Shot%202014-04-02%20at%2012.57.20%20pm.png" alt="blank slate mail preview" /></p>

<h3>POWER PREVIEWING !!1!</h3>

<p>Now what if you want to preview emails for arbitrary users without having to manually update the code?</p>

<p>No problem, luckly it&rsquo;s Ruby! After a bit of inspection inside <a href="https://github.com/rails/rails/blob/4-1-stable/actionmailer/lib/action_mailer/preview.rb#L67-L69">actionmailer</a> and <a href="https://github.com/rails/rails/blob/4-1-stable/railties/lib/rails/mailers_controller.rb#L19-L21">railties</a> I found the right hook…</p>

<p><img src="http://cl.ly/image/0u3f1W2i1n2S/Screen%20Shot%202014-04-02%20at%201.08.00%20pm.png" alt="WARNING: rails hackery ahead" /></p>

<p>Rails asks the previewer if an <code>email_exists?</code> before trying to display it, so we&rsquo;ll hook into that method and<br />
auto-define a previewer instance method for each user in out system (please don&rsquo;t try this with more than a hundred users).</p>

<p>Here teh codez:</p>
    <pre><code class="ruby">class UserNotifierPreview &lt; ActionMailer::Preview
  def self.email_exists?(*)
    User.all.each do |user|
      next if user.email.blank?
      email = user.email.gsub(/W/, '_')
      method_name = "weekly_report_#{email}"
      user_id = user.id

      define_method method_name do
        UserNotifier.weekly_report(User.find(user_id))
      end
    end

    super
  end

  # …
end
</code></pre>

<p>Of course it&rsquo;s advisable to be careful and probably reduce the set of users you&rsquo;re gonna prepare a preview method for.<br />
In my case the development database is quite manageable and hence <code>User.all</code> is fine.</p>

<h3>Final Pro-tip</h3>

<p>You find cumbersome to dig the sources of the gems in your bundle even<br />
if <a href="https://manual.macromates.com/en/using_textmate_from_terminal.html">you setup the <code>$EDITOR</code> var in your bash</a> and are a regular client of <code>bundle open &lt;my-gem&gt;</code>?</p>

<p>Then you can give a try to the <a href="https://github.com/elia/bundler.tmbundle#readme"><strong>Bundler</strong> bundle</a> for <a href="http://macromates.com/download">TextMate2</a> which will give you an<br />
incredible speed boost while perusing the gems in your current <code>Gemfile.lock</code>.</p>

<p>To get some more info on my <a href="http://macromates.com/download">TextMate2</a> setup you can read:</p>

<h4><a href="http://dev.mikamai.com/post/77705936763/pimp-my-textmate2-ruby-edition">Read next: Pimp my TextMate2 — Ruby edition</a></h4>

<p><img src="https://f.cloud.github.com/assets/1051/190195/03670698-7ed5-11e2-983d-6da8b0d0dd7a.png" alt="opening gems from the current bundle" /></p>