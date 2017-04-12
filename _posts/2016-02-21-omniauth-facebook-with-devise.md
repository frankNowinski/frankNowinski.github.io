---
layout: post
title: Omniauth-Facebook With Devise
modified:
categories:
description:
tags: []
image:
feature:
credit:
creditlink:
comments:
share:
date: 2016-02-21T19:25:01-05:00
---
Instead of putting the burden on yourself and your team for implementing an authentication feature for your app, many programmers are deflecting the responsibility to tech giants such as Facebook, Google, and Twitter to securely log in their users. Besides saving your team from writing code, the major benefit of using another authentication provider is that your app becomes considerably more robust against security vulnerabilities.

Omniauth is the miraculous gem that allows you to use alternative authentication providers in your app. For the purpose of this blog I’m going to demonstrate how you can use [Omniauth-Facebook](https://github.com/mkdynamic/omniauth-facebook) with [Devise](https://github.com/plataformatec/devise), a popular gem used for user authentication.

First, you’ll need to include the following gems in your <code>Gemfile</code>:

{% highlight ruby linenos %}
gem 'devise'
gem 'omniauth-facebook'
{% endhighlight %}

Now run <code>bundle install</code>

Next, create your User model with Devise by entering the following two commands in your terminal:

{% highlight ruby linenos %}
rails generate devise:install
rails g devise User
{% endhighlight %}

This will create a migration file for your User model with all the predefined fields that ship with Devise. You can view all of these attributes by going to <code>db/migrate/20160215180931_devise_create_users.rb</code> (your file name will have different numbers).

Before you run <code>rake db:migrate</code> you’ll need to add two more fields to the User table so you’ll be able to create users with data returned to you from Facebook.

{% highlight ruby linenos %}
rails g migration AddOmniauthToUsers provider:string uid:string
{% endhighlight %}

Now you can run <code>rake db:migrate</code>

Earlier when you installed Devise you created several files for your rails application, and one of those files happens to be <code>config/initializers/devise.rb</code>. Go to that file to declare Facebook as your authentication provider and fill in your Facebook <code>APP_ID</code> and <code>APP_SECRET</code>:

{% highlight ruby linenos %}
config.omniauth :facebook, "APP_ID", "APP_SECRET"
{% endhighlight %}

To get an <code>APP_ID</code> and <code>APP_SECRET</code> you’ll have to create an app at the [Facebook Developers website](https://developers.facebook.com/). If this is your first time creating a Facebook app then follow this [guide](https://developers.facebook.com/docs/apps/register).

In order for Devise to be aware that Omniauth exists in your application you’ll need to let your User model know. Include the following code in <code>app/models/user.rb</code>:

{% highlight ruby linenos %}
devise :omniauthable, :omniauth providers => [:facebook]
{% endhighlight %}

The above code creates these two routes or url methods:
<code>user_omniauth_authorize_path(proivider)</code>
<code>user_omniauth_callback_path(proivider)</code>

To activate the <code>user_omniauth_authrorize_path</code> you’ll need to display a link in your application. Input the following link in the view where you would like your user to have the option to sign in through Facebook:

{% highlight erb linenos %}
<%= link_to “Sign in with Facebook”, user_omniauth_authorize_path(:facebook) %>
{% endhighlight %}

Although you won’t be explicitly calling <code>user_omniauth_callback_path</code> in your views, it’s essential to have in your app. After a user signs in through Facebook they are redirected to this callback url within your application where you’ll have access to the data you received from Facebook. To declare this callback url update your <code>config/routes.rb</code> file:

{% highlight ruby linenos %}
devise_for :users, :controllers => { :omniauth_callbacks => “users/omniauth_callbacks” }
{% endhighlight %}

Now we’ll need to create a controller so we can create users with Facebook's authentication and sign them in to our app. Label the controller file <code>app/controllers/users/omniauth_callbacks_controller.rb</code> and set up the file like so:

{% highlight ruby linenos %}
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def facebook
    @user = User.from_omniauth(request.env["omniauth.auth"])

    @user.save if !@user
    sign_in_and_redirect @user, :event => :authentication
    set_flash_message(:notice, :success, :kind => "Facebook") if is_navigational_format?
  end

  def failure
    redirect_to root_path
  end
end
{% endhighlight %}

On line 3 inside the <code>facebook</code> method you’ll notice we’re calling the <code>from_omniauth</code> method on the User class. To utilize this method we’ll have to define it, so head over to <code>app/models/user.rb</code> and update your file with the following code:

{% highlight ruby linenos %}
def self.from_omniauth(auth)
  where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
    user.name = auth.info.name
    user.password = Devise.friendly_token[0,20]
  end
end
{% endhighlight %}

Inside this method we use the data that is provided by Facebook to either find a user or create one. The <code>first_or_create</code> method is pretty self-explanatory: it returns the user if they exist in the database or it creates a new user. It's important to note that when initializing a new user the <code>first_or_create</code> method automatically sets the <code>uid</code> and <code>provider</code> fields with their appropriate data.

To create a new user for your app you will need to include at minimum the <code>password</code> field inside the <code>first_or_create</code> loop (remember the <code>uid</code> is provided for you by default). However, the programmer is free to use whatever data Facebook provides to incorporate into their user class. For instance, you can extract the users Facebook's profile picture to use in your application by adding <code>user.image = auth.info.image</code> inside the <code>first_or_create</code> loop.

In a nutshell, that's how to use Omniauth-Facebook in conjunction with Devise to allow your users to sign in securely to your website.
