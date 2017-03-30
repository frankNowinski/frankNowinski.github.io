---
layout: post
title: What Is Idempotency and Why Is It Useful?
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
date: 2017-03-30T15:57:45-04:00
---

Just the other day I was on a phone interview when I was asked to explain the benefits of idempotency. At first, this term bewildered me, but after asking the interviewer to elaborate, I had a good idea as to what he was referring to. Nevertheless, after the interview I researched the term to get a better understanding of what idempotency is and why it's useful when constructing RESTful APIs. This is what I learned.

Wouldn't it be nice if you never lost your wifi connection and were always able to connect to the internet? I think we can all agree that it would, but we aren't quite technologically there yet. We still lose our wifi connection, our requests may not reach the server, our responses may not reach the client, etc. Since the internet is susceptible to an unreliable connection, we want to make our APIs as robust and reliable as possible.

Continuing on what I said above, there are a few ways a network could lose it's connection:

* The initial request from the client to the server could fail.
* A call could fail while the server is processing the request.
* The request could get processed, but the response from the server to the client could fail.

For example, say you're making a GET request to a SHOW action but the request never connects to the server. In this scenario, the request would simply get resent to the server. Since there's a chance this request can get called more than once, we want to design our actions so that we can rest assured knowing that the same result will be returned no matter how many times the method is called, which is essentially what idempotency is.<strong> An idempotent method returns the same result, regardless if you call the method once or if you call it 100 times. </strong> Therefore, in an idempotent SHOW action, when the initial request fails, the request would get resent and the SHOW action would return the same result as if the initial call was processed successfully.

Let's take a look at an idempotent UPDATE action to illustrate what would happen if a request would fail while the server is fulfilling the operation. The UPDATE action is responsible for updating a record in the database with the parameters that are sent to the action. If a request to the UPDATE action failed either before or after the record was updated, the request would be resent to the action where it would proceed to run normally. You don't have to be concerned with any adverse side effects because if the record is updated once or updated five times, it's still going to update the record with the same parameters that it was passed.

Finally, lets take a look at the CREATE action, which isn't considered idempotent, and discuss what we can do to ensure this method is robust and reliable. Typically, you'll want the CREATE method to be processed once and only once to prevent adding multiple records to the database; or, if you're endpoint is charging your customers money, charging your customers multiple times. To prevent these unwanted side effects, we can rely on <i>idempotentcy keys</i> which is a unique ID that's generated and sent along with the request from the client to the server. Now, on the server side, you can read the idempotentcy keys ID and determine what type of logic to perform.

If the request failed before it was processed, a second request would be sent and the action would proceed to be processed as normal. If the request is successfully processed, the result is cached and the response is sent back to the client. In the event that the client doesn't receive the response, the CREATE action would receive the request again, read the idempotency key, know that the request was already processed, and return the cached result.

Any API should be concerned with minimizing adverse side effects and uncertainty and by implementing idempotent methods in your controllers, you do just that.   
