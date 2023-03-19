---  
layout: post  
title: "Preventing Automatic Focus on Android EditText"  
categories: android
tags: android 
comments: true
showAds: true
description: There are times when focus automatically goes to editText on Android Kotlin. At this time, the focus is automatically prevented.
---

### Preventing Automatic Focus on Android EditText
- Prevent automatic focus on EditText when the layout is clicked.

---

The EditText is given focus automatically. When a window with EditText is opened, the keyboard is automatically displayed.

If you want the keyboard to be displayed automatically, just use it as it is. However, there are times when you need to prevent automatic focus.

**To prevent this phenomenon:**

You only need to add **2 lines** to the xml code.

> android:focusable="true" 

> android:focusableInTouchMode="true"


Where should you add it?

**Generally, add it to the parent layout.**

Alternatively, you can add it where you want.

**Example**

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="<http://schemas.android.com/apk/res/android>"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:focusable="true"
    android:focusableInTouchMode="true">

    <EditText
        android:id="@+id/et01"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>

```

**As you can see in the LinearLayout(parent layout)**

**Two more lines are added at the end.**

**android:focusable="true"**

**android:focusableInTouchMode="true"**

Because of these attributes, when the screen appears, focus is given to LinearLayout instead of EditText.

Therefore, you can prevent the keyboard from automatically appearing.