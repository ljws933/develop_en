---  
layout: post  
title: "Open Activity on top of Lock Screen - Android 13"  
categories: android
tags: android 
comments: true
showAds: true
description: It is about how to show the screen even when the screen is locked on Android, and the method used before and after Android 12 is different, and for this, Manifest and Activity settings are required.
---

### Opening an Activity on the Lock Screen in Android 13
With the release of Android 12, the way to open an Activity on the lock screen has changed slightly. Additional permissions are required and the way it is implemented is different. Here is a quick and simple guide on how to do it.

---

## How to open an Activity on the lock screen in Android 13

### To open an Activity on the lock screen in Android 13, you need to first modify the androidManifest.

Modify the androidManifest.xml as follows:

> androidManifest.xml
> 

``` kotlin
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
<uses-permission android:name="android.permission.DISABLE_KEYGUARD" />

<!-- Configure the target Activity as follows -->
<activity
    android:name=".ui.activity.OnAlarmActivity"
    android:permission="android.permission.SYSTEM_ALERT_WINDOW"
    android:excludeFromRecents="true"
    android:exported="false"
    android:launchMode="singleTask"
    android:showOnLockScreen="true"/>

```

Then define the Activity that will be displayed on the lock screen as follows:

> LockScreenOverlayActivity
> 

Define it as follows within **onCreate()**:

``` kotlin
// Open an Activity on the lock screen in Android 12 and up & turn on the screen
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O_MR1) {
    setShowWhenLocked(true)
    setTurnScreenOn(true)
    (getSystemService(Context.KEYGUARD_SERVICE) as KeyguardManager).apply {
        requestDismissKeyguard(this@OnAlarmActivity, null)
    }
} else {
    this.window.addFlags(
        WindowManager.LayoutParams.FLAG_DISMISS_KEYGUARD or
                WindowManager.LayoutParams.FLAG_SHOW_WHEN_LOCKED or
                WindowManager.LayoutParams.FLAG_TURN_SCREEN_ON
    )
}

```

Once you have completed these steps, you can call the Activity and it will be displayed on the lock screen.

---

### Summary

1. Add SYSTEM_ALERT_WINDOW to the androidManifest.xml file and also to the Activity.
2. Set setShowWhenLocked to true in the Activity.
3. The method used to open an Activity on the lock screen is different for Android versions before and after Android 12.


