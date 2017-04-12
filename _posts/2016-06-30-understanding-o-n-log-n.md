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

The long awaited day has come and you finally graduated from a coding bootcamp. There is only one obstacle in your way from landing your dream job as a Web Developer – the technical interview. To prepare, you head to Wikipedia and start looking up the most prominent sorting algorithms. It's here where you're introduced to Big O notation and runtime complexities such as <code>O(n)</code> and <code>O(n^2)</code>. The first few complexities are pretty straightforward, it’s only when you find yourself staring at <code>O(n log(n))</code> when the confusion kicks in. If you can relate, or you just want to brush up on what exactly <code>O(n log(n))</code> means, then this blog post is for you.

First, let’s begin with understanding exactly what Big O notation is and why it's so useful. <strong>Big O notation is the measurement we use to gauge the runtime or complexity of an algorithm as the input gets arbitrarily large.</strong>

Computer scientists agreed that measuring the exact runtime of an algorithm was an inadequate way to test its efficiency since the same algorithm could run faster on one computer opposed to another equally capable computer because of external variables (such as containing a faster processor). Therefore, they concluded to gauge an algorithm not by how long it takes to run, but by how much the runtime grows as the input gets larger.

For example, a simple iteration through an array of <i>n</i> items to find the largest value would be considered <code>O(n)</code>, or linear time, since you have to iterate through every item at least once to see if it is the largest digit. An implementation of this in Ruby would be:

{% highlight ruby %}
def linear_time(digits)
    max = digits.first
    digits.each do |digit|
        max = digit if digit > max
    end
end
{% endhighlight %}

Demonstrating <code>O(n^2)</code> is a little more complex. Let’s say we want to print out all the digits between 1 and 100 that are equally divisible by the numbers in an array. This is how it would look look in Ruby:

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

Here, we’re nesting two loops: an outer and inner loop. First, the outer loop must iterate over the length of the array. Then, for each iteration of the outer loop, the inner loop must iterate each item in it's array, finally giving us a runtime of <code>O(n^2)</code> or quadratic time.

After a brief explanation of linear and quadratic, let's now try to tackle the meaning behind <code>O(n log(n))</code>. The first part of the complexity, <code>O(n)</code>, should look familiar – we're iterating over the length of the array, touching each item once.

The second part of the complexity, <code>log(n)</code>, may look a little cryptic if you've never encountered it before. Essentially, this asks how many times do we have to divide <i>n</i> in half to convert all the items in <i>n</i> to equal subarrays of 1? To better explain this complexity, I'll walkthrough how to implement mergesort, a popular algorithm that has a complexity of <code>O(n log(n))</code>. (Other common algorithms that have this complexity are heapsort, timesort and, quicksort.)

The basic premise of mergesort comes down to these three steps:
<ul>
    <li>Divide the array in half</li>
    <li>Sort the divided array</li>
    <li>Merge the sorted arrays</li>
</ul>

To sort the divided array, mergesort reduces each element in the array into a subarray of 1 by using a common computer science function called recursion. Let's say we want to sort the following array of integers from 1-8: <code>[8, 2, 4, 7, 3, 1, 6, 5]</code>. Mergesort believes in reducing this array of eight elements into eight individual subarrays by repeatedly dividing itself in half until each element is its own subarray.

Following our example, the array is first cut in half to equal <code>[8, 2, 4, 7]</code>. Then, mergesort is recursively called (since the array's length is larger than 1) and cut in half again to equal <code>[8, 2]</code>. This process continues until we're left with <code>[8]</code> and <code>[2]</code>, the first two elements of the array. Now that there's only two elements, we can compare one to the other and begin sorting our array. While implementing mergesort, you usually split up the array into the left and right half. Here, <code>[8]</code> would be declared the left portion and <code>[2]</code> the right. So, the first pair of our sorted array would be <code>[2, 8]</code> since <code>8</code> is less than <code>2</code>. Next, we would sort <code>[4, 7]</code>, which just so happens to be sorted. Finally, we would merge the first two pairs into <code>[2, 4, 7, 8]</code>, giving us the sorted left portion of our unsorted array. This process is completed on the right half of the array, and then the left and right portions are merged together. You can see a full implementation of mergesort in Ruby below:

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

As stated earlier, mergesort's complexity is <code>O(n log(n))</code>. The <code>log(n)</code> is attributed to the number of times we divide the array in half until each element is converted to subarrays of 1. The <code>n</code> comes from the time it takes to merge the left and the right sorted halves together. <code>O(n log(n))</code> is such a popular complexity because it has the best worst-case runtime you can get achieve for sorting.
