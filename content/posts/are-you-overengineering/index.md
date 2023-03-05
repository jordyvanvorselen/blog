---
title: "Are you over-engineering your software?"
date: 2023-03-05T12:00:00+01:00
draft: true
tags: ["over engineering", "software engineering", "architecture"]
categories: ["architecture"]
description: "Are you choosing the <i>best</i> or the <i>simplest</i> tool for the job?"
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"
---

## The best tool

In my career I have a seen a number of very complex applications. These applications would be impossible or very hard to build with server-side rendered frameworks compared to a single-page app framework. They had to handle millions of users while performing complex user interface actions. They needed to be covered by a proper test suite. They had to be dynamic and multiple people needed to work on them at the same time. These tools were built using React *for a reason*.

In my eyes a framework like React is an example of a "*best* tool for the job". A tool you dust off and fetch if you encounter a specific usecase that would require the complexity of React. It has a lot of pros and cons.

There's absolutely nothing wrong with this.

## De facto standard

But then I've also seen projects that chose React because it was *the standard*. They just followed the hype. I've encountered *admin panels* written in React. Yes, admin panels. And no, these were not highly customized. Simple CRUD forms that had to be touched by an admin once every few weeks...

Aren't we sometimes going too far in IT?

The fact that new fancy tools are available to solve more complex problems, does not mean we have to use them every chance we get. The *boring* "old" stuff works just as fine in these cases.

## The simplest tool

These customers most likely got very good admin panels. But would they have bought that admin panel if they knew **they could get it in a quarter of the time**? Or at least something similar? Tools like Ruby on Rails or Django would not require a separate frontend and backend team. No external API would be needed - the only consumer would be the admin panel anyway. It would just simple result in a cheaper product for the customer.

And this is not the only example. Quite often developers choose *the best* solutions, while our customers do not always require them. I beg to differ. Choose the *simplest* and most *boring* solution you can get away with for the project.

It doesn't matter if it's shiny or standard. It doesn't matter if it's exciting tech or boring tech. In the end the customer will care about how fast you can deliver value by *solving the business problem*.
