---
type: post
title: "Running Shell Commands with Gradle and a Better Way to Delombok"
date: "2015-01-22"
comments: true
categories: [Android, Gradle, Lombok, Javadoc]
---

A few months ago, I [posted](http://andydyer.org/blog/2014/09/29/delombok-and-javadoc-with-gradle/) about how to use Gradle tasks to "[delombok](http://projectlombok.org/features/delombok.html)" code using Lombok annotations before generating Javadocs. My solution for running the delombok task used Ant and was based on what I found after the requisite Google & StackOverflow searching. This worked just fine until Android Studio 1.0 and the associated Gradle build tools were released at the end of the year.

The crux of the problem appeared to be a change in the way dependencies are merged during compilation. Countless "package does not exist errors" were causing the delombok task to fail. I first attempted to solve this by changing the way the classpath was built in the task. While I was able to reduce the number of errors, I didn't succeed in completely fixing the problem. All the while, running the delombok task directly from the command line with `java -jar build-libs/lombok.jar delombok src -d build/src-delomboked` ran successfully.

Eventually, I decided to [do the simplest thing that could possibly work](http://c2.com/xp/DoTheSimplestThingThatCouldPossiblyWork.html). This led me to learning how to use the [Exec](http://www.gradle.org/docs/current/dsl/org.gradle.api.tasks.Exec.html) command to run shell commands with Gradle tasks. The end result is below. It's much more succinct and no longer subject to changes in Android Studio's build process. The only non-obvious thing is that each part of the command must be quoted separately and comma-delimited.

```groovy
task delombok(type: Exec) {
    commandLine 'java', '-jar', 'build-libs/lombok.jar', 'delombok', 'src', '-d', 'build/src-delomboked'
}
```

I hope this saves someone some time.