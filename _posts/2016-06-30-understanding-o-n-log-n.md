---
layout: post
title: Understanding O(n log(n))
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
date: 2016-06-30T20:04:18-04:00
---

The long awaited day has come and you finally graduated from a coding bootcamp. There is only one obstacle in your way from landing your dream job as a Web Developer – the technical interview. To prepare, you head to Wikipedia and start looking up the most prominent sorting algorithms. It's here where you're introduced to Big O notation and runtime complexities such as `O(n)` and `O(n^2)`. The first few complexities are pretty straightforward, it’s only when you find yourself staring at `O(n log(n))` when the confusion kicks in. If you can relate, or you just want to brush up on what exactly `O(n log(n))` means, then this blog post is for you.

First, let’s begin with understanding exactly what Big O notation is and why it's so useful. <strong>Big O notation is the measurement we use to gauge the runtime or complexity of an algorithm as the input gets arbitrarily large.</strong>

Computer scientists agreed that measuring the exact runtime of an algorithm was an inadequate way to test its efficiency since the same algorithm could run faster on one computer opposed to another equally capable computer because of external variables (such as containing a faster processor). Therefore, they concluded to gauge an algorithm not by how long it takes to run, but by how much the runtime grows as the input gets larger.

For example, a simple iteration through an array of <i>n</i> items to find the largest value would be considered `O(n)`, or linear time, since you have to iterate through every item at least once to see if it is the largest digit. An implementation of this in Ruby would be:

{% highlight ruby %}
def linear_time(digits)
  max = digits.first
  digits.each do |digit|
    max = digit if digit > max
  end
end
{% endhighlight %}

Demonstrating `O(n^2)` is a little more complex. Let’s say we want to print out all the digits between 1 and 100 that are equally divisible by the numbers in an array. This is how it would look look in Ruby:

{% highlight ruby %}
def quadratic_time(divisors, range_of_digits)
  divisors.each do |divisor|
    range_of_digits.each do |digit|
      puts digit if digit % divisor == 0
    end
  end
end
{% endhighlight %}
<small>\*Not the most efficient solution</small>

Here, we’re nesting two loops: an outer and inner loop. First, the outer loop must iterate over the length of the array. Then, for each iteration of the outer loop, the inner loop must iterate each item in it's array, finally giving us a runtime of `O(n^2)` or quadratic time.

After a brief explanation of linear and quadratic, let's now to try to tackle the meaning behind `O(n log(n))`. The first part of the complexity, `O(n)`, should look familiar – we're iterating over the length of the array, touching each item once.

The second part of the complexity, `log(n)`, may look a little cryptic if you've never encountered it before. Essentially, this asks how many times do we have to divide <i>n</i> in half to convert all the items in <i>n</i> to equal subarrays of 1? I'll walkthrough how to implement mergesort, a popular algorithm that has a complexity of `O(n log(n))`, to better explain this complexity. (Other common algorithms that have this complexity are heapsort, timesort and, on average, quicksort.)

The basic premise of mergesort comes down to these three steps:
<ul>
  <li>Divide the array in half</li>
  <li>Sort the divided array</li>
  <li>Merge the sorted arrays</li>
</ul>

To sort the divided array, mergesort reduces each element in the array into a subarray of 1 by using a common computer science function called recursion. Let's say we want to sort the following array of integers from 1-10: `[8, 2, 4, 7, 3, 1, 6, 9, 10, 5]`. Mergesort believes in reducing this array of 10 elements into 10 individual subarrays by repeatedly dividing itself in half until each element is its own subarray.

Following our example, first the array is reduced to `[8, 2, 4, 7, 3]`. Then, mergesort is recursively called (since the array's length is larger than 1) and cut in half again to equal `[8, 2]`. This process continues until we're left with `[8]` and `[2]`, the first two elements of the array. Now the left portion of the array is compared to the right portion and merged together to form a sorted array, giving us `[2, 8]`. This technique is then applied to the right portion of the array, and then the two arrays are merged together. See a full implementation of mergesort in Ruby below:

{% highlight ruby %}
def merge_sort(arr)
  if arr.length <= 1
    arr
  else
    middle = (arr.length/2).floor
    left = merge_sort(arr[0..middle-1])
    right = merge_sort(arr[middle..-1])
    merge(left, right)
  end
end

def merge(left, right)
  if left.empty?
    right
  elsif right.empty?
    left
  elsif left.first < right.first
    [left.first] + merge(left[1..left.length], right)
  else
    [right.first] + merge(left, right[1..right.length])
  end
end
{% endhighlight %}

Now, you can see that `log(n)` is attributed to the number of times we have to divide the array of integers in half until they're converted to subarrays of 1, and the `n` comes from the time it takes to merge both the left portion and the right portion together. To conclude, `O(n log(n))` is such a popular  complexity because it has the best worst-case runtime we can get for sorting.
