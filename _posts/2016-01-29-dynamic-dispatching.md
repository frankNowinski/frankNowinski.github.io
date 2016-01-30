---
layout: post
title: Dynamic Dispatching Basics
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
date: 2016-01-29T05:46:01-05:00
---
Besides the added benefit of writing more concise and considerably less code, metaprogramming has many other advantages as well. Possibly the most appealing feature of metaprogramming is it allows your code to become much more flexible and easier to edit. Although there are many aspects of metaprogramming, I would like to focus on one concept in particular: dynamic dispatching. Dynamic dispatching is a valuable tool for any programmer because you can call methods throughout your program at run time.

In Ruby, we rely on the <code>send</code> method to dynamically dispatch. To illustrate how the <code>send</code> method works I’ll use the below example:

{% highlight ruby linenos %}
"Hello, World".send("reverse")
{% endhighlight %}
* The arguments passed into the send method could be a String or a Symbol.

Here, the <code>send</code> method, or the message, converts the <code> "reverse"</code> argument into a method and determines whether it’s a viable method to be called upon the receiver, the String object <code>"Hello, World"</code>. Since it is, Ruby evaluates it and outputs the following:

{% highlight ruby linenos %}
"Hello, World".send("reverse") #=> "dlroW ,olleH"
{% endhighlight %}

However, if the method cannot be called upon the object then an error message is displayed. For example, if we tried to add the integer 1 to the String object <code>"Hello, World"</code>, we would get the following <code>Type Error</code>:

{% highlight ruby linenos %}
"Hello, World".send(:+, 1)
 #=> TypeError: no implicit conversion of Fixnum into String
{% endhighlight %}

---

#Updating a Hash using Dynamic Dispatching

Another opportune time to utilize dynamic dispatching is when you want to update a hash. Let’s say you’re creating an application that tracks a users fitness goal for the week. At the beginning of the week the user designates how many minutes they plan on working out each muscle group and throughout the week they’re able to update their progress and monitor their results. The users weekly goal is stored in a hash like the one below where the key is the muscle group and the value is the amount of time they plan on working out that muscle:

{% highlight ruby linenos %}
"goal"=> {
  "chest"=> 45,
  "back"=> 45,
  "legs"=> 45,
  "shoulders"=> 30,
  "arms"=> 30,
  "cardio"=> 120
}
{% endhighlight %}

Our intention is to have the <code>goal</code> hash reflect the users progress after they conclude a workout. Let’s say the user worked out their chest for 40 minutes as well as doing about 20 minutes of cardio. We could expect to receive this data in a hash where the key is the muscle group and the value is the duration of the workout corresponding to that muscle.

{% highlight ruby linenos %}
updated_muscles = {"chest"=> 40,  "cardio"=> 20}
{% endhighlight %}
* We’ll assign the hash to a variable called <code>updated_muscles</code> so we can use this data later on in our dynamic dispatching function.

Although it may not be the most eloquent code, we could use a simple if/else statement to update our <code>goal</code> hash:

{% highlight ruby linenos %}
updated_muscles.each do |muscle, updated_t|
  original_t = goal.send("#{muscle}")
  if muscle == "chest"
    goal[:chest] = (original_t - updated_t)
  elsif muscle == "back"
    goal[:back] = (original_t - updated_t)
  elsif muscle == "legs"
    goal[:legs] = (original_t - updated_t)
  elsif muscle == "shoulders"
    goal[:shoulders] = (original_t - updated_t)
  elsif muscle == "arms"
    goal[:arms] = (original_t - updated_t)
  elsif muscle == "cardio"
    goal[:cardio] = (original_t - updated_t)
  end
end
{% endhighlight %}

Is it just me or does there seem to be a lot of repetitious code in the above function? Let’s see if we can refactor this function using dynamic dispatching.

{% highlight ruby linenos %}
updated_muscles.each do |muscle, updated_t|
  original_t = goal.send("#{muscle}")
  goal.send(("#{muscle}="), (original_t - updated_t))
end
{% endhighlight %}

That’s better. These two functions perform the same task, just with different syntax. Now let’s examine the latter function to fully understand the functionality behind it.

To correctly update the <code>goal</code> hash we have to subtract the users inputted workout duration from their original goal. As the function iterates over each key-value pair stored inside the <code>updated_muscles</code> variable, the time that is associated with the muscle group that the current iteration is being called upon is stored in a local variable called <code>original_t</code>. In our example, the first iteration being called is the chest key-value pair, therefore <code>original_t</code> is set equal to 45, the value of the chest key from the original <code>goal</code> hash.

Now we can use the value of <code>original_t</code>, subtract it from the value of <code>updated_t</code>, and assign the difference to the original <code>goal</code> hash, thereby successfully updating our hash like we intended. On the current iteration, we access the chest key from our <code>goal</code> hash and assign it the value of 5 (45 – 40). This process would be repeated until each key-value pair in <code>updated_muscles</code> is iterated over. The updated <code>goal</code> hash would become:

{% highlight ruby linenos %}
"goal"=> {
  "chest"=> 5,
  "back"=> 45,
  "legs"=> 45,
  "shoulders"=> 30,
  "arms"=> 30,
  "cardio"=> 100
}
{% endhighlight %}

By using dynamic dispatching you can significantly reduce the lines of code you need to express a solution. In addition, your code will be able to handle future edits with ease. What’s not to like?
