---
title: "Write elegant integration tests"
draft: true
date: 2025-07-22T15:00:00
tags:
  - Java
  - Spring Boot
  - SQL
  - Integration Testing
  - REST
  - API
  - Backend
  - Software Engineering
  - Flyway
  - Clean Database
categories:
  - Java
description:
  "A step-by-step guide on how to write elegant, isolated integration tests for a Spring Boot REST
  API. We'll go over everything from setting up the database to creating a robust setup for the
  testdata that will speed up writing both unit and integration tests in the future."
images: []
resources:
  - name: "featured-image"
    src: "featured-image.png"
canonical: https://kabisa.nl/en/insights/TODO/
---

<!--more-->

So, you finally decided that you need more than just unit tests for your application? Did a bug or
two slip through because they were not caught by the unit tests that use extensive mocking?

And after production broke you told your manager that it would have never happened if you'd only
have integration tests? And _somehow_ they agreed and gave you the time to set them up?

Then you've come to the right place! Let's walk the talk and **create that first integration test**.
Let's become _confident_ when making changes again! :muscle:

But before we can get to writing that first beautiful integration test, we have to set up the
integration test architecture first - and while doing so, many questions pop up:

> What's the difference between [unit and integration tests]()?
>
> Should I [use mocks]() in integration tests?
>
> Should I use [a real database]() in integration tests?
>
> How to make sure [each test is isolated]() from other tests?
>
> MockMvc or TestRestTemplate? [What's the difference]()?
>
> How to properly [seed the test data]() without clutter everywhere?
>
> :exploding_head:

If you didn't notice it yet, the questions above all link to an article answering the question.
Writing down all the answers to these questions in this single blog post would make it very long and
unclear. We'll briefly touch upon the answer for each question, but if you want to know more about
_why_ we make certain choices in this architecture then you can find the answer in one of the links
above.

In this article we will focus on actually setting up the architecture for the integration tests. We
will write our **first fully isolated integration test** for a Spring Boot REST API. We will tidy
the setup away into a base class, so that adding more integration tests in the future will be a
breeze for everyone in your team.

#### What's the difference between unit and integration tests?

Unit tests are tests that test a single unit of code in isolation. This means that all dependencies
of the unit under test are mocked away. This is done to make sure that the unit under test is
actually the only thing being tested.

This makes unit tests exceptionally fast and easy to write. However, it also means that the unit
tests are not testing the integration between the unit under test and its dependencies. This is
where integration tests come in.

Integration tests are tests that test the integration between multiple units of code. This means
that the dependencies of the unit under test are not mocked away, but are actually used.

#### So should I use mocks in integration tests?

But "multiple units of code" leaves a lot of room for interpretation. What parts of the code you
mock away, depends on what type of integration test you're writing.

However, in this article we are focusing on writing integration tests for API endpoints. This means
that we will _preferably_ not mock away any _internal_ dependencies, because we want to test the
integration between all layers of _our_ application.

It's fine to mock away sending push messages via firebase, sending emails or calling external APIs.
But we will not mock away the database, the service layer or the repository layer.

So for the tests we're going to write, the answer to the question above is therefore: **only for
external dependencies**.

#### Should I use a real database in integration tests?

_Yes, you should._ Most of the time.

We want to test the integration between our application and the database. To do so, we need to use
the same type of database as we use in production. This makes sure that you can find bugs in the
queries that you write, before they hit production.

And yes, even ORM's can sometimes behave differently when using a different database.

But the big tradeoff here is speed - spinning up a database is costly and setting up, maintaining and
running the tests will take more time.

#### How to make sure each test is isolated from other tests?

This is a very important question. If you don't make sure that each test is isolated from other
tests, then you will get unmaintainable flaky tests. Flaky tests are tests that sometimes fail and
sometimes pass, even though the code under test hasn't changed.

To make sure that each test is isolated from other tests, we will make sure that each test starts
with a clean database. This means that we need to reset the database _before_ each test.

#### MockMvc or TestRestTemplate? What’s the difference?

MockMvc is a helper that is provided by Spring Boot which allows you to test your API endpoints by
mocking HTTP requests. This means that you don't have to start a server to test your API endpoints.
This is great for controller unit tests, but not for integration tests.

TestRestTemplate is a class that is provided by Spring Boot. It allows you to test your API
endpoints by actually sending HTTP requests to a running server. This means that you need to start a
server to test your API endpoints. This is slower, but more realistic.

As we want to test if our API _actually_ works, we will use TestRestTemplate.

#### How to properly seed the test data without clutter everywhere?

Prefer factories over fixtures blogpost.
