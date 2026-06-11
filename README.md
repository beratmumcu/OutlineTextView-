
# OutlineTextView

A custom Android `TextView` component in Kotlin that renders readable text with a clean outline (stroke) on top of any background. It is optimized to prevent the **infinite layout/draw recursion loop** (`requestLayout` and `invalidate` spam) which commonly occurs when changing text colors inside `onDraw()`.

## The Problem with Standard Outlined TextViews

Typically, drawing an outline in Android requires modifying the text paint style to `STROKE`, calling `super.onDraw()`, then restoring it to `FILL` and calling `super.onDraw()` again.
To change the color between stroke and fill stages, developers call `setTextColor()`.

However, in the Android Framework, `setTextColor()` automatically schedules layout passes and redraw requests:
```java
public void setTextColor(int color) {
    mTextColor = ColorStateList.valueOf(color);
    invalidate();
    requestLayout();
}
```
Calling `setTextColor()` inside `onDraw()` triggers `invalidate()`, which schedules another `onDraw()`, leading to **100% CPU usage, battery drain, device overheating, and frozen rendering loops**.

## The Solution

`OutlineTextView` resolves this by introducing a rendering lock state (`isDrawing`). While `isDrawing` is active inside the custom `onDraw()`, all incoming calls to `invalidate()` and `requestLayout()` are intercepted and silenced. Redundant invalidations are skipped, but the internal state updates cleanly.

## Key Features

- **No Infinite Invalidation Loops:** Runs smoothly at 60/120 FPS with minimal CPU/GPU overhead.
- **Highly Readable:** Perfect for custom subtitles, game HUDs, desktop pet speech bubbles, or floating widgets.
- **Customizable:** Easily change outline width and color programmatically.

## How to Use

### 1. Define in XML Layout

Add the custom component to your layout file:

```xml
<com.yourpackage.OutlineTextView
    android:id="@+id/outlinedText"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Hello, World!"
    android:textSize="18sp"
    android:textStyle="bold"
    android:textColor="#FFFFFF" />
```

### 2. Configure in Kotlin

Set custom outline attributes in your activity or service:

```kotlin
val tvOutlined = findViewById<OutlineTextView>(R.id.outlinedText)

// Customize the outline properties
tvOutlined.outlineColor = Color.BLACK
tvOutlined.outlineWidth = 6f // Width in pixels
```
