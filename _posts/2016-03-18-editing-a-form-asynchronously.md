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

Recently I built a Rails application that allows a user to create workouts corresponding to a specific muscle group. Throughout my app I continuously relied on Ajax and JavaScript to make for a more seamless user experience. In this blog post I'll explain in detail how I used Ajax and JavaScript to allow a user to edit their workout without having to refresh the page.

<img src="{{ workout.png }}" alt="{{ workout }}"/>

Say a user enters a chest workout but a few days later decides they want to enter beast mode and increase their chest workouts reps from 12 to 15? Wouldn't it be a nice user experience if they could edit their workout right in the workout table row rather than requesting a new webpage? Fortunately, this exceptionally convenient feature can be accomplished with the help of Ajax and JavaScript.

Implementing this feature is complex, and requires a lot of moving components so it would be best to break it down into small, manageable parts. The first obstacle is setting up the DOM so it could either display the table data (the workout) or the form for the user to edit their workout. Initially, the workout values should be displayed in the DOM, and then when the user clicks the 'Edit' link it should hide the workout data and display an input form. So within each workout row, the table data will either reflect html text or a form. Since we'll be toggling between these two options, we can designate a particular class to the elements I want to display and a class to the elements I want to hide.

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

  <td><%= link_to 'Edit', edit_workout_plan_workout_path(current_user.current_plan,
  workout), class: 'edit-link', remote: :true %></td>
  <td><%= link_to 'X', workout_plan_destroy_workout_path(workout.exercise.workout_plan,
  workout), data: {id: workout.id }, method: :delete, remote: :true %></td>
<% end %>
{% endhighlight %}

Within each table data field we have the capability to either render text or an input form depending on what element we choose to show/hide. Initially, we want to display the workout values so we'll need to hide all of the input form fields. Without even assigning a class to the input fields we can accomplish this task by writing some basic CSS:

{% highlight ruby linenos %}
.workout-rows input {
  display: none;
}
{% endhighlight %}

Now all of the input fields will default to being hidden within the DOM unless told otherwise.

The next barrier will be hiding the plain html text while simultaneously displaying the workout input fields when the user clicks on the 'Edit' link. To achieve this we'll have to create an event listener that will perform this function whenever an 'Edit' link gets clicked:

{% highlight javascript linenos %}
  $('a.edit-link').click(function(){
    var workoutRow = $(this).parents('tr')

    // Hide plain text workout row and display edit workout input form
    $('span', workoutRow).addClass('hide-row');
    $('input', workoutRow).attr('id', 'edit-workout')
  });
{% endhighlight %}

First, we're tracking down what particular workout row we're dealing with. Then, we conceal the span element showing the workout values by adding the 'hide-row' class, and we display the edit workout form by adding the 'edit-workout' id to the input fields.

Now that the user has upgraded their workout to fulfill their beast mode requirements, we'll need to submit their form and update the workout row without refreshing the page. Enter obstacle number three: submitting the updated workout form via Ajax.

Remote true makes capturing the inputted values and transferring it to the backend server a piece  of cake; all you have to do is add `remote: :true` to your form and create the following event listener in your JavaScript file:

{% highlight javascript linenos %}
  $('form').submit(function(event){
    event.preventDefault();
  })
{% endhighlight %}

The new workout values will now be transferred to the controller action declared in the form where you can now successfully update your database. Okay, your database reflects the  correct values for this particular workout but we'll have to write some JavaScript if we want these values to appear in the DOM. Since the Ajax `dataType` for a `remote true` request is `script`, the controller action that the form is directed to will implicitly try to render a JavaScript file by the title of that action. In the case of this application, rails explicitly rendered `views/workouts/update.js.erb` where I wrote the following JavaScript code:

{% highlight javascript linenos %}
  var workoutId = <%= @workout.id %>;

  // Assign updated workout values to variables
  var workoutName = $('input.' + workoutId +'-name').val();
  var workoutSets = $('input.' + workoutId +'-sets').val();
  var workoutReps = $('input.' + workoutId +'-reps').val();


  // Display new workout values in DOM
  $('span[data-workout-name="' + workoutId + '"]').text(workoutName);
  $('span[data-workout-name="' + workoutId + '"]').toggleClass('hide-row');

  $('span[data-workout-sets="' + workoutId + '"]').text(workoutSets);
  $('span[data-workout-sets="' + workoutId + '"]').toggleClass('hide-row');

  $('span[data-workout-reps="' + workoutId + '"]').text(workoutReps);
  $('span[data-workout-reps="' + workoutId + '"]').toggleClass('hide-row');

  // Remove input fields from DOM
  $('input.' + workoutId +'-name').removeAttr('id');
  $('input.' + workoutId +'-sets').removeAttr('id');
  $('input.' + workoutId +'-reps').removeAttr('id');
{% endhighlight %}

First, we'll have to assign the workout id to a variable so we could reference it within our jQuery selectors. Next, we'll assign the new values that the user entered into the edit workout form to variable. Then, we'll update the DOM to display the new workout value and remove the `hide-row` class so the span will appear. Lastly, we'll remove the input fields by removing the id field.

Although it was a multistep process, giving users the ability to update their data without within the DOM and not have to be redirected to a new page is a feature worth implementing.  
