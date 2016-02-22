---
layout: post
title: Omniauth Facebook With Devise
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
Initializing a User Using Omniauth-Facebook with Devise


Instead of putting the burden on yourself and your team for implementing an authentication feature for your app, many programmers are deflecting the responsibility to Tech Gaints such as Facebook, Google, and Twitter to securely log in their users. Besides saving your team from writing code, the major benefit of using another authentication provider is that your app becomes considerably more robust against security vulnerabilities.

Omniauth is the miraculous gem that allows you to use alternative authentication providers in your app. However, for the purpose of this blog, I’m going to demonstrate how you can use Omniauth with Devise, a popular gem used for user authentication.

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

This will create a migration file for your User model with all the predefined attributes that ship with Devise. You can view all of these attributes by going to `db/migrate/20160215180931_devise_create_users.rb` (your file will have different numbers).

Before you run `rake db:migrate` you’ll need to create a new table to add two more attributes to the User model so you’ll be able to create User instances from data returned to you from Facebook.

`rails g migration AddOmniauthToUsers provider:string uid:string`

Now you can run `rake db:migrate`

Earlier when you ran the two commands to install Devise into your rails application you created several files. One of those files was `config/initializers/devise.rb`. Head there so you can declare the authentication provider you’re using (Facebook) and declare your Facebook “APP_ID” and “APP_SECRET.”

Include in config/initializers/devise.rb:

config.omniauth :facebook, "APP_ID", "APP_SECRET"

To get an ‘APP_ID’ and ‘APP_SECRET’  you’ll have to create an app at the Facebook Developers website https://developers.facebook.com/. If this is your first time creating a Facebook app then follow this guide: https://developers.facebook.com/docs/apps/register

In order for Devise to be aware that Omniauth exists in your application you’ll need to declare it. Go to your user model and input the following code:

devise :omniauthable, :omniauth providers => [:facebook]

This code will create the following routes or url methods:

  user_omniauth_authorize_path(proivider)
user_omniauth_callback_path(proivider)

To activate the user_omniauth_authrorize_path you’ll need to display a link in your application. In the view where you would like your user to have the option to sign in through Facebook, input the following link:

<%= link_to “Sign in with Facebook”, user_omniauth_authorize_path(:facebook) %>

Although you won’t be explicitly calling the user_omniauth_callback_path, it’s essential to have in your app. After a user signs in through Facebook they are redirected to this url where you’ll have access to their data. To declare this callback url you’ll have to update your ‘config/routes.rb file:

devise_for :users, :controllers => { :omniauth_callbacks => “users/omniauth_callbacks” }

We’ve specified our url methods that we’ll need to use for our Facebook authentication so now we’ll need to create a controller. Label the controller file “app/controllers/users/omniauth_callbacks_controller.rb” and set up the file like so:

Copy users omniauth section

On line 4 inside the ‘facebook’ method you’ll notice we’re calling the ‘from_omniauth’ method on the User class. To utilize this method we’ll have to define it, so head over to app/models/user.rb and update your file with the following code:

From omniauth method

Inside this method we can extract data that is returned to us from Facebook and initialize a new User. The argument of this method (auth) is assigned:

{"provider"=>"facebook",
 "uid"=>"10207528486042900",
 "info"=>{"name"=>"Frank Nowinski", "image"=>"http://graph.facebook.com/10207528486042900/picture"},
 "credentials"=>
  {"token"=>
    "CAAMOIT5sGIcBALLCG7IZCcruM7RppvzNQWZClaC9fDtkgiYL1U0QlQVCfe7Euw1yPErZBq0j78osqaPrTZAHIkbGCl7bfiQLGRxLyfo2EI2v23jt4LNB1yCKjFQTA6z5ZC3c91l1kkHAEJU2J8PLj8ZAjF5WBNqC5d3oeEneZABLvxiCoUFFgZBhmMwwC6C0NfCwXVZAOwBJtAgZDZD",
   "expires_at"=>1461190586,
   "expires"=>true},
 "extra"=>{"raw_info"=>{"name"=>"Frank Nowinski", "id"=>"10207528486042900"}}}
