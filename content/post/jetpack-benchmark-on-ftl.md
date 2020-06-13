---
title: "Jetpack Benchmark on Firebase Test Lab"
date: 2020-03-06T10:30:55+01:00
draft: false
categories: ["Android"]
---

As previously outlined in my [talk at Mobius Conference](http://andydyer.org/blog/2019/12/22/appcraft-faster-than-a-speeding-release-train/) in Saint Petersburg last year, our team at Zalando is building a server-driven UI platform we call AppCraft. AppCraft allows us to add or update screens in [our app](https://play.google.com/store/apps/details?id=de.zalando.mobile) with only server changes instead of the normal app release constrained way of doing things.

On the client side, we are building every screen the same way: fetch layout & data from the server, parse it, and render it.

Since the API response time is mostly our of our control, this leaves:

1. JSON parsing
1. Building the UI component tree (we currently use [Litho](https://github.com/facebook/litho) for this)
1. Rendering

The size of the response JSON and the complexity of the layout varies per screen, so benchmarking the above steps for each screen (we currently have three of them live in the app!) seemed like a good way to start tracking the performance of our library and its change over time.

### Jetpack Benchmark library

My goal here is to share how to get benchmarks running on Firebase Test Lab, so I won't spend long telling you about the Jetpack Benchmark library itself; Google has [already done that](https://developer.android.com/studio/profile/benchmark). However, there are a couple of things worth mentioning.

#### Respect your tools

Would you carelessly grab a sharp knife by the blade? Probably not. The same applies here. Any good load testing or benchmarking tool can become a denial-of-service attack if (ab)used (in)correctly. The Jetpack Benchmark library does whatever you tell it to over and over as quickly as possible. Therefore, you **do not** want to make API requests to production servers (or probably any server) in your tests. Benchmark tests should run on isolated portions of your code that you want to profile. Mock API responses or any dependencies that you don't want to be affected by multiple rapid iterations.

#### Project setup

The [official docs](https://developer.android.com/studio/profile/benchmark) explain how to add the benchmark library to your project and important details such as the need to disable debugging for your app while benchmarking for accurate test results. They also mention that tests go in the benchmark module and the target code must be in a library.

In order to configure your app for running benchmarks (loading mocked responses from local files instead of hitting live endpoints, for example), it may also be helpful to add a custom `Application` and `Activity` to the target library's `debug` source set.

### Benchmarks on CI

Chris Craik from Google published [this article](https://medium.com/androiddevelopers/fighting-regressions-with-benchmarks-in-ci-6ea9a14b5c71) last fall about how they're comparing benchmark results over time to spot regressions. The article has a lot of great detail about the challenges of doing such a comparison. It also inspired me to try running benchmarks on Jenkins and Firebase Test Lab (FTL), which we use for CI on our project.

#### Firebase Test Lab, Flank & Fladle

If you're here to learn about running benchmarks on FTL, chances are you're familiar with [Flank](https://github.com/Flank/flank) and [Fladle](https://github.com/runningcode/fladle). We use these to configure the devices to use for each group of tests (screen shot tests, benchmarks, other UI tests) and how the tests run (in parallel, retries on failure, etc).

Unfortunately, as I learned, configuring the benchmark library on Jenkins and FTL comes with a couple of interesting challenges:

1. FTL runs tests on physical devices that are not directly attached to the build machine. This means the benchmark library's default behavior of copying JSON results from the device to the build machine won't work.
1. The docs for [running benchmarks on CI](https://developer.android.com/studio/profile/run-benchmarks-in-ci) tell us to set the `androidx.benchmark.output.enable` environment variable to `true` via the command line. It turns out environment variables with dots in them are not valid on FTL. We tried escaping them in various ways, but were unsuccessful.

### Benchmarks on FTL & Jenkins

To solve our two challenges, we'll need a few ingredients. We'll look at each and the relevant code/build script snippets for them below:

1. A custom `AndroidBenchmarkRunner` to output results JSON
1. A `@BenchmarkTest` annotation to group our benchmark tests
1. Fladle config to specify a physical device, copy results from the device
1. Run benchmarks with Flank, publish results JSON to artifacts in Jenkins pipeline script

#### Custom benchmark runner class

We need a custom `Application` class to set up the appropriate mocks for our benchmark tests. With a custom `AndroidBenchmarkRunner` class, we can create an instance of our `BenchmarkApp` class. We can also add the problematic environment variable to enable JSON output in the `Bundle` argument to the runner's `onCreate()` function:

{{< gist abdyer 053a04804d2d1cafcafd8ba7f54a8877 >}}

Then, update the benchmark module's `build.gradle` to use the custom runner:

{{< gist abdyer 0369db76e4c3ec9e1379e72198deee1d >}}

#### Benchmark test annotation

Benchmark tests will likely run separately from other UI tests since they require (or are at least strongly encouraged to be run on) a physical device and don't need to run as often as other types of tests. Initially, we scheduled our benchmarks to run weekly. Marking the tests with a custom annotation will allow us to filter them in our Fladle config:

{{< gist abdyer b65d0e57b871455cebf06c0849952408 >}}

#### Fladle config

Next, configure [Fladle](https://github.com/runningcode/fladle) to run our tests on a physical device and download the results:

- `devices` - specifies the FTL device configuration options (learn more about available configurations [here](https://firebase.google.com/docs/test-lab/android/available-testing-devices))
- `directoriesToPull` and `filesToDownload` - grabs everything in the device's `Download` directory
- `testTargets` - filters this task to only benchmark tests tagged with our `@BenchmarkTest` annotation

{{< gist abdyer 232ec78ee0f640bf9d332ba80a3d30bd >}}

#### Running benchmarks & saving results

Finally, we're ready to run our benchmark tests with [Flank](https://github.com/Flank/flank). By convention, a Gradle task is created from our Fladle config, the name of which is prepended with `runFlank` to become `runFlankBenchmarkTests` in our case. In our Jenkins pipeline script, we include the module name to run the benchmark tests with `benchmark:runFlankBenchmarkTests`.

After the tests run, all that's left is to use [`archiveArtifacts`](https://jenkinsci.github.io/job-dsl-plugin/#method/javaposse.jobdsl.dsl.helpers.publisher.PublisherContext.archiveArtifacts) from the Jenkins Job DSL to add the results to the job artifacts:

```javascript

  archiveArtifacts allowEmptyArchive: true, artifacts: '**/*.json'

```

### Conclusion

As we've seen, we can use the Jetpack Benchmark library to benchmark our code on Firebase Test Lab by adding a few missing ingredients. Weekly benchmark results will allow us to monitor the performance of our rendering library over time, but automatically parsing & charting the results and spotting regressions would be even more powerful. If you have suggestions for Linux friendly plotting libraries, please let me know!

And if you're interested in working on interesting problems like this while building Android apps used by tens of millions of users, check out our [open positions](https://jobs.zalando.com/en/tech/jobs)!
