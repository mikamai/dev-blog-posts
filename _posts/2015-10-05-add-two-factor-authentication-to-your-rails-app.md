---
ID: 79
post_title: >
  Add Two Factor Authentication to your
  Rails app
author: Nicola Racco
post_date: 2015-10-05 13:06:28
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/10/05/add-two-factor-authentication-to-your-rails-app/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/130546908414/add-two-factor-authentication-to-your-rails-app
tumblr_mikamayhem_id:
  - "130546908414"
---
Recently we started adding two-factor auth to all our apps by default. And obviously there is a gem for this: [devise-two-factor](https://github.com/tinfoil/devise-two-factor/)

In this article I guide you through the setup of two-factor authentication using this gem.<!--more--> If the article looks dauntingly long, don't fear: there's a [demo](https://github.com/mikamai/rails-2fact-auth-example) :)

### Setup

This tutorial builds on an existing rails application using devise for authentication, so please follow the devise readme before continuing. We're going to use `AdminUser` model, of course you can use whatever model name you prefer, for example you may want to stick to a more general `User` model.

You can find the complete final code for this article on [github](https://github.com/mikamai/rails-2fact-auth-example).

Add to your `Gemfile`:

```ruby
gem &#039;devise-two-factor&#039; # for two factor
gem &#039;rqrcode_png&#039; # for qr codes
```

Then, run `bundle` to install the gems.

Now we need to tell the user model to use the two-factor authentication, and we need also to add some database columns to store the secret code for authenticating the [OTP](https://en.wikipedia.org/wiki/One-time_password) password. The gem offers a generator in order to generate these colums. Run `rails generate devise_two_factor AdminUser TWO_FACTOR_SECRET_KEY_NAME`, where `AdminUser` is the name of the model you wish to add two-factor auth to, and `TWO_FACTOR_SECRET` is the ENV variable name for your two factor encryption key (the variable must be a random sequence of characters, similar to the `SECRET_KEY_BASE` env variable). Remember, `git diff` is your friend when you need to check what generators did to your application code.

When the generation is complete you should see a new migration, like the following (`admin_users` will be replaced with your model table name):

```ruby
class AddDeviseTwoFactorToAdminUsers &lt; ActiveRecord::Migration
  def change
    add_column :admin_users, :encrypted_otp_secret,      :string
    add_column :admin_users, :encrypted_otp_secret_iv,   :string
    add_column :admin_users, :encrypted_otp_secret_salt, :string
    add_column :admin_users, :otp_required_for_login,    :boolean
    add_column :admin_users, :consumed_timestep,         :integer
  end
end
```

Edit the migration and add another column: this one will store a temporary `otp_secret` during the two-factor activation process (which we’ll see later):

```ruby
add_column :admin_users, :unconfirmed_otp_secret, :string
```

Now, check your model (`AdminUser` in my case): you should see that the devise configuration `database_authenticatable` has been replaced with `two_factor_authenticatable`:

```ruby
class AdminUser &lt; ActiveRecord::Base
  devise :rememberable, :trackable, :lockable,
         :session_limitable, :two_factor_authenticatable,
         :otp_secret_encryption_key =&gt; ENV[&#039;TWO_FACTOR_SECRET&#039;]
  # ...
end
```

Run `rake db:migrate` and the setup is complete.

### Authentication

If you haven’t done it already, run `rails generate devise:views` to copy all devise views inside your application.

Open `app/views/devise/sessions/new.html.erb` and add a new field called `otp_attempt`. You should obtain something like:

```html
&lt;h2&gt;Log in&lt;/h2&gt;

&lt;%= form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %&gt;
  &lt;div class=&quot;field&quot;&gt;
    &lt;%= f.label :email %&gt;&lt;br /&gt;
    &lt;%= f.email_field :email, autofocus: true %&gt;
  &lt;/div&gt;

  &lt;div class=&quot;field&quot;&gt;
    &lt;%= f.label :password %&gt;&lt;br /&gt;
    &lt;%= f.password_field :password, autocomplete: &quot;off&quot; %&gt;
  &lt;/div&gt;

  &lt;div class=&quot;field&quot;&gt;
    &lt;%= f.label :otp_attempt %&gt;&lt;br /&gt;
    &lt;%= f.text_field :otp_attempt, autocomplete: &quot;off&quot; %&gt;
  &lt;/div&gt;

  &lt;% if devise_mapping.rememberable? -%&gt;
    &lt;div class=&quot;field&quot;&gt;
      &lt;%= f.check_box :remember_me %&gt;
      &lt;%= f.label :remember_me %&gt;
    &lt;/div&gt;
  &lt;% end -%&gt;

  &lt;div class=&quot;actions&quot;&gt;
    &lt;%= f.submit &quot;Log in&quot; %&gt;
  &lt;/div&gt;
&lt;% end %&gt;

&lt;%= render &quot;devise/shared/links&quot; %&gt;
```

Then, open `app/controllers/application_controller.rb` to permit a new login parameter for our model:

```ruby
before_action :configure_permitted_parameters, if: :devise_controller?

protected

def configure_permitted_parameters
  devise_parameter_sanitizer.for(:sign_in) &lt;&lt; :otp_attempt
end
```

Ok, authentication is ready. But there is no way to activate the two factor auth, for now.

### Two Factor Activation

It's time for the controller and views. Add the following to your authentication model (`AdminUser` in my case), in order to ease the controller activate/deactivate actions:

```ruby
def activate_two_factor params
  otp_params = { otp_secret: unconfirmed_otp_secret }
  if !valid_password?(params[:password])
    errors.add :password, :invalid
    false
  elsif !validate_and_consume_otp!(params[:otp_attempt], otp_params)
    errors.add :otp_attempt, :invalid
    false
  else
    activate_two_factor!
  end
end

def deactivate_two_factor params
  if !valid_password?(params[:password])
    errors.add :password, :invalid
    false
  else
    self.otp_required_for_login = false
    self.otp_secret = nil
    save
  end
end

private

def activate_two_factor!
  self.otp_required_for_login = true
  self.otp_secret = current_admin_user.unconfirmed_otp_secret
  self.unconfirmed_otp_secret = nil
  save
end
```

So, when this method is called, `params` is required to contain the user password and the otp attempt. If they are both valid the method will activate the two factor authentication.

We’ll have the following routes:

```ruby
namespace :admin do
  get    &#039;/two_factor&#039; =&gt; &#039;two_factors#show&#039;, as: &#039;admin_two_factor&#039;
  post   &#039;/two_factor&#039; =&gt; &#039;two_factors#create&#039;
  delete &#039;/two_factor&#039; =&gt; &#039;two_factors#destroy&#039;
end
```

Now, the two factors controller (read the comments):

```ruby
class Admin::TwoFactorsController &lt; ApplicationController
  before_filter :authenticate_admin_user!

  def new
  end

  # If user has already enabled the two-factor auth, we generate a
  #   temp. otp_secret and show the &#039;new&#039; template.
  # Otherwise we show the &#039;show&#039; template which will allow the user to disable
  #   the two-factor auth
  def show
    unless current_admin_user.otp_required_for_login?
      current_admin_user.unconfirmed_otp_secret = AdminUser.generate_otp_secret
      current_admin_user.save!
      @qr = RQRCode::QRCode.new(two_factor_otp_url).to_img.resize(240, 240).to_data_url
      render &#039;new&#039;
    end
  end

  # AdminUser#activate_two_factor will return a boolean. When false is returned
  #   we presume the model has some errors.
  def create
    permitted_params = params.require(:admin_user).permit :password, :otp_attempt
    if current_admin_user.activate_two_factor permitted_params
      redirect_to root_path, notice: &quot;You have enabled Two Factor Auth&quot;
    else
      render &#039;new&#039;
    end
  end

  # If the provided password is correct, two-factor is disabled
  def destroy
    permitted_params = params.require(:admin_user).permit :password
    if current_admin_user.deactivate_two_factor permitted_params
      redirect_to root_path, notice: &quot;You have disabled Two Factor Auth&quot;
    else
      render &#039;show&#039;
    end
  end

  private

  # The url needed to generate the QRCode so it can be acquired by
  #   Google Authenticator
  def two_factor_otp_url
    &quot;otpauth://totp/%{app_id}?secret=%{secret}&amp;issuer=%{app}&quot; % {
      :secret =&gt; current_admin_user.unconfirmed_otp_secret,
      :app    =&gt; &quot;your-app&quot;,
      :app_id =&gt; &quot;YourApp&quot;
    }
  end
end
```

Finally, the views:

```html
&lt;!-- app/views/admin/two_factors/new.html.erb --&gt;

&lt;div class=&quot;page-header&quot;&gt;&lt;h2&gt;Enable Two Factor Auth&lt;/h2&gt;&lt;/div&gt;

&lt;p&gt;To enable &lt;em&gt;Two Factor Auth&lt;/em&gt;, scan the following QR Code:&lt;/p&gt;

&lt;p class=&quot;text-center&quot;&gt;&lt;%= image_tag @qr %&gt;&lt;/p&gt;

&lt;p&gt;Then, verify that the pairing was successful by entering your password and a code below.&lt;/p&gt;

&lt;%= form_for current_admin_user, url: [:admin, :two_factor], method: &#039;POST&#039; do |f| %&gt;
  &lt;div class=&quot;field&quot;&gt;
    &lt;%= f.label :password %&gt;&lt;br /&gt;
    &lt;%= f.password_field :password, autocomplete: &quot;off&quot; %&gt;
  &lt;/div&gt;

  &lt;div class=&quot;field&quot;&gt;
    &lt;%= f.label :otp_attempt %&gt;&lt;br /&gt;
    &lt;%= f.text_field :otp_attempt, autocomplete: &quot;off&quot; %&gt;
  &lt;/div&gt;

  &lt;div class=&quot;actions&quot;&gt;
    &lt;%= f.submit &quot;Enable&quot; %&gt;
  &lt;/div&gt;
&lt;% end %&gt;
```

```html
&lt;!-- app/views/admin/two_factors/show.html.erb --&gt;

&lt;div class=&quot;page-header&quot;&gt;&lt;h2&gt;Disable Two Factor Auth&lt;/h2&gt;&lt;/div&gt;

&lt;p&gt;Type your password to disable &lt;em&gt;Two Factor Auth&lt;/em&gt;&lt;/p&gt;

&lt;%= form_for current_admin_user, url: [:admin, :two_factor], method: &#039;DELETE&#039; do |f| %&gt;
  &lt;div class=&quot;field&quot;&gt;
    &lt;%= f.label :password %&gt;&lt;br /&gt;
    &lt;%= f.password_field :password, autocomplete: &quot;off&quot; %&gt;
  &lt;/div&gt;

  &lt;div class=&quot;actions&quot;&gt;
    &lt;%= f.submit &quot;Disable&quot; %&gt;
  &lt;/div&gt;
&lt;% end %&gt;
```

So, if a logged user visits `/admin/two_factor` and they have no two-factor auth enabled, they will see the `new` template. Filling the form will activate the two factor auth.

Once the user has two-factor auth enabled, visiting `/admin/two_factor` will render the `show` template. They can fill the form with their password to disable the two factor auth.

### Improvements

If you prefer, you can change the 'show' action to always render a template where a user can:

- activate **or reconfigure** the two factor
- disable the two factor if it’s enabled

You can also add the backup codes, using the [TwoFactorBackuppable](https://github.com/tinfoil/devise-two-factor#backup-codes) strategy.

### Issues

If you use Docker, you’ll surely encounter a problem. During the `assets:precompile` the app will try to connect to the DB. [I've opened an issue](https://github.com/tinfoil/devise-two-factor/issues/47) but haven’t received yet a reply until now.

A workaround I found is to ignore devise routes during the `assets:precompile`. Create the file `config/initializers/precompile.rb`:

```ruby
# used in config/routes.rb to ignore some routes. See &lt;a href=&quot;https://github.com/tinfoil/devise-two-factor/issues/47&quot;&gt;https://github.com/tinfoil/devise-two-factor/issues/47&lt;/a&gt; for details
module Precompile
  # Public: ignore the following block during rake assets:precompile
  def self.ignore
    unless ARGV.any? { |e| e == &#039;assets:precompile&#039; }
      yield
    else
      line = caller.first
      puts &quot;Ignoring line &#039;#{line}&#039; during precompile&quot;
    end
  end
end
```

And wrap your devise routes inside `Precompile.ignore` like in the following example:

```ruby
Precompile.ignore do
  devise_for :admin_users, path: &#039;admin/admin_users&#039;
end
```

This way when the routes file is parsed during the `assets:precompile` task, it will not load the `AdminUser` model that are generating a connection attempt on the DB.

**A/N: Today my work duties are covering me, and writing technical posts is not so easy as one could think. A special thanks to [Andrea Longhi](https://dev.mikamai.com/author/andrea-longhimikamai-com/) that reviewed and tested this post.**