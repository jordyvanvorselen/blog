---
title: "Do Repeat Yourself"
draft: false
date: 2024-01-12T15:00:00
tags:
  - Best Practices
  - Software Engineering
  - Abstractions
categories:
  - Best Practices
description:
  "One of the first things we learn aspiring software engineers is the principle Don't Repeat
  Yourself. While this is a very useful rule of thumb, it doesn't apply in every case. In this post
  we will discuss when it does not."
images: []
resources:
  - name: "featured-image"
    src: "featured-image.png"
canonical: https://kabisa.nl/en/insights/do-repeat-yourself/
---

<!--more-->

One of the first things we teach aspiring software engineers is the principle **Don't Repeat
Yourself**. The notion that as software engineers we should _never_ write the same code twice.
Duplication makes our code harder to change. So to prevent this, we learn to take all cases of the
oh so dreaded duplication away.

Most of the time, the principle applies and will make the codebase better. So, after learning this,
it's quite likely each and every case of duplication becomes one we fight. Before we know it we are
waging war against duplication and exterminating it left and right using our favourite weapons -
refactoring it into a function, utility class or even a standalone library for others to use.

But is it _always_ worth it? Is abstracting _each and every_ duplication away worthy of your
precious time? And if not for time - is it always beneficial for maintainability?

Here, I'll say it:

> In some cases DRY is not worth it.
>
> In some cases DRY does not help, but **harms** maintainability.

Before burning me at the stake :fire:, please let me explain...

#### Abstractions

Refactoring something specific, like extracting duplicated lines to a _generic_ function, is an
[abstraction](<https://en.wikipedia.org/wiki/Abstraction_(computer_science)#:~:text=In%20software%20engineering%20and%20computer%20science%2C%20abstraction%20is%20the%20process%20of%20generalizing%20concrete%20details%2C>) -
albeit on a granular scale.

Refactoring a duplicated piece of code into the right abstraction is very satisfying. Especially
when the _moment of change_ inevitably knocks on your door. The business changes, so the code needs
to change. Oh, how convenient! We abstracted this code into a function and now we only have to
change it in one place instead of having to change it throughout the whole codebase.

It's the _moment of validation_ for our choices of the past. The indisputable proof we made the
right choice. :muscle:

In this case it was worth the time that was invested into the refactor. It made our lives _easier_
when change was needed.

#### A bad abstraction

But in other cases creating an abstraction that looks very similar to the abstraction above, can
make our lives _harder_.

Let's take a look at an example.

Once again we extract some duplication between two classes into a helper function while
simultaneously dreaming of the moment of validation - which will surely arrive soon. We wait a while
and surely enough, the business changes again. Of course, now we need to update the software.

We refine the change and give it quite a low estimate, as we expect to be able to easily implement
the change. This is part of the _clean code territory_ in the project after all! :tada:

But instead of being met with the happy experience of _convenience_, we notice that it's now a
little harder to add the new business rules than before.

Apparently, only the behaviour of one of the two original classes needs to be changed. As we already
invested the time to create this beautiful abstraction, we absolutely can't revert back to the old
ways. That would feel _bad_, since then we would be admitting we made the wrong choice in the past.
:thinking_face:

So we decide to just add a parameter to the extracted function. I mean, just one more function
parameter is still readable. We can implement this change pretty quickly. Nothing wrong with just a
single parameter, right? And anyway, we might be able to benefit from the abstraction
[in the future](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it).

_And today, a bad abstraction was born._

And the 6 other engineers that touched the code after us? They will surely keep the abstraction in
place. Someone spent effort on creating this abstraction, so it should be there for a good reason!
Undoing this work and going back to the original duplication sounds like _madness_, it'd be almost
_insulting_ to the person who created it.

So some of them add a few positional parameters, others add parameters with default values and the
last one even adds a few keyword arguments:

```java
public void generate(Data data, MailData mailData, sendMail = false, type = Type.A, alsoGenerateY = true) /* etc... */ {
    generateX(data);

    // Parameters and checks keep getting added.
    // Complexity keeps rising over time, at some
    // point it becomes unmaintainable.
    if (sendMail) {
        if (type != Type.B && mailData != null) {
            sendMail(mailData);
        }

        if (alsoGenerateY && type != Type.A && mailData.X == true) {
            generateY(data);
        }
    }
}
```

The code is DRYer than the Sahara at high noon, but the cycle goes on, and on, and on...

What do you think about this? Let me know in the comments :point_down:
