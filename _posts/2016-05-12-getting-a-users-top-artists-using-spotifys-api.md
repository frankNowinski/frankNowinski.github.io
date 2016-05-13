---
layout: post
title: Getting a Users Top Artists Using Spotify's API
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
date: 2016-05-12T22:02:58-04:00
---
Spotify’s API has a whole host of endpoints that allow programmers to extract data; you can access everything between artists and their top songs to information about the users themselves. One endpoint I found particularly intriguing was the “User’s Top Artists and Tracks” endpoint where (as the title implies) Spotify collects data of a users most listened to artists and tracks. In this blog post I’m going to demonstrate how to retrieve data from this endpoint so you can utilize it in your rails application.

To integrate Spotify’s API into your app, add `gem 'rspotify'` to your Gemfile and run `bundle`.

In order to access a users Spotify account we’ll need to obtain an access token which can be accomplished by configuring the following two files. In `application.rb` add:

RSpotify::authenticate(ENV["SPOTIFY_CLIENT"], ENV["SPOTIFY_SECRET"])

Then, in `config/initializers/omniauth.rb` include:

{% highlight ruby linenos %}
require 'rspotify/oauth'

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :spotify, ENV["SPOTIFY_CLIENT"], ENV["SPOTIFY_SECRET"], scope: 'user-read-email user-top-read'
end
{% endhighlight %}

By including the `user-top-read` scope, you’ll receive an access token which authorizes your application to access a users top artists. You can access additional user information by including other scopes found here.

Create a link in your app so the user can log in to their Spotify account and give you permission to access their top artists:

{% highlight erb linenos %}
  <%= link_to ‘Sign in with Spotify’, ‘/auth/spotify’ %>
{% endhighlight %}

Now we’ll need to create a callback so we can manipulate the data that’s returned to us. Head to your `routes.rb` file and declare what controller and action you’ll like your callback to hit:

{% highlight erb linenos %}
  get ‘/auth/spotify/callback’, to: ‘users/spotify’
{% endhighlight %}

You can now access basic user information such as name and email address as well as a unique access token in the spotify action.

After you retrieve the access token you’re ready to make a request to the Spotify API for a users top artists. To facilitate formatting the URL, I relied on the `curb` gem. To install `curb` into your application simply include `gem 'curb'` in your `Gemfile` and run `bundle`.
The `curb` gem offers a straightforward and readable way to append headers in your GET request. The following code will correctly format your url to meet Spotify’s API guidelines:

{% highlight erb linenos %}
  http = Curl.get("https://api.spotify.com/v1/me/top/artists?limit=25") do |http|
    http.headers['Accept'] = 'application/json',
    http.headers['Authorization'] = "Bearer #{token}"
  end.body_str

  JSON.parse(http)
{% endhighlight %}

You can request a minimum of one or a maximum of fifty artists by modifying the `limit` query parameter at the end of the endpoint (ex: `limit=50`). Finally, replace `#{access_token}` with the actual access token and you’ll have a list of the users top 25 artists.