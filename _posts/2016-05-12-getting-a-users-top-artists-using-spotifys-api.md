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
Spotify’s API has a whole host of endpoints that allow programmers to extract data; you can access everything from artists and their top songs to information about the users themselves. One endpoint I found particularly intriguing was the <em>User’s Top Artists and Tracks</em> endpoint where (as the title implies) Spotify collects data of a users most listened to artists and tracks. This is when I got thinking - if Spotify aggregates your top artists, wouldn't it be cool if you could view the upcoming concerts of your favorite artists in your area? In this blog post I’m going to demonstrate how I used Spotify's API in conjunction with the BandsInTown API to output all of the nearby upcoming concerts of my favorite artists.

Before we get underway, let's add the <a href="https://github.com/guilhermesad/rspotify">RSpotify</a> gem into our app to help access the Spotify API as well as the <a href="https://github.com/taf2/curb">Curb</a> gem for formatting a get request (more on `Curb` later). Include the following gems in your Gemfile and run `bundle`:

{% highlight ruby linenos %}
`gem 'rspotify'`
`gem 'curb'`
{% endhighlight %}

If our end goal is to display upcoming concerts of our top artists on Spotify, our first priority is to obtain a users top artists. In order to access a users Spotify account you’ll need to obtain an access token, provided by Spotify, that gives the developer permission to receive user information. This can be accomplished by configuring the following two files. In `application.rb` add:

{% highlight ruby linenos %}
RSpotify::authenticate(ENV["SPOTIFY_CLIENT"], ENV["SPOTIFY_SECRET"])
{% endhighlight %}

Then, in `config/initializers/omniauth.rb` include:

{% highlight ruby linenos %}
require 'rspotify/oauth'

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :spotify, ENV["SPOTIFY_CLIENT"], ENV["SPOTIFY_SECRET"],
  scope: 'user-read-email user-top-read'
end
{% endhighlight %}

By including the `user-top-read` scope you're asking the user for permission to access their top artists from Spotify. If the user accepts when they are prompted at log in, you'll be granted an access token that you will later use in another request to the Spotify API. You can request additional user information by including other scopes found <a href="https://developer.spotify.com/web-api/using-scopes/">here</a>.

Next, provide a link so the user can log into their Spotify account:

{% highlight erb linenos %}
<%= link_to "Sign in with Spotify", "/auth/spotify" %>
{% endhighlight %}

Now, we’ll need to create a callback so we can manipulate the data that’s returned to us from Spotify. Head to `routes.rb` and declare what controller and action you’ll like your callback to hit:

{% highlight ruby linenos %}
get "/auth/spotify/callback", to: "users/spotify"
{% endhighlight %}

You now have access to basic user information, such as name and email address, as well as the unique access token in the `spotify` action of your app.

Once you've retrieved the access token you're ready to make a subsequent request to the Spotify API for a users top artists. To facilitate formatting the endpoint I relied on the `curb` gem which we previously installed in our app.

The `curb` gem offers a straightforward way to append headers in your GET request. The following code will correctly format your request to meet Spotify’s guidelines for the <em>Users Top Artist</em> endpoint:

{% highlight ruby linenos %}
http = Curl.get("https://api.spotify.com/v1/me/top/artists?limit=25") do |http|
  http.headers['Accept'] = 'application/json',
  http.headers['Authorization'] = "Bearer #{access_token}"
end.body_str

JSON.parse(http)
{% endhighlight %}

You can request a minimum of one or a maximum of fifty artists by modifying the `limit` query parameter at the conclusion of the endpoint (ex: `limit=50`). Finally, replace `#{access_token}` with the actual access token, pass in the http local variable as an argument to the `JSON.parse` method and you'll receive your top artists!


<h2>Finding local concerts with the BandsInTown API</h2>

The BandsInTown API endpoint requires the following five parameters:

<ol>
  <li>artist_name</li>
  <li>location</li>
  <li>app_id</li>
  <li>api_version</li>
  <li>format</li>
</ol>

Below is the BandsInTown URL we’re aiming to construct:

{% highlight ruby linenos %}
"http://api.bandsintown.com/artists/#{artist_name}/events/recommended?location=#{location}&app_id=#{app_id}&api_version=#{api_version}&format=#{format}"
{% endhighlight %}

Now let’s populate these parameters.

<ol>
  <li>Input the name of the artist in the `artist_name` parameter. Feel free to use spaces to separate words since we’ll be encoding the url anyways. For this demonstration, let’s look up the artist `Kanye West`.</li>
  <li>To get upcoming concerts in an area nearby your current location, insert `use_geoip&radius=50` in the `location` parameter. `use_geoip` uses the ip address of the computer you’re currently on to find your location. If you’d like to broaden your search of nearby concerts, increase the value of the radius parameter (maximum value is 150).</li>
  <li>Your app_id could be anything you wish, although it’s best practice to use a word, or words, that reflect the app you’re building. For the purpose of this tutorial, my app_id will be `discover-shows`.</li>
  <li>There are two versions of the BandsInTown API: 1.0 and 2.0. We’ll use 2.0 because it’s more current.</li>
  <li>Finally, fill in the format parameter with ‘json’ so our data will be formatted in json when it is returned to us.</li>
</ol>

So, now our url should look like this:

{% highlight ruby linenos %}
"http://api.bandsintown.com/artists/Kanye West/events/recommended?location=use_geoip&radius=50&app_id=discover-shows&api_version=2.0&format=json"
{% endhighlight %}

See that space between Kanye and West? Yeah, that’s no good, and it will generate an error if we submit this to the BandsInTown API. To prevent this, it’s as easy as passing our URL into the URI.escape method as an argument. The URI.escape method will replace any whitespace characters with `%20`, resulting in a valid URL.

{% highlight ruby linenos %}
escaped_url = URI.escape("http://api.bandsintown.com/artists/#{self.name}/events/recommended?location=use_geoip&radius=50&app_id=discover-shows&api_version=2.0&format=json")
{% endhighlight %}

Next, we’ll convert this URL into an URI object by passing it into the URI.parse method.

{% highlight ruby linenos %}
escaped_url = URI.escape("http://api.bandsintown.com/artists/#{self.name}/events/recommended?location=use_geoip&radius=50&app_id=discover-shows&api_version=2.0&format=json")
uri = URI.parse(escaped_url)
{% endhighlight %}

Now, we’re ready to send a get request to the BandsInTown API. We can accomplish this by passing the `uri` variable into the `get_response` method of the `Net::HTTP` class:  

{% highlight ruby linenos %}
response = Net::HTTP.get_response(uri).body
{% endhighlight %}

Chaining `body` to the end of the `get_response` method returns the body of the response, or the data in which we are looking for. The response is a representation of Kanye West’s upcoming concerts in JSON format. All that’s left to do is to parse the response!

{% highlight ruby linenos %}
JSON.parse(response)
{% endhighlight %}

Let’s refactor our code and encapsulate it inside an method called `events`:

{% highlight ruby linenos %}
  def events
    escaped_uri = URI.escape("http://api.bandsintown.com/artists/#{self.name}/events/recommended?location=use_geoip&radius=50&app_id=discover-shows&api_version=2.0&format=json")
    uri = URI.parse(escaped_uri)
    JSON.parse(Net::HTTP.get_response(uri).body)
  end
{% endhighlight %}


View all of the other endpoints Spotify's API has to offer <a href="https://developer.spotify.com/web-api/endpoint-reference/">here</a>.
