---
layout: post
title: Using CSV.parse to Access Stock Data
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
date: 2016-01-29T05:44:58-05:00
---
The Yahoo! Finance API makes it remarkably easy to retrieve realtime stock data and store it in a comma separated values (csv) file. But how do you get the values from the csv file into your program? That was the second challenge that I faced as I tried to implement my stock ruby gem.

Fortunately, this task was made substantially more easy after I discovered that Ruby has a CSV library with specifically defined methods that allow the programmer to easily transfer data from a csv file into their program. Below is a high level overview of the code that I utilized to migrate stock data from a csv file into my gem:

{% highlight ruby linenos %}
open(url) do |csv|
  CSV.parse(csv) do |row|
    puts row
  end
end
{% endhighlight %}

With just that code and a correctly assembled URL (see Formatting the URL blog post) I was able to retrieve realtime stock data from the Yahoo Finance API and access it within my program.

However, before I was able to get this code functioning there were a few obstacles I had to overcome. First, I had to require both the 'csv' and 'open-uri' gems in my Gemfile. Next, I had to pass the formatted URL containing the stock(s) the user wanted to look up along with the tags or information they wanted to view into the open-uri module. This returned a csv file that I could now use as an argument within our CSV.parse method.

The CSV.parse method simply goes row by row within the csv file and transfers the data into your program as an array of String objects. Each row is its own array and each value is it's own String object within that array.

I think it's time for an example. Say we requested to view the name and asking price of Apple's and Google's stock. The csv file would look like this:

{% highlight ruby linenos %}
"Apple Inc.", 115.46
"Alphabet Inc.", 755.97
{% endhighlight %}
<pre>(Alphabet Inc. is Google's parent company)</pre>

The CSV.parse method would then create a new array for each row of data and separately input each value as it's own String into that array. The final result would be:

{% highlight ruby linenos %}
[“Apple Inc.“, “115.46“]
[“Alphabet Inc.“, “755.97“]
{% endhighlight %}

Now each array is represented as a row in the CSV.parse loop. So, if we wanted to access Apple's name, and we were on the first iteration of row, we would enter <code>row[0]</code> in our loop, or if we wanted to access Apple's price we would enter <code>row[1]</code>. Then on our next iteration of row we would be able to access Alphabet Inc.'s elements.

Thanks to Ruby, the process of transferring data from a csv file to your program is made easy.
