---
layout: post
title: AngularJS: Factories vs Services
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
date: 2016-08-01T14:59:20-04:00
---

When you’re using AngularJS, just like in most other computer languages, you’ll most likely want to keep your controllers thin, storing most of your business logic and data elsewhere. This type of architectural design is referred to as “Separation of Concerns” and maintains that you split your applications tasks into different sections, where each section is responsible for a particular task. In Angular, this is achieved through services. Two of the most popular types of services in Angular are Factories and Services.

Factories and Services are pretty similar but have one key difference: you instantiate a Factory as an object, add properties to the object, and then return the object whereas you instantiate a Service with the <i>new</i> keyword, add properties to <i>this</i>, and return <i>this</i> or the Service instance.

To further clarify this distinction, let’s look at an example from an app that I'm currently working on that calculates a users daily return on investment for their stock portfolio. In the services, I'll make a call to the backend API to retrieve all of a users stocks.

{% highlight javascript %}
var app = angular.module("app", []);
app.controller('StockController', function($scope, StockFactory) {
  $scope.stocks = StockFactory.setStocks();
});

app.factory('StockFactory', function($http) {
  var stockFactory = {};

  stockFactory.setStocks = function() {
    return $http.get('api/v1/stocks');
  };

  return stockFactory;
});
{% endhighlight %}

First, you instantiate the Factory object (<code>stockFactory</code>), attach functions or properties to the object, and then finally return the object. Now, the function <code>setStocks</code> is available to use in the <code>StockController</code>.

Angular's Services work very similarly, except you attach properties directly to <code>this</code> and will be returning the Service itself:

{% highlight javascript %}
var app = angular.module("app", []);
app.controller('StockController', function($scope, StockService) {
  $scope.stocks = StockService.setStocks();
});

app.factory('StockService', function($http) {
  this.setStocks = function() {
    return $http.get('api/v1/stocks');
  };
});
{% endhighlight %}

Just like in the Factory, setStocks is available in every controller in which you inject, through dependency injection, the StockService.

In short, Factories and Services do pretty much the same thing, it's just a matter of code style. That being said, according to "Mastering Web Application Development with AngularJS" by <i>Pawel Kozlowski</i>, "The factory method is the most common way of getting objects into AngularJS dependency injection system. It is very flexible and can contain sophisticated creation logic. Since factories are regular functions, we can also take advantage of a new lexical scope to simulate "private" variables. This is very useful as we can hide implementation details of a given service."
