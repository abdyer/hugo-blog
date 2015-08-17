---
layout: post
title: "Music Library 3.0 - Material Design Update"
date: "2014-11-02"
comments: true
categories: [Android, Apps, Lollipop, Music Library, Material Design, UI]
facebook:
    image: http://andydyer.org/images/music_library_title_list_v3.png
twitter_card:
    type: summary_large_image
    image: http://andydyer.org/images/music_library_title_list_v3.png
---

<img src="/images/music_library_icon.png" alt="Music Library" style="width: 150px; float: left; margin-right: 20px" />
I started developing [Music Library](https://play.google.com/store/apps/details?id=com.dandydev.medialibrary) about four years ago when I wanted an app for organizing my record collection. Armed with my Nexus One, Eclipse, and a copy of Apress' [Pro Android](http://www.amazon.com/gp/product/1430246804/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1430246804&linkCode=as2&tag=slacod-20&linkId=FMLK73AOGDPPJEM3), I spent my nights and weekends learning the inner workings of my now-favorite mobile OS.

Over the years, Music Library has been a playground of sorts for exploring various open source libraries, patterns, and best practices. While there is still a fair amount of code I would write differently today (I'm looking at you, ContentProvider), the app has been and continues to be an enjoyable side project.

Whether fixing bugs or implementing new features, I adhere to the [Boy Scout Rule](http://programmer.97things.oreilly.com/wiki/index.php/The_Boy_Scout_Rule) as much as possible, balancing the urge to rewrite everything with what makes sense for the changes being made. Despite my attempts at pragmatism, Google's roughly semiannual overhaul of the Android UI has a way of quickly dating apps that do not keep up with the latest design guidelines. Remember gradients everywhere in Gingerbread? Or when Honeycomb and Holo ushered in the last big design change? [Material Design](http://www.google.com/design/spec/material-design/introduction.html) continues that tradition, only more ambitious in scope; this time unifying the design of mobile, web, and Chromebook apps in one fell swoop.

The [official announcement](http://googleblog.blogspot.com/2014/10/android-be-together-not-same.html) of Lollipop and the release of the latest support library last month motivated me to update Music Library yet again to keep up with the latest evolution of the platform. After surveying the contents of the new version of the support library, I got to work bringing Material Design to the app.

I started by applying the `Theme.AppCompat.Light.DarkActionBar` theme as described on the [Android Developers](http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html) blog, which changed the action bar, color scheme, and widgets from the now too-familiar Holo electric blue to the new Material look.

Once that was done, I found that the spinner navigation the app was using has been deprecated in Lollipop. Taking a cue from Google's own apps such as Gmail and the Play Store, I decided to move the shelf list into a left navigation drawer. Menu options such as Scan Barcode, Settings, and editing the list of shelves also moved into the nav drawer. This left only the more common actions of Add, Edit, Search, and Sort in the action bar/overflow menu, cleaning things up significantly.

From there, I turned to the album art images, making them more prominent throughout the app. I used the `GridLayoutManager` and `CardView` widget to display the list of albums in a grid rather than a list, putting the focus on the artwork that makes our favorite albums immediately recognizable. This had the additional benefit of obviating the coverflow view that while fun to play with, always felt a bit out of place on Android.

To further enhance the look of the app, I used the [Palette library](https://blog.stylingandroid.com/palette-part-2/) to extract a dark vibrant color from each album's artwork for the action bar and fonts. As the selected album changes, the app's colors morph to complement the artwork. I'm amazed how well it works. I've yet to see a case where it didn't result in just the right color for the image being displayed.

As you can see from the before and after screen shots below, the redesign breathes new life into the app. I'm quite pleased with the result. I look forward to applying more Material Design elements to this app and others over the next few months.
<br/>
<table style="margin-left: auto; margin-right: auto">
  <tr>
    <th>Version 2.x (Holo)</th>
    <th>Version 3.0 (Material Design)</th>
  </tr>
  <tr>
    <td><a href="/images/music_library_title_list_v2.png"><img src="/images/music_library_title_list_v2.png" alt="Title List v2.x" style="width: 253px; margin-left: 20px; margin-right: 20px; margin-bottom:20px" /></a></td>
    <td><a href="/images/music_library_title_list_v3.png"><img src="/images/music_library_title_list_v3.png" alt="Title List v3.0" style="width: 253px; margin-right: 20px; margin-bottom: 20px" /></a></td>
  </tr>
  <tr>
    <td><a href="/images/music_library_title_edit_v2.png"><img src="/images/music_library_title_edit_v2.png" alt="Title Edit v2.x" style="width: 253px; margin-left:20px; margin-right: 20px; margin-bottom: 20px" /></a></td>
    <td><a href="/images/music_library_title_edit_v3.png"><img src="/images/music_library_title_edit_v3.png" alt="Title Edit v3.0" style="width: 253px; margin-right: 20px; margin-bottom: 20px" /></a></td>
  </tr>
</table>
<br/>
<div style="text-align: center">
    <a href="https://play.google.com/store/apps/details?id=com.dandydev.medialibrary" title="Get Music Library">
        <img alt="Get it on Google Play" width="203" src="/images/google_play_badge@2x.png">
    </a>
</div>