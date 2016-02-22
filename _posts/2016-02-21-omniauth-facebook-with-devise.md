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

Omniauth is the miraculous gem that allows you to use alternative authentication providers in your app. However, for the purpose of this blog, I’m going to demonstrate how you can use Omniauth-Facebook with Devise, a popular gem used for user authentication.

First, you’ll need to include the following gems in your ‘Gemfile’:

{% highlight ruby linenos %}
gem 'devise'
gem 'omniauth-facebook'
{% endhighlight %}

Now run `bundle install`

Next, create your User model with Devise by entering the following two commands in your terminal:

{% highlight ruby linenos %}
rails generate devise:install
rails g devise User
{% endhighlight %}

This will create a migration file for your User model with all the predefined fields that ship with Devise. You can view all of these attributes by going to `db/migrate/20160215180931_devise_create_users.rb` (your file will have different numbers).

Before you run `rake db:migrate` you’ll need to create a new table to add two more attributes to the User model so you’ll be able to create user instances from data returned to you from Facebook.

`rails g migration AddOmniauthToUsers provider:string uid:string`

Now you can run `rake db:migrate`

Earlier when you ran the two commands to install Devise into your rails application you created several files. One of those files was `config/initializers/devise.rb`. Go to that file to declare the authentication provider you’re using (Facebook) and set your Facebook `APP_ID` and `APP_SECRET`:

{% highlight ruby linenos %}
config.omniauth :facebook, "APP_ID", "APP_SECRET"
{% endhighlight %}

To get an `APP_ID` and `APP_SECRET` you’ll have to create an app at the [Facebook Developers website](https://developers.facebook.com/)g. If this is your first time creating a Facebook app then follow this [guide](https://developers.facebook.com/docs/apps/register).

In order for Devise to be aware that Omniauth exists in your application you’ll need to let your User model know:

{% highlight ruby linenos %}
devise :omniauthable, :omniauth providers => [:facebook]
{% endhighlight %}

This will create the following routes or url methods:
`user_omniauth_authorize_path(proivider)`
`user_omniauth_callback_path(proivider)`

To activate the `user_omniauth_authrorize_path` you’ll need to display a link in your application. In the view where you would like your user to have the option to sign in through Facebook, input the following link:

{% highlight ruby linenos %}
<%= link_to “Sign in with Facebook”, user_omniauth_authorize_path(:facebook) %>
{% endhighlight %}

Although you won’t be explicitly calling `user_omniauth_callback_path` in your views, it’s essential to have in your app. After a user signs in through Facebook they are redirected to this url where you’ll have access to their data. To declare this callback url update your `config/routes.rb` file:

{% highlight ruby linenos %}
devise_for :users, :controllers => { :omniauth_callbacks => “users/omniauth_callbacks” }
{% endhighlight %}

Now we’ll need to create a controller where we can create our users with Facebook's authentication and sign them in. Label the controller file `app/controllers/users/omniauth_callbacks_controller.rb` and set up the file like so:

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

On line 3 inside the `facebook` method you’ll notice we’re calling the `from_omniauth` method on the User class. To utilize this method we’ll have to define it, so head over to `app/models/user.rb` and update your file with the following code:

{% highlight ruby linenos %}
def self.from_omniauth(auth)
  where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
   user.name = auth.info.name
   user.password = Devise.friendly_token[0,20]
  end
end
{% endhighlight %}

Inside this method we use the data that is provided by Facebook and create a new user. The `first_or_create` method is pretty self-explanatory: it returns the user if they exist in the database or it creates a new user. It's important to note that when initializing a new user the `first_or_create` method automatically sets the `uid` and `provider` fields with their appropriate data.

Besides the `uid` and `password`, the programmer is free to use whatever data that Facebook provides to incorporate it into their user. For instance, you can  extract the users Facebook's profile picture to use in your application by adding `user.image = auth.info.image` inside the `first_or_create` loop.

In a nutshell, that's how to use Omniauth-Facebook in conjunction with Devise to allow your users to sign in securely to your website.
