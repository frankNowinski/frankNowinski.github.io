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

Implementing this feature is complex, and requires a lot of moving components so it would be best to break it down into small, manageable parts. Our first obstacle is setting up our DOM so we could either display the table data (the workout) or the form for the user to edit their workout. Here, we can assign a particular class to the element that we want to display and a class to the elements we want to hide.

{% highlight erb linenos %}
<% exercise.workouts.each do |workout| %>
  <tr class="workout-rows" data-muscle-group-id="<%= exercise.muscle_group.id %>" data-workout-id="<%= workout.id %>">

  <%= form_for [exercise.workout_plan, workout], remote: :true do |f| %>
    <td>
      <span data-workout-name="<%= workout.id %>"><%= workout.name %></span>
      <%= f.text_field :name, {class: "form-control #{workout.id}-name", value: workout.name} %>
    </td>

    <td>
      <span data-workout-sets="<%= workout.id %>"><%= workout.sets %></span>
      <%= f.text_field :sets, {class: "form-control #{workout.id}-sets", value: workout.sets} %>
    </td>

    <td>
      <span data-workout-reps="<%= workout.id %>"><%= workout.reps %></span>
      <%= f.text_field :reps, {class: "form-control #{workout.id}-reps", value: workout.reps} %>
    </td>
    <%= f.submit style: 'display: none;' %>
  <% end %>
<% end %>
{% endhighlight %}
