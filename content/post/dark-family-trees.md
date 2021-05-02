---
author: "Andy Dyer"
title: "Spoiling the Dark plot with Kotlin"
date: 2021-05-01T09:01:11+01:00
draft: false
categories: ["Programming", "Kotlin", "Random"]
---

### Intro

As promised in [my last post](https://andydyer.org/blog/2021/03/21/building-a-family-tree-dsl-with-jetpack-compose-syntax/), it's time to use our family tree DSL to spoil the plot of [Netflix's Dark series](https://dark.netflix.io)! 😈

To quickly recap, we have a small but functional (my puns are always intended) DSL for building a family tree in Kotlin:

{{< gist abdyer c838ebb995850c1fab59f43c9cefeb35 >}}

### The Dark family trees in code

The family trees are of course the focus of the series as they are revealed to us piecemeal; episode by episode, season by season. After watching it, I was overwhelmed with all the details and quickly gave up trying to fit it all together. Sure, I remembered some of the bigger points, but it makes a lot more sense after stepping through the [Dark website](https://dark.netflix.io) to build the family trees below:

{{< gist abdyer 75384e70e24da65d6c8a1ca3dd29f7aa >}}

Let's make a few observations about the above before using the power of code to uncover a few more:

1. The Nielsen tree is the deepest, which highlights the family's central role in the plot.
1. As you should know if you're reading this <u>**spoiler post**</u>, the years many people were born do not line up with their partners or children in many cases. We'll get to that.
1. Hannah Kahnwald appears in multiple places.

### Analyzing the Dark family trees

Now that we have these family trees, what can we confirm about the plot from the data?

#### Who are the time travelers?

Since we have a hierarchy and birth years, we can traverse the tree and compare them to identify the time travelers. For our purposes, a time traveler is someone who is more than 30 years older than their partner or 70 years older than their children (Egon didn't time travel, but Hannah did before their lovechild was born).

The code below is one way to implement this logic, but please let me know (or [open a PR](https://github.com/abdyer/family-tree-dsl/pulls)) if you have suggestions for improving something:

{{< gist abdyer b2dccdaa6d08711820f8be174d1984bc >}}

#### Who is their own ancestor?

Such a weird thought, right? Once again, I'm sorry to be the one to spoil this for you if you missed this juicy bit of the plot, but a couple people in the Dark family trees are their own ancestor. 🤯 While the approach for identifying time travelers involved recursively mapping the tree, we'll need to collect the list of ancestors as we go to identify the self ancestors. We can do with with a `data class`, the [`fold` function](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/fold.html), and more recursion:

{{< gist abdyer 386a793ca6b550a185b9cf033f567b7a >}}

#### The big reveal

With the `main()` function below, we'll print the answers to our time traveler and self ancestor questions:

{{< gist abdyer 7aa1b48481fc57b1051995c1bc0daa09 >}}

Which gives us:

```bash

    Time Travelers:
    Mikkel Nielsen
    Hannah Kahnwald
    Silja Tiedemann
    Bartosz Tiedemann
    Elisabeth Doppler
    Noah Tiedemann

    Self Ancestors:
    The Unknown
    Charlotte Doppler

```

### Conclusion

Now we have sufficiently spoiled the Dark plot with code! 🤓 You can find the full code [here on GitHub](https://github.com/abdyer/family-tree-dsl).

Please reach out if you'd like to quibble about anything you think I may have gotten wrong. And if anyone has the galaxy brain to correlate the family trees above with the image below, please be my guest. I suspect the loops match up with time travelers and partnerships in the tree, but did not come up with any defensible theories.

<img src="/images/dark-loops.png" alt="Dark family tree loops" style="width: 80%; margin-left: auto; margin-right: auto; display: block" />
