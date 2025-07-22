---
title: "Factories: Clean Test Data Without the Pain"
draft: false
date: 2025-07-22T15:00:00
tags:
  - Best Practices
  - Software Engineering
  - Abstractions
categories:
  - Best Practices
description:
  "Tired of brittle test setups and copy-pasted builders? Learn how to manage unit test data with maintainable, flexible factories. This guide walks through test factory patterns in Java (and other languages), helping you avoid flaky tests, reduce duplication, and write more resilient test suites."
images: []
resources:
  - name: "featured-image"
    src: "featured-image.png"
---

<!--more-->

Many people treat test code as a second class citizen. But when working in high-performance software
teams that follow the associated best practices like TDD, this is not a viable option for long term
maintainability. When managing a few tests, using the builder pattern or directly instantiating classes
feels like a viable approach. When the test suite grows this becomes duplication with a little variety.

And then you want to change something in the domain classes.

:bomb: boom :exploding_head:

Have fun updating all your test files.

Someone in your team _might_ introduce a central class with some static methods that generate the most
common testdata (shout out to the 12 overload variations of `createUser()`). But even this is a big
waste of time to maintain as it quickly grows complex.

Throw some flaky tests in the mix and then all speed is gone.

Poorly managed testdata doesn't just slow you down - it chips away at your confidence to change code.

#### The better solution

Stop using the 2000 long helper class. Stop copy-pasting direct instantiations over all your test files.
No more direct builder usage with the inevitable null pointer exceptions.

Start using _factories_. :sparkles:

A factory is the central place where the testdata lives for our domain objects. Let's say we have a
simple `User` domain object.

_the example is in Java, but the ideas translate to almost any language_

```java
@Entity
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Table(schema = "public")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private String email;

    private String hashedPassword;

    private Role role; // GUEST, MEMBER or ADMIN

    // The method we are going to test
    public boolean readOnly() {
        return role == Role.GUEST;
    }
}
```

We will probably need a `User` instance in a lot of tests. It's better to create a **central place** where
we save the _default_ testdata for a `User`. Then we only have to maintain that in one place, and we can
re-use it everywhere.

##### Iteration 1 - The simplest version

Let's create the simplest factory:

```java
public class UserFactory {
    public static User generate() {
        return User.builder()
            .id(1)
            .email("jordyvanvorselen@gmail.com")
            .hashedPassword("325ada3@#@#dsrs4a23qaaea")
            .role(Role.MEMBER)
            .build();
    }
}
```

As simple as that? Yes, this will work. It's probably one of the simplest implementations of the factory
we can make up. But there are a few downsides. Let's write a test using this factory and gradually improve
upon it.

```java
class UserTest {

    private User user;

    @BeforeEach
    void beforeEach() {
        user = UserFactory.generate();
    }

    @Nested
    class ReadOnly {

        @Test
        void returnsTrueWhenGuest() {
            final var result = user.readOnly();

            assertThat(result).isTrue();
        }
    }
}
```

Alright, that works. But this only tests one of multiple code paths of the `readOnly` method. We also
need to test the non-happy flow, where the user is a `Role.MEMBER` or `Role.ADMIN`.

##### Iteration 2 - Override single properties

It would be very convenient if we can overwrite _just enough_ in the test file. And the best thing is
that our factory can support that! Let's change it so that we can do this:

```java
public class UserFactory {
    public static User.UserBuilder generate() { // because the User entity has the @Builder annotation, we can return the UserBuilder instead of the actual instance
        return User.builder()
            .id(1)
            .email("jordyvanvorselen@gmail.com")
            .hashedPassword("325ada3@#@#dsrs4a23qaaea")
            .role(Role.MEMBER); // we no longer call .build() in here, we will delegate that responsibility to the test itself.
    }
}
```

Alright, 2 very simple changes - but _very_ important ones. These allow us to override any property of a user,
while keeping the default values in the other properties:

```java
final var user = UserFactory.generate().build();
user.getEmail(); // "jordyvanvorselen@gmail.com"
user.role(); // "Role.MEMBER"

final var otherUser = UserFactory.generate().role(Role.ADMIN).build();
user.getEmail(); // "jordyvanvorselen@gmail.com"
user.role(); // "Role.ADMIN" <-- overridden!
```

Now, we can create the other tests:

```java
class UserTest {

    private User user;

    @BeforeEach
    void beforeEach() {
        user = UserFactory.generate().build();
    }

    @Nested
    class ReadOnly {

        @Test
        void returnsTrueWhenGuest() {
            final var result = user.readOnly();

            assertThat(result).isTrue();
        }

        @Nested
        class WhenMember {
            @BeforeEach
            void beforeEach() {
                user = UserFactory.generate().role(Role.MEMBER).build();
            }

            @Test
            void returnsFalse() {
                final var result = user.readOnly();

                assertThat(result).isFalse();
            }
        }

        @Nested
        class WhenAdmin {
            @BeforeEach
            void beforeEach() {
                user = UserFactory.generate().role(Role.ADMIN).build();
            }

            @Test
            void returnsFalse() {
                final var result = user.readOnly();

                assertThat(result).isFalse();
            }
        }
    }
}
```

Nice! :sunglasses: Now we have 100% test coverage :+1:

But using static testdata does not feel very identical to what our production
environment does. And as we want to catch as many bugs as possible with realistic
data, we can improve our factory to use generated fake data.

##### Iteration 3 - Generated realistic testdata

In java we can use the [java-faker]() package to generate all kinds of testdata. In
other languages `faker` is also available.

```java
public class UserFactory {
    public static User.UserBuilder generate() {
        return User.builder()
            .id(faker.random().numberBetween(0, 500)) // generates a random id
            .email(faker.internet().emailAddress()) // generates a random email instead of the hardcoded one
            .hashedPassword("325ada3@#@#dsrs4a23qaaea")
            .role(Role.MEMBER);
    }
}
```

There are plenty of possibilities in faker. From random names to star wars themed company names, everything
is possible.

##### Iteration 4 - Avoid flakyness

But a few months later, we added tests with multiple user instances created by the factory. And they
are green for a long time. But all of a sudden - one of these tests fails in CI... :x:

Huh? We can't explain it, we didn't even touch the code.

Re-run it...

:white_check_mark: it passes

Oh no, now our test is flaky. So what is the problem?

```java
    .id(faker.random().numberBetween(0, 500)) // :thinking_face: :thinking_face:
```

The id is not unique... When using the factory multiple times, there is a very slight chance that the
same id will be generated by faker. As our id is unique, it results in weird behavior and flaky tests.

Let's fix this!

We can use `AtomicInteger` and incorporate it into any field that should be unique:

```java
public class UserFactory {
    private static final AtomicInteger counter = new AtomicInteger(0);

    public static User.UserBuilder generate() {
        return User.builder()
            .id(counter.getAndIncrement()) // generates a UNIQUE id
            .email(faker.internet().emailAddress())
            .hashedPassword("325ada3@#@#dsrs4a23qaaea")
            .role(Role.MEMBER);
    }
}
```

This will ensure that these fields are _always_ unique.

Run CI :wink:

:white_check_mark: test passes

And the best part of it all is that we can use this atomic integer counter in any field that needs to
be unique. Just concat it behind a name and it's unique.

##### Conclusion

So, next time you're tempted to copy-paste a builder or instantiate an object directly in your test - don’t.

_Create a factory instead._

It centralizes your testdata, improves readability, reduces duplication, and makes
your tests a lot easier to maintain. You’ll create a reliable foundation for your entire test suite. Your future
self and your team will thank you.
