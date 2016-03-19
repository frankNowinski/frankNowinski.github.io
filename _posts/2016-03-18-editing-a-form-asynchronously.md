---
layout: post
title: Editing a Form Asynchronously
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
date: 2016-03-18T17:37:55-04:00
---

One essential feature of any good app is responsive and user-friendly forms.  

Recently I built a Rails application that enables a user to create new workouts corresponding to a specific muscle group, which is depicted by the diagram below.

But what happens if the user wants to enter beast mode and increase their reps from 12 to 15? Wouldn't it be a nice user experience if the user could edit their workout right in the workout table row rather than requesting a new webpage? Fortunately, we can accomplish this feature with the help of JavaScript and AJAX.

Implementing this feature is complex, and requires a lot of moving components so it would be best to break it down into small, manageable parts. Our first obstacle is setting up our DOM so we could either display the table data (the workout) or the form for the user to edit their workout. Initially, the workout data should be displayed in the DOM, and then when the user clicks the 'Edit' link it should hide the workout data and display an input form. So within each workout row, the table data will either reflect html text or a form. Since we'll be toggling between these two options, we can designate a particular class to the elements we want to display and a class to the elements we'll want to hide.

I'll be referring to the following code throughout this blog:

{% highlight erb linenos %}
<% exercise.workouts.each do |workout| %>
  <!-- Workout row -->
  <tr class="workout-rows" data-muscle-group-id="<%= exercise.muscle_group.id %>" data-workout-id="<%= workout.id %>">

  <%= form_for [exercise.workout_plan, workout], remote: :true do |f| %>
    <!-- Either display the html text or the form input -->
    <td>
      <span data-workout-name="<%= workout.id %>"><%= workout.name %></span>
      <%= f.text_field :name, {class: "form-control #{workout.id}-name"}%>
    </td>

    <td>
      <span data-workout-sets="<%= workout.id %>"><%= workout.sets %></span>
      <%= f.text_field :sets, {class: "form-control #{workout.id}-sets"} %>
    </td>

    <td>
      <span data-workout-reps="<%= workout.id %>"><%= workout.reps %></span>
      <%= f.text_field :reps, {class: "form-control #{workout.id}-reps"} %>
    </td>
    <!-- So the user can press enter to submit the form -->
    <%= f.submit style: 'display: none;' %>
  <% end %>

  <td><%= link_to 'Edit', edit_workout_plan_workout_path(current_user.current_plan, workout), class: 'edit-link', remote: :true %></td>
  <td><%= link_to 'X', workout_plan_destroy_workout_path(workout.exercise.workout_plan, workout), data: {id: workout.id }, method: :delete, remote: :true %></td>
<% end %>
{% endhighlight %}

Within each table data field we have the capability to either render text or an input form depending on what element we choose to show/hide. To begin with we'll want to display the workout text so we'll need to hide the input form field. Without even assigning a class to the input fields we can accomplish this task by writing some basic CSS:

{% highlight css %}
.workout-rows input {
  display: none;
}
{% endhighlight %}

Now all of the input fields will default to being hidden within the DOM unless told otherwise.

The next barrier will be hiding the plain html text while simultaneously displaying the workout input fields when the user clicks on the 'Edit' link. To achieve this we'll have to create an event listener that will perform this function whenever an 'Edit' link gets clicked:

{% highlight javascript %}
  $('a.edit-link').click(function(){
    var workoutRow = $(this).parents('tr')

    // Hide plain text workout row and display edit workout input form
    $('span', workoutRow).addClass('hide-row');
    $('input', workoutRow).attr('id', 'edit-workout')
  });
{% endhighlight %}

First, we're tracking down what particular workout row we're dealing with. Then, we can conceal the span element showing the plain html by adding the 'hide-row' class, and we can display the edit workout input by adding the 'edit-workout' id to the input fields.

Now that the user has upgraded their workout to fulfill their beast mode requirements, we'll need to submit their form and update the workout row without refreshing the page. Enter obstacle number three: submitting the updated workout form via Ajax.

Remote true makes capturing the inputted values and transferring it to the backend server a piece  of cake. All you have to do is add `remote: :true` to your form and create the following event listener in your JavaScript file:

{% highlight javascript %}
  $('form').submit(function(event){
    event.preventDefault();
  })
{% endhighlight %}

The new workout values will now be transferred to the controller action declared in the form where you can now successfully update your database. Okay, your database reflects the  correct values for this particular workout but we'll have to write some JavaScript if we want these values to appear in the DOM. Since the Ajax `dataType` for a `remote true` request is `script`, the controller action that the form is directed to will implicitly try to render a JavaScript by the title of that action. In the case of my application, rails explicitly rendered `views/workouts/update.js.erb` where I displayed the following code:

{% highlight javascript %}
var workoutId = <%= @workout.id %>

// Assign inputed values to be updated to variables
var workoutName = $('input.' + workoutId +'-name').val();
var workoutSets = $('input.' + workoutId +'-sets').val();
var workoutReps = $('input.' + workoutId +'-reps').val();


// Display new workout values in DOM
$('span[data-workout-name="' + workoutId + '"]').text(workoutName)
$('span[data-workout-name="' + workoutId + '"]').toggleClass('hide-row')


$('span[data-workout-sets="' + workoutId + '"]').text(workoutSets)
$('span[data-workout-sets="' + workoutId + '"]').toggleClass('hide-row')

$('span[data-workout-reps="' + workoutId + '"]').text(workoutReps)
$('span[data-workout-reps="' + workoutId + '"]').toggleClass('hide-row')

// $('span.' + workoutId + '-sets').text(workoutSets)
// $('span.' + workoutId + '-sets').removeClass('hide-row')

// Remove input fields from DOM
$('input.' + workoutId +'-name').removeAttr('id')
$('input.' + workoutId +'-sets').removeAttr('id')
$('input.' + workoutId +'-reps').removeAttr('id')  
{% endhighlight %}
