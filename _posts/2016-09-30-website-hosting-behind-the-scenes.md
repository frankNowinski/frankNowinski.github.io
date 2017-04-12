---
layout: post
title: Behind the Scenes of Website Hosting
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
date: 2016-09-30T04:16:56-04:00
---
Ever wonder what a hosting service is doing being the scenes that allows your website to be viewed in the browser? If this question perplexes you, as it did me, read on.

You just spent the last month building an awesome app and now you want to show it off to the world. Before you do anything, you’ll need to obtain a domain name so users are able to visit your website. Owning a domain name is like owning an incredibly small piece of the internet, and with ownership usually comes a price tag. Buying your own custom domain name is going to cost you around $10 to $40. However, there’s an alternative solution: you can use a free domain name that’s often provided by the hosting service. Heroku, for one, provides free domain names but requires the url to be formatted like so: <code>[name of app].herokuapp.com</code>.

Underneath a domain name is just an IP address that identifies a specific computer. However, instead of inputting your IP address (ex: 90.47.136.226) into the address bar, we use domain names like www.github.com because who wants to write, much less remember, a 10 digit number?

When a user inputs a domain name into the address bar, how does the browser know what to show? This is where Domain Name System (DNS) or nameservers come in. The concept of DNS is pretty straightforward: DNS is responsible for routing your domain name to the correct server that is hosting your website. Therefore, whenever a user visits your domain name, it will ping the server that is correlated to that  name (or IP address) and load it's data.

Now that we have our domain name and the DNS is correctly routing visitors to our website, the last step (as you probably guessed) is hosting our website on a web hosting server. A web hosting server stores all of the HTML files, pictures, videos and other documents that your website is made up of. In order for these files to be displayed when a user visits them, they need to be stored in a folder on a computer that is connected to the internet. That is exactly what a web hosting service provides – a place to “serve” your website files.

Simple as that. Those are the three steps that must be taken in order to successfully host your website on the web.
