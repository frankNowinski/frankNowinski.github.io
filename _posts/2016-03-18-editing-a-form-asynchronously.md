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

Within each table data field we have the capability to either render text or an input form depending on what element we choose to show/hide. To begin with we'll want to display the workout text so we'll need to hide the input form field. Without even assigning a class to inputs field we can accomplish this task by writing some basic CSS:

{% highlight css %}
.workout-rows input {
  display: none;
}
{% endhighlight %}
