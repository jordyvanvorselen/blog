---
title: "Stop arguing about code style"
draft: false
date: 2023-11-08T10:00:00
tags: 
- Best Practices
- Software Engineering
- Code Style
- Formatting
categories:
- Best Practices
description: "As software engineers we all have our own opinions on how our code should look. But having discussions about code style at every pull request review adds up to be a giant waste of time. Here's a simple solution to be done with this once and for all."
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"
---

<!--more-->

#### Imagine

Yesterday, you finished a user story. You worked hard to add a nice new API endpoint, some automated tests and well-structured documentation. And just before you went home
you pushed everything to git and made sure to create a pull request to request feedback from your colleagues.

Today you start the day fully refreshed, and you open the pull request to check for comments. To your delight the pull request has already been reviewed! And there's *just a single
comment* on it.

Let's check it out.

```java
@RestController
@RequestMapping(value = "/api/v1")
@Tag(name = "Books", description = "Books API")
public class BookApiController {
    // Can you please remove this blank line? We prefer to not add a blank line after class declaration.
    @Autowired
    BookRepository bookRepository;

    ...
```
*^ The code comment represents the comment on the pull request.*

Oh man :sweat:. How would you feel?

I'm not going to argue that keeping or removing the blank line is a good idea. But I am going to argue that having this kind of comments on pull requests is
costing your team time. And they cost more time than you think. Instead of being able to merge it *now*, you first had to:

* Implement this tiny change;
* Re-request a review;
* Wait for the reviewer to approve.

These extra steps are causing both you and the reviewer to switch contexts a few times for something that could have been prevented in the first place. And it takes even more time if
you and the reviewer don't agree, because then you should add the time arguing in as well.

And all this arguing is probably not solidified by documenting it somewhere. In the best case, it is put in a document outside the codebase somewhere - but nobody
will ever read it. So you'll probably end up in the same or similar discussions time and time again.
 
Letting this go on and on can add up and result in an unnecessary drain of energy, and quite some time wasted. Time that could otherwise *be spent programming*
and thus *delivering value*.

#### How to prevent this

There's a simple but very effective solution for this problem.

> Automate it away using **linters** and **formatters**. 

Yes, almost all of us know these tools exist, but it still amazes me how many teams I have joined that did not fully utilize them. Modern languages like Dart,
Rust and Go even come equipped with formatters out of the box. Most other popular languages have 3rd-party tools for this. Just add the linter and formatter to your continuous integration pipeline, so that every pull
request is automatically checked, and be done with it. If it does not adhere to the rules, the build fails. 

This eliminates *all* discussion about code style on pull requests. If there's need for a discussion, it will be about the *rules*, not the code. And these
discussions are only *sparked* by pull requests now and **should never block merging** them. Discuss the rule change seperately from the pull request.

If you want it to change, merge it anyway and change it *later* after the team has discussed and approved the new rule.

##### But won't our git blame history be ruined by all the changes?

No, you can [exclude commits](https://github.blog/changelog/2022-03-24-ignore-commits-in-the-blame-view-beta/) from the git blame history.

##### But what if we don't agree with a rule?

Discuss rule changes in the team **separate from the pull request** that sparked it. This way the delivery won't be halted. Don't waste a lot of time discussing
the rules:

* Timebox the discussion;
* Let the team vote;
* If the votes are even - flip a coin or spin a random name picker.

Then simply change the configuration and solidify the decision in the codebase.

##### How can we test if everything keeps working correctly?

Don't you automate the testing? If you are worried about simple changes like this, it might be a good idea to start doing so.

Just use the same testing approach you'd normally use to test upgrades or refactors. Most of the changes are probably small trivial things anyway. In my
experience most formatting or linting tools with an autofix feature are quite safe to run - even in large codebases.

##### Won't the failing build become annoying?

Surprisingly, it's way less annoying if 'machine says no :robot:' than if a human does so. Make sure to configure format on save to fix code style automatically.
Now you'll just have to worry about the occasional linter error here and there.

#### Conclusion

In conclusion, code formatting tools do not only take away the need of manual formatting. It is only one of the many benefits. The most important benefit is that it will enforce
a shared understanding in the team of how the code should be formatted. This will prevent features being blocked by code style discussions.