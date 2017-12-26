---
author: "Andy Dyer"
date: "2017-12-24T20:53:35"
categories: ["Android", "Apps"]
title: "Introducing Biografy"
type: "post"
---

### What is Biografy?

[Biografy](https://play.google.com/store/apps/details?id=com.dandydev.biografy) is a lifelogging app that automatically builds a timeline of each day. The [_Events_](/images/biografy/events.png) view logs events from various apps including music, podcasts, videos, calls and chats. Each event includes rich detail such as the time, activity (stationary, walking, biking, etc.), and location/place. The [_Motion_](/images/biografy/motion.png) view lists car/transit trips, bike rides, runs, and walks. With the [_Map_](/images/biografy/map.png) view, you can see where you were when each event occurred and gain insights like where you spend your time based on the clusters of pins. Finally, the [_Notes_](/images/biografy/notes.png) view is perfect for adding context to remember each day. Want to travel back in time? Simply tap the toolbar to choose the date.

<div style="display: flex; flex-wrap: wrap;">
    <div style="flex: 100%; max-width: 100%;">
        <a href="/images/biografy/events.png" target="_blank">
            <img src="/images/biografy/events.png" alt="Biografy - Events" style="width: 200px" />
        </a>
        <a href="/images/biografy/motion.png" target="_blank">
            <img src="/images/biografy/motion.png" alt="Biografy - Motion" style="width: 200px" />
        </a>
        <a href="/images/biografy/map.png" target="_blank">
            <img src="/images/biografy/map.png" alt="Biografy - Map" style="width: 200px" />
        </a>
        <a href="/images/biografy/notes.png" target="_blank">
            <img src="/images/biografy/notes.png" alt="Biografy - Notes" style="width: 200px" />
        </a>
    </div>
</div>
<br/>

### How does it work?

Biografy does most of its work by listening to system notifications. Notification access is a special permission, so you will be prompted to enable it on the first launch. When a notification for a supported app is received, Biografy adds the time, current location and Google Place (if available) along with the current activity (stationary, walking, biking, etc.). Motion events are imported from [Google Fit](https://www.google.com/fit/). You may need to setup the Google Fit app separately in order for fitness events to be imported into Biografy.

### What else does it do?

In addition to automatically listening to app notifications and fitness/motion events, Biografy also recognizes songs (great for remembering the records you listen to) and imports track listens from [LastFM](http://last.fm/) (just add your user name on the Settings screen). If you’re like me and put your phone in Airplane Mode while sleeping, Biografy will also track sleep and wake events. Finally, if there's a moment during the current day you want to capture manually, simply tap the "add" button in the lower right to add a quick note.

### What about security and privacy?

Companies like Google, Facebook, and Amazon already have this data and much more. Biografy allows you to reclaim (at least some) control over this data by organizing it and making it useful to you instead of advertisers. Data is stored locally on the device and securely backed up to your [Google Drive](https://drive.google.com/drive/my-drive) account using [Auto Backup](https://developer.android.com/guide/topics/data/autobackup.html) on devices running Android 6.0 (Marshmallow) or later. On devices with a fingerprint sensor, Biografy will optionally prompt for authentication when opening the app for an additional layer of security.

While developing and [dogfooding](https://en.wikipedia.org/wiki/Eating_your_own_dog_food) the app over the past year or so, I experimented with storing the data in [Firebase](https://firebase.google.com/) (i.e. “the cloud”). With this, it would be possible to support multiple devices and deliver a web dashboard displaying aggregated stats like hours of sleep per night, hours of physical activity per week, top tracks/podcasts, most frequently contacted friends, etc. While cool (I even wrote it in [Elm](http://elm-lang.org/)!), I ultimately decided to remove the cloud data storage before releasing the app because I do not want to be responsible for storing such personal data.

In the future, I plan to add a stats/dashboard view to the app and may investigate running a web app on the device in such a way that it could be accessed by a computer on the same local network. This would allow your personal data to remain on your phone while still being available for aggregation in a more full-featured desktop app. Stay tuned for future updates on this.

For now, the most important thing to know is your personal data remains on the device under your control. To be fully transparent, I do track basic app actions for supporting and improving the app, but no personally identifiable information is ever included. Crashes are also logged, but also without any personally identifiable data.

### Which apps are currently supported?

I developed Biografy to support the apps I use and some of the most popular apps you are likely to be using. Adding support for new apps isn’t too difficult, so please let me know if there’s an unsupported app you’d like me to add. In short, it should be possible to support any app that displays periodic notifications for its content. Most media and social/messaging apps are great for this.

The following apps are currently supported:

* [Phone](https://play.google.com/store/apps/details?id=com.google.android.dialer)
* [Google Hangouts](https://play.google.com/store/apps/details?id=com.google.android.talk)
* [Google Play Music](https://play.google.com/store/apps/details?id=com.google.android.music)
* [MixCloud](https://play.google.com/store/apps/details?id=com.mixcloud.player)
* [Plex](https://play.google.com/store/apps/details?id=com.plexapp.android)
* [Pocket Casts](https://play.google.com/store/apps/details?id=au.com.shiftyjelly.pocketcasts)
* [SoundCloud](https://play.google.com/store/apps/details?id=com.soundcloud.android)
* [Spotify](https://play.google.com/store/apps/details?id=com.spotify.music)
* [WhatsApp](https://play.google.com/store/apps/details?id=com.whatsapp)
* [YouTube](https://play.google.com/store/apps/details?id=com.google.android.youtube)

### Conclusion

I hope you enjoy using Biografy as much as I enjoyed building it. If so, I’d love it if you leave a review on [Google Play](https://play.google.com/store/apps/details?id=com.dandydev.biografy). As I mentioned, I’ve been using it myself for over a year, so hopefully you find it largely bug-free. If not, please contact me via the support link in the Settings screen. There’s also a form for requesting support for new apps if there’s one you’d like me to add.
