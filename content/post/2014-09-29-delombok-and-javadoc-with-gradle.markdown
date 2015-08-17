---
layout: post
title: "Delombok and Javadoc with Gradle"
date: "2014-09-29"
comments: true
categories: [Android, Gradle, Lombok, Javadoc]
---

I recently had my first experience working with Javadoc to generate documentation for an SDK I've been developing at work. In general, I'm in the "[clean code doesn't need comments](http://blog.codinghorror.com/coding-without-comments/)" camp, but SDKs tend to be a limited view into a larger abstraction, so good documentation is a necessity.

Javadoc has been around since the introduction of the Java language, so I won't include a primer here. If you want to learn more, [Oracle has you covered](http://www.oracle.com/technetwork/java/javase/documentation/javadoc-137458.html).

After adding the necessary documentation comments to the public-facing portions of the SDK, I naturally wanted a Gradle task to easily generate the documentation. The following will add tasks to generate Javadocs for each build type:

```groovy
android.libraryVariants.all { variant ->
    task("generate${variant.name.capitalize()}Javadoc", type: Javadoc) {
        description "Generates Javadoc for $variant.name."
        source = 'src/main/java'
        ext.androidJar = "${android.sdkDirectory}/platforms/${android.compileSdkVersion}/android.jar"
        classpath = files(variant.javaCompile.classpath.files) + files(ext.androidJar)
        options.links("http://docs.oracle.com/javase/7/docs/api/");
        options.links("http://d.android.com/reference/");

        // exclude generated files
        exclude '**/BuildConfig.java'
        exclude '**/R.java'

        // exclude any internal packages
        exclude '**/com/acme/sdk/api/**'
    }
}
```

After adding the above to your `build.gradle` file, generate your Javadocs from the command line with `gradle generateReleaseJavadoc`. If you're not using any annotation libraries to generate the mundane/boilerplate code in your application, this should be all you need. However, I highly recommend exploring how you can take advantage of some of these libraries.

An annotation library I use in almost every project is [Lombok](http://projectlombok.org/). Be forewarned that the unappealing web site belies the power of this library. As you can see from the [feature list](http://projectlombok.org/features/index.html), it includes annotations for generating getters/setters, null checks, and much more. You'll want to install the [Lombok Plugin](http://plugins.jetbrains.com/plugin/6317) for IntelliJ to eliminate misleading syntax errors when using these annotations in Android Studio.

Assuming I've convinced you to try Lombok, you'll eventually find that Javadoc comments on `@Getter` annotated fields are missing in the generated documentation. This is because the annotated code is replaced with the generated code during compliation and Javadoc is run against the compiled code.

To fix this, we need to "[delombok](http://projectlombok.org/features/delombok.html)" our code before running Javadoc on it. This involves running the `delombok` command on the original source, copying the resulting code into a separate directory, and then running Javadoc on the delomboked directory. With Gradle, this is most easily done by adding a dependent delombok task to the Javadoc task above.

You'll also need to include `lombok.jar` in your `libs` directory in order to run the `delombok` command. I include the Lombok library in my project via a Gradle dependency, but the task didn't seem to find the command without the JAR file. I don't have any other JAR files in my `libs` directory and don't include it in my build, so this was a pretty easy workaround. You can certainly put the JAR in another directory within your project and change the classpath in the task if needed.

Once this has been done, the complete script is as follows:

```groovy
def srcJava = 'src/main/java'
def srcDelomboked = 'build/src-delomboked/main/java'

task delombok {
    inputs.files file(srcJava)
    outputs.dir file(srcDelomboked)

    doLast {
        FileCollection collection = files(configurations.compile)
        FileCollection sumTree = collection + fileTree(dir: 'bin')

        ant.taskdef(name: 'delombok', classname: 'lombok.delombok.ant.DelombokTask', classpath: 'libs/lombok.jar')
        ant.delombok(from:srcJava, to:srcDelomboked, classpath: sumTree.asPath)
    }
}

android.libraryVariants.all { variant ->
    task("generate${variant.name.capitalize()}Javadoc", type: Javadoc) {
        description "Generates Javadoc for $variant.name."
        setDependsOn(['delombok'])
        source = srcDelomboked
        ext.androidJar = "${android.sdkDirectory}/platforms/${android.compileSdkVersion}/android.jar"
        classpath = files(variant.javaCompile.classpath.files) + files(ext.androidJar)
        options.links("http://docs.oracle.com/javase/7/docs/api/");
        options.links("http://d.android.com/reference/");

        // exclude generated files
        exclude '**/BuildConfig.java'
        exclude '**/R.java'

        // exclude any internal packages
        exclude '**/com/acme/sdk/api/**'
    }
}
```

Running `gradle generateReleaseJavadoc` will now delombok your code and then run Javadoc on the output.

Thanks for reading!