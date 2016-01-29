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
For the CLI Data Gem Project I paired with Roberto to create a program that would use the Yahoo! Finance API to return realtime stock data to the user.

Our main challenge was finding a way to retrieve realtime stock information and display it in our program in a formatted table. With the help of Yahoo! Finances incredibly convenient API, this process was made exceptionally easy. If used correctly, this API would return realtime stock data within a csv file—all we had to do was properly assemble the URL.

To accomplish this we would need to input information into two positions of the base URL: the stock tickers position and the tags position. The stock tickers position (represented as "{STOCKS}" in the below URL) contains the stock tickers that you want to look up joined together by an addition sign ("+"). So if the user wanted to look up Apple's and Google's stock, the URL in the stock tickers position would read: "AAPL+GOOG."

Base URL: "http://finance.yahoo.com/d/quotes.csv?s={STOCKS}&amp;f={TAGS}"

The second position we needed to populate was the tags position (represented by "{TAGS}" in the above URL). Tags are just characters that symbolize different descriptions that, when included, return information relating to a stock. So if you include an "n" tag it displays the stocks name, an "a" tag displays the stocks asking price, a "t8" tag displays the stocks 1 Year Target Price (you get the idea). Continuing with our example, if the user wanted to look up the name, earnings per share and 52 week high for Apple's and Google's stock, we would include the "n", "e" and "k" tags in the tags position of the base URL. However, when we add the tags to the URL we concatenate them together, so the end result would be "nek."

Below is the final URL of the aforementioned example that will return a csv file that contains the specific information that we requested.

http://finance.yahoo.com/d/quotes.csv?s=AAPL+GOOG&amp;f=nek
