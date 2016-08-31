---
layout: post
title: Factories vs Services
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
date: 2016-08-31T10:50:40-04:00
---
When you’re using AngularJS, just like in most other computer languages, you’ll most likely want to keep your controllers thin, storing most of your business logic and data elsewhere. This type of architectural design is referred to as “Separation of Concerns” and maintains that you split your applications tasks into different sections, where each section is responsible for a particular task. In Angular, this can be achieved through services. Two of the most popular types of services in Angular are factories and services.

Although factories and services share many similarities, there is a key difference between them: you instantiate a factory as an object, add properties to the object, and return the object, whereas you create a new instance of a service (with the <code>new</code> keyword), add properties to <code>this</code>, and return <code>this</code>. Therefore, you can achieve the same things in a factory as you can in a service but since a factory is an object, it is much more flexible.

To further clarify this distinction, let’s look at an example from an app that I'm currently working on that calculates a user's daily return on investment for their stock portfolio. For demonstrating purposes, I'll make a call to the backend API to retrieve all of the stocks that are held in a user's portfolio.

{% highlight javascript %}
app.service('StockService', function($http) {
    this.setStocks = function() {
        return $http.get('api/v1/stocks');
    };
});

app.factory('StockFactory', function($http) {
    return {
        setStocks: function() {
            return $http.get('api/v1/stocks');
        }
    };
});
{% endhighlight %}

The above example shows how to receive a user's stocks in both a service and a factory. Looks pretty similar right? They may be performing the same functionality but the architecture that make up these services are quite different.

A service is viewed as a constructor function and upon instantiation, implicitly creates an empty object and assigns it to the variable <code>this</code>. <code>this</code> is also automatically returned to you.

A factory differs from a service in that a factory is just called, and whatever is returned from the factory is the value of that service. As a result, factories give programmers more freedom to manipulate the data according to their needs before they return it.

In addition, factories enable the programmer to instantiate a new object and assign it the appropriate data. For example, let's say we want to create a new stock object of Apple's stock containing only the stock symbol (AAPL) and the price. Here is how we could implement this using a factory:  

{% highlight javascript %}
app.factory('Stock', function($http) {
  return function(symbol, price) {
      return {
          symbol: symbol,
          price: price
      }
  };
});
{% endhighlight %}

In our <code>StockController</code>, we can create a stock object like this:

{% highlight javascript %}
var app = angular.module("app", []);
app.controller('StockController', function(Stock) {
    var stock = new Stock('AAPL', '106.10');
});
{% endhighlight %}

As a result, our <code>stock</code> variable would be assigned <code>{symbol: "AAPL", price: "106.10"}</code>.

In short, factories and services have very similar functionality but have a key distinguishable difference. That being said, I'll leave you with a quote by <i>Pawel Kozlowski</i> from his book "Mastering Web Application Development with AngularJS": "The factory method is the most common way of getting objects into AngularJS dependency injection system. It is very flexible and can contain sophisticated creation logic. Since factories are regular functions, we can also take advantage of a new lexical scope to simulate "private" variables. This is very useful as we can hide implementation details of a given service."
