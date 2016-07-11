---
layout: post
title: Formatting the URL for the Yahoo Finance API
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
date: 2016-01-29T04:17:17-05:00
---
On a mission to create a ruby gem to return realtime stock data to the user, I was confronted with a dilemma: how can I attain stock data from the internet and displaying it in a formatted table within my gem? With the help of Yahoo! Finances incredibly convenient API, this process was made exceptionally easy. If used correctly, this API would return realtime stock data within a csv file—all I had to do was properly assemble the URL.

To accomplish this I needed to input information into two positions of the base URL: the stock tickers position and the tags position. The stock tickers position (represented as <code>{STOCKS}</code> in the below URL) contains the stock tickers that you want to look up joined together by an addition sign (<code>+</code>). So, if the user wanted to look up Apple's and Google's stock, the URL in the stock tickers position would read: <code>AAPL+GOOG</code>.

> Base URL: "http://finance.yahoo.com/d/quotes.csv?s=<code>{STOCKS}</code>&amp;f=<code>{TAGS}</code>"

The second position that is required to be populated was the tags position (represented by <code>{TAGS}</code> in the above URL). Tags are just characters that symbolize different descriptions that, when included, return information relating to a stock; so, if you include an "n" tag it displays the stocks name, an "a" tag displays the stocks asking price, a "t8" tag displays the stocks 1 Year Target Price (you get the idea). Continuing with our example, if the user wanted to look up the name, earnings per share, and the 52 week high for Apple's and Google's stock, we would include the "n", "e", and "k" tags in the tags position of the base URL. However, when we add the tags to the URL we concatenate them together, so the end result would be "nek."

Below is the final URL of the aforementioned example that will return a csv file that contains the specific information that we requested.

> http://finance.yahoo.com/d/quotes.csv?s=AAPL+GOOG&amp;f=nek
