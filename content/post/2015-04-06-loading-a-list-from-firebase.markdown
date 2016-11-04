---
layout: post
title: "Loading a List of Objects from Firebase"
date: "2015-04-06"
comments: true
categories: [Android, Apps]
---

I've been working with [Firebase](https://www.firebase.com/) lately in preparation for adding data synchronization to my [Music Library](https://play.google.com/store/apps/details?id=com.dandydev.medialibrary) app. Their [docs](https://www.firebase.com/docs/android/quickstart.html) did a great job of getting me set up. Using the [sample app](https://github.com/firebase/firebase-login-demo-android) as a guide, I even got Google+ OAuth working without much trouble. From there, it didn't take long to load data into a Firebase instance partitioned by Google account. All that was left was to query the data out of Firebase and I'd be able to see the sync magic in action.

As you probably guessed from the title of this post, fetching a collection of objects from Firebase was not quite as straightforward. The documentation on [retrieving data](https://www.firebase.com/docs/android/guide/retrieving-data.html) covers the methods for executing asynchronous queries, but uses HashMap results instead of the objects I initially stored. Um...no thanks.

I eventually noticed an overload of the `getValue()` method  of the `DataSnapshot` class that takes a `GenericTypeIndicator<T>` parameter. In the [JavaDocs](https://www.firebase.com/docs/java-api/javadoc/com/firebase/client/GenericTypeIndicator.html) for that class, is the secret sauce:

```java
GenericTypeIndicator<List<Beer>> typeIndicator = new GenericTypeIndicator<List<Beer>>() {};
List<Beer> beers = snapshot.getValue(typeIndicator);
```
<br/>
Combining that with [these](https://gist.github.com/gsoltis/86210e3259dcc6998801) awesome RxJava Firebase bindings, I can now load a list with something as simple as:

```java
private Observable<List<Beer>> loadBeers() {
    Firebase firebase = new Firebase("https://some-instance.firebaseio.com/beers");
    GenericTypeIndicator<List<Beer>> typeIndicator = new GenericTypeIndicator<List<Beer>>() {};
    return RxFirebase.observe(firebase.orderByKey())
        .map(snapshot -> snapshot.getValue(typeIndicator));
}
```

Aside from this little hiccup, Firebase is hard to beat for easy persistence that automatically syncs across devices. I look forward to seeing what [Google](https://firebase.googleblog.com/2014/10/firebase-is-joining-google.html) does with it.
