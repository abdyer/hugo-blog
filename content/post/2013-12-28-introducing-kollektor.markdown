---
type: post
title: "Introducing Kollektor"
date: "2013-12-28"
comments: true
categories: [Android, Apps, Kollektor, Vinyl, Music]
---

##### Why a new app?

When I first decided to get into Android development, I created the [Music Library](https://play.google.com/store/apps/details?id=com.dandydev.medialibrary) app to catalog my vinyl collection. It served me well as I learned the ins and outs of Android programming including [content providers](http://developer.android.com/guide/topics/providers/content-providers.html), interacting with other applications via [intents](http://developer.android.com/reference/android/content/Intent.html), working with the file system to save/load images, making API requests, etc. In order to test the workings of the Android app market, I published the ad supported Music Library Free and the paid Music Library versions of the app. In addition to not displaying ads, Music Library also included the ability to backup your collection to the device's SD card and export data to a Google Drive spreadsheet.

Both apps have seen a modest number of downloads over the past couple years. I've heard from people around the world who have found the apps useful, from music critics to casual collectors like myself. In addition to making it easy to catalog your record collection, Music Library included customization options such as the ability to add album art and manually add/edit release information. However, one of the most recurring feature requests was to support syncing data across devices.

##### What it do?

I initially started down the road of building my own API to manage the app's data to facilitate the data sync. A web site would provide another way to access your collection, eventually serving as a platform for adding social features for sharing with friends. Along the way, I took a closer look at [Discogs](http://www.discogs.com/) and realized that in addition to providing a very large database of releases for searches, its API was also fairly mature. By re-imagining Music Library as a Discogs client, I would improve upon Music Library in the following ways:

- Enable viewing your collection on all your devices while simultaneously removing the need to support import/export/backup features.
- Remove the need to add/edit release information manually on the device. An artist/album title search or barcode scan make it easy to add releases to your collection. Album art, track tiles, and other details are all pulled from Discogs' user-sourced release database.
- Implement an in-app purchase to remove ads and support future development. Aside from being much more straightforward, it also avoids the need to have an upgrade process for transferring data from the free version to the paid version.

##### Open source shout-outs

Building a new app from scratch allowed me to see what a few years of experience and leveraging open source libraries could do to streamline development. I used the following libraries to build the app (many of which I've mentioned in [my recent Android libraries talks](http://andydyer.org/blog/2013/10/11/big-android-bbq-2013-android-open-source-libraries-you-need-in-your-life/)):

- [Volley](https://developers.google.com/events/io/sessions/325304728) - API requests and asynchronous image loading
- [GSON](https://code.google.com/p/google-gson/) - Parsing API JSON responses
- [GreenDAO ORM](http://greendao-orm.com/) - Local data storage
- [Scribe](https://github.com/fernandezpablo85/scribe-java) - OAuth login and request signing
- [Otto](http://square.github.io/otto/) - Event bus, activity/fragment communication
- [Fading Action Bar](https://github.com/ManuelPeinado/FadingActionBar) - Release detail scrolling/parallax effect
- [UserVoice SDK](https://github.com/uservoice/uservoice-android-sdk) - Support & feature requests

##### Kollektor

I always felt that "Music Library", while descriptive was a bit too generic of a name for the app. I chose to name the new app "Kollektor" simply because I'm learning German and liked the funky spelling. I briefly considered "Plattensammler", but decided it's too long and less obvious.

##### What's next

A quick list of potential features to be added to Kollektor:

- Include other Discogs fields such as notes and reviews
- Album check-in/now playing sharing
- Sharing your collection with friends
- Importing collections from Music Library (this may take some time as Discogs does not support a bulk data import process and there is no direct mapping of Amazon releases to Discogs releases)

##### Download

[Kollektor](https://play.google.com/store/apps/details?id=com.dandydev.kollektor) on Google Play