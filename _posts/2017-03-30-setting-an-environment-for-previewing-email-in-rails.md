---
ID: 1263
post_title: >
  Previewing email in Rails with
  ActionMailer::Preview and Letter Opener
author: Matteo Revelli
post_date: 2017-03-30 15:02:23
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/03/30/setting-an-environment-for-previewing-email-in-rails/
published: true
---
Developing email templates to be sent by our Rails application may sometimes be a long and tedious operation because under development you must continuously send to yourself the email in order to test the changes.

If you don’t want to waste your time, Rails provides a very helpful tool for those who need to build the layout for the emails of our application: <em>ActionMailer::Preview</em>. In fact, with this tool (available from Rails 4.1), you are able to generate the Rails mailer view without the need to actually send the email.
<!--more-->

Let’s start by setting up the email template and controller action with ActionMailer. We will generate a Mailer which sends a welcome email to users who sign up on our website.

First of all, generate the Mailer:

```
rails generate mailer UserMailer welcome_email
```

This will also produce the view, called <code><span style="color: #999999;">app/views/user_mailer/</span>welcome_email.html.erb</code>

Great! Let’s define the method that allows us to send the welcome email:

```
# /app/mailers/user_mailer.rb
class UserMailer &lt; ApplicationMailer
  def welcome_email(user)
    @user = user
    mail to: user.email, subject: &quot;Welcome&quot;
  end
end
```
```
# /app/models/user.rb
def send_welcome_email
  UserMailer.welcome_email(self).deliver_now
end
```
Now we have to tell the user_controller to deliver the welcome email when a user signs up successfully:
```
# /app/controllers/users_controller.rb
def create
  @user = User.new(user_params)
  ...
  if @user.save
    ...
    @user.send_welcome_email
  end
end
```

Now we want our email template to show the username that our application has just recorded.
```
# /app/views/user_mailer/welcome_email.html.erb
&lt;h1&gt;Welcome &lt;%= @user.name %&gt;&lt;/h1&gt;
&lt;p&gt;
  This is a welcome email!
&lt;/p&gt;
```

Now, what if we wanted to see our welcome email in our browser to check that it looks as expected? There's where <em>ActionMailer::Preview</em> comes in handy. Actually the Rails generator has already done most of the work for us. The command we launched at the beginning not only generated the Mailer but it created also this file: <code><span style="color: #999999;">/test/mailers/previews/</span>user_mailer_preview.rb</code> (note: if you have <code>rspec</code> bundled in your project the file will be created under <code>/spec</code>)

The file already contains the code needed for the preview to work:
```
# /test/mailers/previews/user_mailer_preview.rb

# Preview all emails at http://localhost:3000/rails/mailers/user_mailer
class UserMailerPreview &lt; ActionMailer::Preview

# Preview this email at http://localhost:3000/rails/mailers/user_mailer/welcome_email
  def welcome_email
    UserMailer.welcome_email
  end
end
```

All we have to do is edit it, since we have modified <code>welcome_email</code> to accept the username as parameter:
```
# /test/mailers/previews/user_mailer_preview.rb

# Preview all emails at http://localhost:3000/rails/mailers/user_mailer
class UserMailerPreview &lt; ActionMailer::Preview

# Preview this email at http://localhost:3000/rails/mailers/user_mailer/welcome_email
  def welcome_email
    UserMailer.welcome_email(User.last)
  end
end
```
Et voila! As the comment in the file says, the email preview is now visible at <code>http://localhost:3000/rails/mailers/user_mailer/welcome_email</code> when the <code>rails server</code> is up and running.

Convenient as this approach is, it still requires us to take care of the <code>test/mailer/previews</code> file in addition to the mailer proper: we have to modify it and keep it up to date to reflect changes in our mailer methods. A different approach which avoids this extra work is using the Letter Opener gem. This gem reroutes all mail sent through our Rails application to be displayed in the browser instead. This is also very useful in the staging environment, if for some reason you have to use production data, because it avoids the risk of sending out test email to production addresses while testing.<!--more-->

The gem is added to the gemfile the usual way:
```
group :development do
  gem &#039;letter_opener_web&#039;
end
```
and then you run `bundle install`

In our routes:
```
if Rails.env.development?
  mount LetterOpenerWeb::Engine, at: &quot;/letter_opener&quot;
end
```
Configure delivery method on your <span style="color: #999999;">`config/environments/`</span>`development.rb`:
```
config.action_mailer.delivery_method = :letter_opener_web
```
At this point we can do the action that would trigger the email notification (in our case signing up to the application) and the welcome email is generated and 'sent' to the letter opener mailbox. All email generated by the application can now be seen at this url: `http://localhost:3000/letter_opener`

<img class="aligncenter wp-image-1280 size-large" src="https://dev.mikamai.com/wp-content/uploads/2017/03/letter_opener-1024x525.png" alt="" width="640" height="328" />

The development environment is ready to be used. From now on, you will notice that working on mail templates will be much faster!

View <em>Letter_Opener_Web</em> on <a href="https://github.com/fgrehm/letter_opener_web">GitHub</a>