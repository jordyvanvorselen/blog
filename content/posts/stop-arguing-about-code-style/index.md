---
title: "Stop arguing about code style"
draft: true
date: 2023-11-08T10:00:00
tags: 
- Best Practices
- Software Engineering
- Code Style
- Formatting
categories:
- Best Practices
description: "As software engineers we all have our own opinions on how to format our code. But having a discussion about code style at every pull request review is a giant waste of time. Here's a simple solution to be done with this once and for all."
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"
---

<!--more-->

Yesterday you finished a user story. You added a nice new API endpoint, added some automated tests and wrote the documentation. Just before you went home
you pushed everything to git and made sure to create a pull request and request feedback from your colleagues.

Today, you open your laptop and open the pull request to check for comments. To your delight the pull request has already been reviewed! And there's *just a single
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

Oh man :tired_face:. I think we all know this feeling too well.

I'm not going to argue that keeping or removing the blank line is a good idea. But I am going to argue that having this kind of comments on pull requests is
costing your team time. And they cost more time than you think. Instead of being able to merge it after the first review, you and the reviewer had to:

* (Optional) Discuss these 
* Implement this tiny change;
* Re-request a review;
* Wait for the reviewer to approve.

These extra steps are causing both you and the reviewer to switch contexts for something that totally could have been prevented. And it takes even more time if
you and the reviewer don't agree, because then you should add the time arguing in as well. Repeating these steps a few times can add up and result in quite some
time that could otherwise be spent programming (and thus delivering value).

#### How to prevent this

There's a simple but very effective solution for this problem.

> Automate it away using **linters** and **formatters**. 

Yes, almost all of us know these tools exist, but
it still amazes me how many teams I have joined that did not utilize them. Add the linter and formatter to your continuous integration pipeline, so that every
pull request is automatically checked. If it does not adhere to the rules, the build fails. 

*Voila*, from now on your team will never have to argue about code style again in pull requests. The discussion will never be about the specific code in the pull
request, but about the rules and configuration.

##### Won't our git blame history be ruined by all the changes?

No, you can [exclude commits](https://github.blog/changelog/2022-03-24-ignore-commits-in-the-blame-view-beta/) from the git blame history.

##### But what if we don't agree with a rule?

Discuss rule changes in the team **separate from the pull request** that sparked it. This way the delivery won't be halted. Don't waste a lot of time discussing
rules:

* Timebox the discussion
* Let the team vote after the discussion
* If the votes are even - flip a coin

Then simply change the configuration. The decisions are now written down in your codebase. :wink:

##### How can we test if it all keeps working correctly?

Don't you automate the testing? If you are worried after changes like this, it might be a good idea to start doing so. But, just use the same testing approach 
you'd normally use to test upgrades or refactors. Most of the changes are probably small trivial things anyway. In my experience most tools are quite safe to
run, even in large codebases.

##### Won't the failing build become annoying?

Surprisingly, it's way less annoying if 'machine says no :robot:' than if a human does so. Make sure to configure format on save to fix code style automatically.
Now you'll just have to worry about the occasional linter error.

#### Conclusion

Quite a simple solution, but it will make your life a whole lot easier. The next time you are about to start arguing about code style, just use the time to automate
the discussion away instead.