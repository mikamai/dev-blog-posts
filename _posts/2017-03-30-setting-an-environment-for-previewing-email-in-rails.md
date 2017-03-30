---
ID: 1263
post_title: >
  Setting an environment for previewing
  email in Rails
author: Matteo Revelli
post_date: 2017-03-30 15:02:23
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/03/30/setting-an-environment-for-previewing-email-in-rails/
published: true
---
Developing email templates to be sent by our Rails application may sometimes be a long and tedious operation because under development you must continuously send to yourself the email in order to test the changes.

If you don’t want to waste your time, Rails provides a very helpful tool for those who need to build the layout for the emails of our application: <em>ActionMailer::Preview</em>. In fact, with this tool (available from Rails 4.1), you are able to make the email preview, just as it happens when you work with the Rails views.

The next step is to configure Letter Opener Web. This gem allows us to simulate sending a mail through our Rails application. In this way, we can reduce the risk of sending our test email to some address while we are working on it.<!--more-->

Let’s start by setting the environment in order to be able to preview our email with ActionMailer. To do this, we can generate a Mailer which sends a welcome email to users who sign up on our website.

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
# /app/models/user.rb
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

Afterwards it is time to edit our email template by showing the username that our application has just recorded.
```
# /app/views/user_mailer/welcome_email.html.erb
&lt;h1&gt;Welcome &lt;%= @user.name %&gt;&lt;/h1&gt;
&lt;p&gt;
  This is a welcome email!
&lt;/p&gt;
```

Let’s setup the email preview. The command we have just launched, has generated the Mailer and created this file: <code><span style="color: #999999;">/test/mailers/previews/</span>user_mailer_preview.rb</code>

So, edit this file in order to display the preview of our email.
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
Now we can install Letter Opener Web to simulate sending emails. Add this gem to Gemfile and run `bundle install`:
```
group :development do
  gem &#039;letter_opener_web&#039;
end
```

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
Once we have signed up, the application delivers you a welcome email which can be seen on this url: `http://localhost:3000/letter_opener`

<img class="aligncenter wp-image-1280 size-large" src="https://dev.mikamai.com/wp-content/uploads/2017/03/letter_opener-1024x525.png" alt="" width="640" height="328" />

The development environment is ready to be used. From now on, you will notice that working on mail templates will be much faster!

View <em>Letter_Opener_Web</em> on <a href="https://github.com/fgrehm/letter_opener_web">GitHub</a>