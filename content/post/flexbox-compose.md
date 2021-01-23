---
author: "Andy Dyer" 
title: "Flexbox Layout Behavior in Jetpack Compose"
date: 2021-01-22T10:31:36+01:00
draft: true
categories: ["Android", "Jetpack Compose"]
---

### Introduction

The [CSS Flexible Box Layout specification](https://drafts.csswg.org/css-flexbox-1/) (AKA flexbox) is a useful abstraction for describing layouts in a platform neutral way. For this reason, it is widely used on the web and even [on mobile](https://github.com/google/flexbox-layout). In the [Zalando Fashion Store app](https://play.google.com/store/apps/details?id=de.zalando.mobile), we are using flexbox to define the layout of our backend-driven screens, which I [spoke about previously](http://andydyer.org/blog/2019/12/22/appcraft-faster-than-a-speeding-release-train/).

Thus far, we have been using Facebook's [Litho](https://github.com/facebook/litho) declarative UI library (which uses the flexbox based [Yoga layout engine](https://github.com/facebook/yoga)) for rendering backend driven screens on Android because it supports many things that are essential when building fully dynamic UI at runtime such as async layout, efficient diffing of changes, and view flattening.

As Google prepares [Jetpack Compose](https://developer.android.com/jetpack/compose) (now in alpha) for production release, we have started evaluating it as a successor to Litho. Compose offers numerous [layout composables](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary#top-level-functions), many with bits of flexbox like behavior. However, there is no `Flexbox` composable that does it all and no blog post explaining how flexbox concepts map to Compose, so I wrote this one. I also built [this sample app](https://github.com/abdyer/flexbox-compose), parts of which I will reference in code examples below.

Before we continue, yes, I know technically it's called *Compose UI* and not simply *Compose*, but [as Jake said](https://jakewharton.com/a-jetpack-compose-by-any-other-name/), most of us are already thinking of it this way. Insert a "UI" where necessary while reading if you'd like.

### Flex

Let's start with the flex attributes, which describe the direction, size, and horizontal/vertical alignment of a layout's children.

#### Flex Direction

[Flex direction](https://drafts.csswg.org/css-flexbox-1/#flex-direction-property) specifies whether items are arranged vertically or horizontally. Compose has [`Row`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary#row) and [`Column`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary#column) composables that work for simple horizontal and vertical layouts.

```kotlin   
@Composable
fun RowExample() {
    Row(
        modifier = Modifier.fillMaxWidth()
            .padding(bottom = 16.dp)
            .background(color = MaterialTheme.colors.primaryVariant),
    ) {
        Child()
        Child()
        Child()
    }
}
```

If [flex wrap](https://drafts.csswg.org/css-flexbox-1/#flex-wrap-property) behavior is needed to control how items wrap across multiple rows, the [`FlowRow`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary#flowrow) and [`FlowColumn`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary#flowcolumn) composables will do this. However, [these were deprecated](https://android-review.googlesource.com/c/platform/frameworks/support/+/1521704) before I even finished writing this article (that's alpha for you), so the best we can do is use the old implementation as a reference for our own.

```kotlin
@Deprecated
@Composable
fun FlowRowExample() {
    FlowRow(
        mainAxisSpacing = 8.dp,
        crossAxisSpacing = 8.dp
    ) {
        repeat(20) {
            Child(width = 48.dp, height = 24.dp)
        }
    }
}
```

#### Flex Grow & Shrink

[Flex grow](https://developer.mozilla.org/en-US/docs/Web/CSS/flex-grow) controls how children will expand to fill available space in their parent layout. [Flex shrink](https://developer.mozilla.org/en-US/docs/Web/CSS/flex-shrink) is its opposite, controlling how children will shrink relative to siblings if their parent layout does not have room for all of them.

Use the `weight()` modifier for flex grow behavior. Compose does not really have a flex shrink analog, but with its variety of layout composables, this can be overcome with a different approach in most cases. Depending on your specific needs, one approach could be to use `Modifier.preferredWidth(IntrinsicSize.Min)` to specify that a composable should not take up any more space than its children require. You can read more about it [here](https://jetc.dev/slack/2021-01-17-matching-parent-size.html) in this question reposted from the [kotlinlang Slack](https://slack.kotlinlang.org/) in Mr. Mark Murphy's excellent [jetc.dev](https://jetc.dev) newsletter.

```kotlin
@Composable
fun FlexGrowExample() {
    Row(
        modifier = Modifier.fillMaxWidth()
            .padding(bottom = 16.dp)
            .background(color = MaterialTheme.colors.primaryVariant),
    ) {
        FlexChild(modifier = Modifier.weight(1F))
        FlexChild(modifier = Modifier.weight(2F))
        FlexChild(modifier = Modifier.weight(1F))
    }
}
```

When the utmost flexibility is needed, there's always [implementing your own](https://developer.android.com/codelabs/jetpack-compose-layouts#5) `Layout` composable or the raw power of the [ConstraintLayout composable](https://developer.android.com/jetpack/compose/layout#contraintlayout), which can be used directly from Compose. If you don't mind reading Java instead of Kotlin, the implementation in Google's [`flexbox-layout` library](https://github.com/google/flexbox-layout/blob/master/flexbox/src/main/java/com/google/android/flexbox/FlexboxHelper.java) is a good starting point for understanding the algorithm.

### Alignment

Alignment controls how items are arranged on their vertical and horizontal axes. This can be done on a parent layout with the `*-content` properties or on the children themselves using the `*-items` properties.

#### Main Axis

Main axis alignment refers to how children are aligned on the main axis of their parent; horizontal for rows and vertical for columns. In the flexbox spec, this is known as [`justify-content`](https://developer.mozilla.org/en-US/docs/Web/CSS/justify-content). In Compose, main axis alignment is controlled by the the `horizontalArrangement` parameter passed to `Row` and the `verticalArrangement` parameter passed to `Column`. Both include options such as start/end, center, and space around/between/evenly for possible values.

```kotlin
@Composable
fun ArrangementExample() {
    Row(
        modifier = Modifier.fillMaxWidth()
            .padding(bottom = 16.dp)
            .background(color = MaterialTheme.colors.primaryVariant),
        horizontalArrangement = Arrangement.SpaceBetween,
    ) {
        Child()
        Child()
        Child()
    }
}
```

#### Cross Axis
Cross axis alignment refers to how children are aligned on the non-main axis of their parent; vertical for rows and horizontal for columns. In the flexbox spec, [`align-items`](https://developer.mozilla.org/en-US/docs/Web/CSS/align-items) controls layout children and [`align-self`](https://developer.mozilla.org/en-US/docs/Web/CSS/align-self) allows children to do so themselves. In Compose, cross axis alignment is controlled by the `verticalAlignment` parameter passed to `Row`, the `horizontalAlignment` parameter passed to `Column`, and the `align` modifier on their child composables. Both include options start, end, and center for possible values.

```kotlin
@Composable
fun AlignmentExample() {
    Row(
        modifier = Modifier.fillMaxWidth()
            .height(150.dp)
            .padding(bottom = 16.dp)
            .background(color = MaterialTheme.colors.primaryVariant),
        verticalAlignment = Alignment.CenterVertically,
    ) {
        Child()
        Child()
        Child()
    }
}
```

You may have noticed that the space around/between/evenly options from `justify-content` are not listed for the cross axis. This is because there is no cross axis space around/between alignment in Compose. However, the resulting layout could be achieved via other modifier combinations.

Flexbox also specifies a `stretch` option for cross axis alignment. In Compose, the `stretch` equivalent would be individual children using the `fillMaxSize()`/`fillMaxWidth()`/`fillMaxHeight()` modifiers. 

### Layout

Layout attributes describe the remaining non-flex properties that affect a view's size and position.

#### Aspect Ratio

Compose's `aspectRatio()` modifier works exactly as you'd expect. It takes a float representing the desired ratio and uses that value to determine the size in the unspecified layout direction (width or height).

For example, specifying `fillMaxWidth()` and `aspectRatio(16F / 9F)` results in a rectangle that fills the width of the screen with a height corresponding to 9/16 of that width.

```kotlin
@Composable
fun AspectRatioExample() {
    Box(
        modifier = Modifier.padding(bottom = 16.dp)
            .background(color = MaterialTheme.colors.secondary)
            .fillMaxWidth()
            .aspectRatio(16F / 9F)
            .border(width = 2.dp, color = MaterialTheme.colors.secondaryVariant)
    )
}
```

#### Padding & Margins

Compose has a `padding()` modifier, but none for margins. Margins can be considered extra padding, so a single value can be used.

#### Absolute Position

When absolute positioning is needed to place one composable on top of another, the [`Box`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary#box) composable can be used. `Box` children can use the `align()` modifier to specify where they are aligned within the box including top start/center/end, bottom start/center/end, and center start/end.

```kotlin
@Composable
fun AbsolutePositionExample() {
    Box {
        Box(
            modifier = Modifier.fillMaxWidth()
                .height(240.dp)
                .background(color = MaterialTheme.colors.primaryVariant)
        )
        Child(modifier = Modifier.align(Alignment.TopStart))
        Child(modifier = Modifier.align(Alignment.TopEnd))
        Child(modifier = Modifier.align(Alignment.BottomStart))
        Child(modifier = Modifier.align(Alignment.BottomEnd))
        Child(modifier = Modifier.align(Alignment.Center))
    }
}
```

### Conclusion

In this article, we have seen how much of the layout behavior defined in the flexbox spec has a direct analog in Compose and a few places where we have to do a bit more work to approximate certain concepts. Please see [the sample app repo](https://github.com/abdyer/flexbox-compose) for the code as well as my first attempt at working with the [Compose Navigation library](https://developer.android.com/jetpack/compose/navigation). Thanks for reading!
