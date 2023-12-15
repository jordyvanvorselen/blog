---
title: "The bug that eluded me for years"
draft: true
date: 2023-11-15T10:00:00
tags:
  - Software Engineering
  - Bugs
  - Debugging
  - Thread Safety
categories:
  - Ruby
description:
  "During a multi-year project there was a bug that kept popping up, but we could not find the
  culprit. We traced it back to a single class but could not figure out why the value was set to
  something unexpected by reading the code."
images: []
mermaid: true
resources:
  - name: "featured-image"
    src: "featured-image.png"
---

<!--more-->

#### The problem

We were building an enterprise-grade Rails application (yes, it
[scales just fine](https://twitter.com/natfriedman/status/1404835709278580739)), but a very annoying
problem kept popping up. We used the popular [Pundit](https://github.com/varvet/pundit) gem to
create policies which make sure only users with the correct role could access certain functionality.
Among some other roles, we had a role called `public` that allowed a user without a role to view a
small subset of the data. Most users using the platform had a specific role however. This setup was
working just fine most of the time, but a few times a year weird issues started popping up.

#### Initial investigation

Users would report not having access to certain data they should have access to. After checking
screenshots and investigating together with some users, we saw that the user had been given the role
`public`. Multiple engineers of my team failed to reproduce the issue, myself included. The weird
part was that the issue also seemed to happen for multiple users on a specific day, but it would be
gone the next day.

We spent a lot of time digging and debugging the authorization code, but could not find the issue.
We decided to leave it as it was working as expected again.

#### A year later

Once again, our users were having the role-switching problem.

The users and the product owner were understandably getting impatient. So we dove into it again,
_literally_ checking the code line by line for a spot where it would fail.

After a few long days we managed to trace the problem into a single spot. It looked a bit like this:

```ruby
class Authoriser
  include Singleton

  ...

  def roles(group = group_resolver.group)
    return default_role if group == 'default'

    return group
  end
end
```

Even though we could still not reproduce the issue, this method **had** to be the problem. It
returned the wrong result, but we didn't understand why. To us, it looked like this method was fine
logic wise.

However, now we understood how the authorization was supposed to work:

<!-- prettier-ignore -->
{{< mermaid >}}
sequenceDiagram
    title How it was supposed to work
    participant User
    participant Endpoint
    participant Pundit
    participant Singleton

    User->>Endpoint: Does API call
    Endpoint->>Pundit: Is user authorized to access?
    Pundit->>Singleton: What's the user's role?
    Singleton->>Singleton: Caches the role
    Pundit->>Singleton: Fetches role from Singleton
    Pundit-->>Endpoint: Yes, user can access

{{< /mermaid >}} <br/>

So we wrote our findings in the ticket, had another awkward chat with management and told them that
we once again had to give up on this issue.

#### And another year passed

I guess you can imagine what will happen by now.

It came back once again.

So this time I was determined to fix this once and for all. It was costing my client loads of money
and was causing frustration for our users.

The weird thing was that this time the problem was paired with another issue we had during **heavy
load**. I could've kept digging in the code and trying to find the issue in the logic, but I chose
to take a step back and started over. I knew we needed to reproduce this issue to find it, but had
no clue how to do this.

So I started thinking about the problem for a while, without cluttering my mind with code. I
realized that the problem occurred at the same time as the system was under heavy load.

Heavy load.

Let's try to add heavy load to my dev server.

So, what did I do? I created a `api_spammer.rb`. :wink: This beautiful script was basically a while
loop that kept spamming our API (which did not have rate limiting at the time).

I ran the script, opened my browser and tried to reproduce it again...

And I failed to reproduce it again. Software kept working as expected. It was starting to get
painful by now.

So in my frustration I opened 10 tmux splits in my terminal. I ran the spammer script in 10
terminals at the same time.

Opened the browser again. And ho and behold:

_it finally worked_.

I had never been so happy before to _see_ a bug in our software.

#### Fixing the issue

So now comes the fun part. The good thing was that I could now check if my 'fixes' actually fixed
anything _without_ deploying to prod and having to wait for another year.

The weird thing was that load should not impact the logic as a Rails endpoint is _stateless_.

The problem here was however, that someone used a Singleton to make this part of the system
_stateful_. And on this day I learned that a Singleton in Ruby is not threadsafe by default.

So what happened when there was a lot of traffic was this:

<!-- prettier-ignore -->
{{< mermaid >}}
sequenceDiagram
    title How it actually worked
    participant User 1
    participant User 2
    participant Endpoint
    participant Pundit
    participant Singleton

    User 1->>Endpoint: Does API call
    User 2->>Endpoint: Does API call
    Endpoint->>Pundit: Is user 1 authorized to access?
    Endpoint->>Pundit: Is user 2 authorized to access?
    Pundit->>Singleton: What's user 1's role?
    Pundit->>Singleton: What's user 2's role?
    Singleton->>Singleton: Caches the role of user 1
    Singleton->>Singleton: Caches the role of user 2 (public)
    Pundit->>Singleton: Fetches role from Singleton
    Pundit->>Singleton: Fetches role from Singleton
    Pundit-->>Endpoint: No, user 1 cannot access
    Pundit-->>Endpoint: No, user 2 cannot access

{{< /mermaid >}} <br/>

The request of `User 1` was handled by another web worker thread than the request of `User 2`.

So how can we fix this?

Instead of using Singletons to share stateful data, I added the
[request_store](https://github.com/steveklabnik/request_store) gem. This adds a thread safe way to
define a global variable.

After using `request_store` to share the user's group to the endpoint, the problem was solved.

#### Learnings

Ruby Singletons are not threadsafe by default. Rails takes a lot of thread-safety concerns away
using it's [Executor](https://guides.rubyonrails.org/threading_and_code_execution.html)
implementation, but it's not _always_ going to do the work for you. So it's good to at least know of
it's existence and how it works on a high level.

Also, don't give up on trying to reproduce a bug. It makes it significantly easier to solve it than
trying to solve it by reading and changing the code. Doing bugfixes without being able to reproduce
is guesswork.
