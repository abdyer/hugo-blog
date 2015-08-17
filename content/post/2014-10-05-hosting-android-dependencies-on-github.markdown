---
layout: post
title: "Hosting Android Dependencies on GitHub"
date: "2014-10-05"
comments: true
categories: [Android, Gradle, Maven, GitHub]
---

As I mentioned in my [last post](http://andydyer.org/blog/2014/09/29/delombok-and-javadoc-with-gradle/), I'm developing an SDK at work. The libraries I use most frequently in my apps are all included as Maven dependencies. Adding a line to my `build.gradle` file is much preferred to downloading a JAR file. To make the SDK as easy as possible for developers to include in their projects, I wanted to deliver it the same way.

[Maven Central](http://central.sonatype.org/) is the de facto repository for open source library hosting. Since the SDK I'm developing is part of a product and as such will not be open source, I needed to find another place to host the binary that would still allow it to be included via Maven.

Libraries hosted on Maven Central are included in `build.gradle `with the following:

```groovy
repositories {
    mavenCentral()
}

dependencies {
    compile 'com.squareup.retrofit:retrofit:1.6.1'
}
```

Including another Maven source is as simple as adding its URL to the `repositories` block and then including the group/artifact/version string as normal in the `dependencies` block:

```groovy
repositories {
    mavenCentral()
    maven { url "https://raw.github.com/username/my-super-cool-sdk/master" }
}

dependencies {
    compile 'com.squareup.retrofit:retrofit:1.6.1'
    compile 'com.username:my-super-cool-sdk:1.0.0'
}
```

But how do we actually prepare a library for Maven? This can be done in Gradle with the Maven plugin:

```groovy
apply plugin: 'maven'

uploadArchives {
    repositories.mavenDeployer {
        def deployPath = file(getProperty('aar.deployPath'))
        repository(url: "file://${deployPath.absolutePath}")
        pom.project {
            groupId 'com.username'
            artifactId 'my-super-cool-sdk'
            version "1.0.0"
        }
    }
}
```

One thing to note is the call to `getProperty('aar.deployPath')` above. This reads the value of the `aar.deployPath` property in my `gradle.properties` file. The library project is in a separate GitHub repository from the one where the SDK binary is hosted, so the deploy path property provides the relative path to the local destination directory:

```groovy
aar.deployPath=../../../../my-super-cool-sdk
```

Once everything is in place, running `gradle uploadArchives` exports the AAR binary and necessary POM/XML files to the target directory. From there, all that's left to do is commit the files and push them to the SDK repository on GitHub.