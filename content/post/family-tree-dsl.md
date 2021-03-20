---
author: "Andy Dyer" 
title: "Building a Family Tree DSL with Jetpack Compose syntax"
date: 2021-03-21T20:05:20+01:00
draft: false
categories: ["Programming", "Kotlin", "Jetpack Compose"]
---

> Originally published on [ProAndroidDev](https://proandroiddev.com/building-a-family-tree-dsl-with-jetpack-compose-syntax-fe652ce6cb0f)

### Why build a DSL?

Ever since first getting into Kotlin, I've known it has a few things that makes building domain-specific languages (DSLs) easier. I've read the mind-expanding [type-safe builders](https://kotlinlang.org/docs/type-safe-builders.html) guide for building a DSL for HTML, but until recently I hadn't found a good use case for building a DSL of my own.

[My team at Zalando](https://jobs.zalando.com/en/jobs/2986308-senior-android-engineer-appcraft-mobile) maintains server-driven UI libraries for Android & iOS that power completely dynamic screens such as home, brand homes, collections, and various landing pages in the [Zalando fashion](https://play.google.com/store/apps/details?id=de.zalando.mobile) [store apps](https://apps.apple.com/de/app/zalando-fashion-shopping/id585629514?l=en). Often, we need to build mock responses during development for new features that don't have a backend implementation yet or models for unit tests. In the past, we'd grab chunks of JSON from API responses and edit them, but navigating a wall of text is tedious and editing JSON is far less fun than writing code.

Even once I had the use case, it took spending some time with [Jetpack Compose](https://developer.android.com/jetpack/compose) for everything to finally "click". I noticed how the `Column` composable used a `ColumnScope.()` receiver for its lambda block (as do most composables, it turns out). It was this detail and a quick look at the source code that helped me realize that the receiver was the piece I was missing to build a nice, clean DSL with the same syntax as Compose.

To keep things simple, we'll see how to apply these same ideas to building a family tree DSL.

### What is the desired syntax?

Our goal is to build a DSL with a syntax similar to Jetpack Compose. This means [functions with named arguments](https://kotlinlang.org/docs/functions.html#named-arguments) and a [trailing lambda for the last parameter](https://kotlinlang.org/docs/lambdas.html#passing-trailing-lambdas) that uses a [receiver](https://kotlinlang.org/docs/lambdas.html#function-literals-with-receiver) to scope the functions called inside:

{{< gist abdyer ad110ca703578c68782c0cd1bb0fe3e0 >}}

That last sentence contained a lot of concepts in a few words, so let's take a quick look at each.

#### Functions with named arguments

Everything in Compose is a function and we understand what named arguments are, but the combination of these allows us to pack all the information we need about an entity in a well-known unit. The function name tells us what the item is while its named arguments & values tell us its most important details. Default values can be provided to allow flexibility while optimizing for the most common use case.

With no prior knowledge of Compose, we immediately know from the brief code block above that we're dealing with a column that has `16 dp` of padding around its contents. We can also surmise that a `Modifier` must be how style related attributes are specified.

#### Trailing lambdas

Functions and named arguments describe an entity, but UIs (and family trees) also need a way to indicate hierarchy. Kotlin's trailing lambdas allow us to pass a function as the last argument to a function outside the closing parenthesis which gives us a nice curly brace wrapper around another function. But UI and family tree parents can both have multiple children, so a single function does not quite give us the syntax we want. Maybe there's a way to scope multiple function calls and have a single class or object "receive" them...

#### Receivers & scope

By declaring our trailing lambda as `A.(B) -> C`, we scope the functions called inside (`B`) to class `A`, returning type `C` (which can also be `Unit`). It's also a good idea to create our own [`@DslMarker` annotations](https://kotlinlang.org/docs/type-safe-builders.html#scope-control-dslmarker) and apply them to our DSL classes.

### Family tree DSL design considerations

Armed these concepts to achieve a Compose like syntax, we can turn our attention to modeling a family tree. A family tree is essentially a recursive hierarchy of partnerships and children. Our family tree DSL will behave as follows:

1. A family groups a member and partnerships.
1. A partnership is a relationship between two people, with one of them being the family member. Each partnership can have one or more children. Each of those children can have their own family and so on.

### Building the DSL

Time for the code! Let's go through each of the classes and functions that compose our DSL. We'll start with the people before moving on to the partnerships and finally the family to create the family tree hierarchy.

#### People

Just like [Soylent Green](https://www.youtube.com/watch?v=Xs-gocza6t8), families are made out of people! We can use a [`sealed class`](https://kotlinlang.org/docs/sealed-classes.html) to define a `Person` with a `name`. A `Person` can either be `Unidentified` (we'll see why this is useful in the next post) or a `FamilyMember` where more details about the person may be known:

{{< gist abdyer a268367c77800cd4c5340726eb847032 >}}

#### Partnerships

To model a relationship between two people and their children, we'll need a `Partnership`. Below we define a custom `@DslMarker` and a `Partnership` class with a `child()` function for adding `children`:

{{< gist abdyer 51dd031ef4d6e434aaa55d8de7c67f26 >}}

The final `block` parameter passed to the `child()` function is our first use of the trailing-lambda-scoped-to-a-receiver secret sauce. We'll dig into the `Family` class next, but before we move on, there are a couple more things to note:
- The first is how `block()` is called on the `Family` instance after we create it. This has the effect of executing all of the function calls inside the lambda on this `Family`, thereby adding their partnerships & children (and children's families, etc.).
- The other is how the `Family` is added to the `Partnership` children internally. This allows us to build the list of`children` for subsequent inspection and still return the new `Family` instance for further tree construction as we'll see next.

#### Family

After defining another custom `@DslMarker`, we can create a `Family` class that groups a `FamilyMember` and `Partnership` list. In general, it's the same pattern we saw in the `Partnership` class above, only this time we're adding to a `partnerships` list with a `partner()` function. There are two here for convenience, making it possible to add a `Partnership` with two different sets of parameters (also useful in the next post):

{{< gist abdyer 1bfec8b9780d13b93c36700f10c8f67a >}}

#### One more function

We need one more function to tie everything together. It will create our root `FamilyMember` and `Family` instances, call the family `block()`, and return the resulting `Family`:

{{< gist abdyer ce812fa7424413b6bfdb78d54ac88a7e >}}

### Using the DSL

Now we can use the DSL to create family trees like the one below with a Compose like syntax:

{{< gist abdyer c838ebb995850c1fab59f43c9cefeb35 >}}

### Conclusion

In this post, we have seen how to combine [functions with named arguments](https://kotlinlang.org/docs/functions.html#named-arguments), [trailing lambdas](https://kotlinlang.org/docs/lambdas.html#passing-trailing-lambdas), and [receivers](https://kotlinlang.org/docs/lambdas.html#function-literals-with-receiver) to build a family tree DSL with a syntax inspired by Jetpack Compose.

I decided to save the analysis of the plot-spoiling family trees I built from [Netflix's Dark series web site](https://dark.netflix.io) for a follow-up post, but they're in [the same repo](https://github.com/abdyer/family-tree-dsl) as the code. I think the time travel aspect makes things even more interesting and modeling "the knot" using the DSL was a fun little project. I came away from it with a slightly better understanding of the story as well as an even deeper appreciation for its hidden genius. 

Thanks for reading and stay tuned for the Dark follow-up!
