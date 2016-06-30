---
layout: post
title: Understading o(n Log(n))
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
date: 2016-06-30T19:48:00-04:00
---

The long awaited day has come and you finally graduated from a coding bootcamp. There is only one obstacle in your way from landing your dream job as a Web Developer – the technical interview. To prepare, you head to Wikipedia and start looking up the most prominent sorting algorithms. It's here where you're introduced to Big O notation and runtime complexities such as `O(n)` and `O(n^2)`. The first few complexities are pretty straightforward, it’s only when you find yourself staring at `O(n log<i>n</i>)` when the confusion kicks in. If you can relate, or you just want to brush up on what exactly `O(n log<i>n</i>)` means, then this blog post is for you.

First, let’s begin with understanding exactly what Big O notation is and why it's so useful. Big O notation is the measurement we use to gauge the runtime or complexity of an algorithm is as the input gets arbitrarily large.

Computer scientists agreed that measuring the exact runtime of an algorithm was an inadequate way to test its efficiency since the same algorithm could run faster on one computer opposed to another equally capable computer because of some external variable (such as containing a faster processor). Therefore, they concluded to gauge an algorithm not by how long it takes to run, but how much the runtime grows as the input gets larger.

For example, a simple iteration through an array of <i>n</i> items to find the largest value would be considered `O(n)`, or linear time, since you have to iterate through every item at least once to see if it is the largest digit. An implementation of this in ruby would be:

{% highlight ruby %}
def linear_time(digits)
  max = array.first
  digits.each do |digit|
    max = digit if digit > max
  end
end
{% endhighlight %}

Demonstrating `O(n^2)` is a little more complex. Let’s say we’re provided with two arrays: the first containing digits `[5, 6, 7, 8, 9, 10]` and the second array is a range of digits from 1 to 100 and we want to, for some inconceivable reason, print out all digits that are equally divisible by the numbers in the first array. Let’s see a Ruby implementation:

{% highlight ruby %}
def quadratic_time(divisors, range_of_digits)
  divisors.each do |divisor|
    range_of_digits.each do |digit|
      puts digit if digit % divisor == 0
    end
  end
end
{% endhighlight %}
*(This is not the most effective way to solve this problem)

Here, we’re nesting two loops: an outer and inner loop. As a result, the outer loop must run the length of the array, or <i>n</i> times, and the inner loop must run <i>n</i> times for the each iteration of the outer loop, giving us O(n^2)` or quadratic time.
