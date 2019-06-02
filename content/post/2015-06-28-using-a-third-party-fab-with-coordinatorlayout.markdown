---
type: post
title: "Using a Third Party FAB with CoordinatorLayout"
date: "2015-06-28"
comments: true
categories: [Android, Apps, Material Design]
---

Before the Design Support Library was announced at Google I/O last month, I had numerous third party libraries in my projects for various elements of Material Design. I've enjoyed replacing the nav drawer, tabs, and parallax scrolling libraries with their support library counterparts. However, on one of my projects, we needed an expanding floating action button (FAB) similar to the one in [Inbox](https://play.google.com/store/apps/details?id=com.google.android.apps.inbox), which the support library does not currently provide.

I chose [this](https://github.com/futuresimple/android-floating-action-button) FAB library since it includes a cool little animation from the plus sign to the "X" when expanding the menu. I foolishly thought I could just drop this into a CoordinatorLayout in place of the support library's FAB. If you've checked out Chris Banes' excellent [Cheesesquare](https://github.com/chrisbanes/cheesesquare) sample, you might have noticed the `app:layout_anchor` and `app:layout_anchorGravity` attributes defined on the `FloatingActionButton` element:

```xml
<android.support.design.widget.FloatingActionButton
        android:layout_height="wrap_content"
        android:layout_width="wrap_content"
        app:layout_anchor="@id/appbar"
        app:layout_anchorGravity="bottom|right|end"
        android:src="@drawable/ic_discuss"
        android:layout_margin="@dimen/fab_margin"
        android:clickable="true"/>
```

These two attributes are what tells the enclosing `CoordinatorLayout` how to position the button and how it should behave when scrolling. To investigate how they are used, I used the good ol' `⌘+B` shortcut to go to the `FloatingActionButton`'s class definition. While the support library is not open source, I was still able to view the decompiled source.

The first thing I noticed is the `@DefaultBehavior(FloatingActionButton.Behavior.class)` annotation at the top of the class. The `FloatingActionButton.Behavior` child class provides the `Behavior` implementation used by `CoordinatorLayout`. I copied and pasted this inner class into my own subclass of the third party library's `FloatingActionsMenu` class, updated the generic type, and started resolving the compile errors. It didn't take me long to realize that several package private classes and methods were being called in this code. There are two ways to solve this. One is to copy the decompiled source of these classes into my project, but there is still the `AppBarLayout`'s package private `getMinimumHeightForVisibleOverlappingContent()` method to deal with. A simpler, yet slightly hackish way to handle this is to just define our class in a `android.support.design.widget` package.

The last issue specific to this library is that its measured height includes its children, so the FAB is not correctly positioned on the edge of the image and content views. After a bit of experimentation, I solved this by overriding `onMeasure()`:

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    // measured height includes child views, so translate Y to align FAB on view edges
    if (getTranslationY() == 0) {
        View fab = getChildAt(getChildCount() - 1); // last child is the FAB
        int shadowRadius = getResources().getDimensionPixelSize(R.dimen.fab_shadow_radius);
        int offset = (getMeasuredHeight() - fab.getMeasuredHeight() + shadowRadius) / 2;
        ViewCompat.setTranslationY(this, offset);
    }
}
```

The [source](https://github.com/abdyer/cheesesquare/blob/master/app/src/main/java/android/support/design/widget/SupportFloatingActionsMenu.java) for `SupportFloatingActionsMenu` is on GitHub in [my Cheesesquare fork](https://github.com/abdyer/cheesesquare). Hopefully we'll see first class support for integrating alternate FAB implementations more cleanly with the support library soon.

Here's a screen shot of the end result:

<br/>

<img src="/images/cheesesquare_fab_menu.png" alt="Cheesesquare FAB menu" style="width: 405px; margin-left: auto; margin-right: auto; display: block" />

<br/>