---  
layout: post  
title: "Android Kotlin Braodcast Receiver test - for beginner"
categories: android
tags: android 
comments: true
showAds: true
description: There are two ways to test a Broadcast Receiver 1. Sending a signal directly to the Broadcast Receiver using ADB. 2. Using Instrumented Unit Test to test on an emulator.
---

### **Testing Android Broadcast Receiver**

There are two ways to test a Broadcast Receiver:
1. Sending a signal directly to the Broadcast Receiver using ADB.
2. Using Instrumented Unit Test to test on an emulator.

Let's take a look at how to use them and their pros and cons.

---

## **Testing Android Broadcast Receiver using ADB**

First, to test it using ADB, you need to create a test environment as follows:

1. Connect your Android device to your computer.
2. Verify that the device is connected using ADB.
    - Enter the command "adb devices" and check if the connected device is displayed.
3. Open the ADB shell to send a broadcast event.
    - Enter the command "adb shell".
4. Send a broadcast event.
    - Use the "am broadcast" command to generate a broadcast intent.
    - For example, enter "am broadcast -a android.intent.action.BOOT_COMPLETED" to simulate a boot completed event.
5. Verify that the event has been received.
    - You can verify that the broadcast has been sent by running an app that checks for broadcast reception.
    - For example, to check for a boot completed event, restart your device, log in, and run the app to verify the event.


### **Modifying AndroidManifest**

You need to register the <receiver> in AndroidManifest as follows:

Set exported to **true** so that you can receive broadcasts from outside.

``` kotlin
<receiver
    android:name=".mybroadcast.MyReceiver"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>

```

### **Notes**

System commands such as **android.intent.action.BOOT_COMPLETED** cannot be called from outside on **Android 26 (Oreo)** or later.

Therefore, you can only test it with a custom action or by specifying a package and class name, as shown below.

```
adb shell am broadcast -n your.package.name/your.package.name.braodcastClassName -e ACTION_BUTTON "action1"

```

**Note:**

- **n** is the package name/broadcast class name.
- **e** is the data to be sent together. (For example, here, "action1" is included in the name ACTION_BUTTON.)

---

## **Testing with AndroidJUnit4**

Create a file in the androidTest package and enter the following to easily test it.

``` kotlin
import android.content.Context
import android.content.Intent
import androidx.test.core.app.ApplicationProvider
import androidx.test.ext.junit.runners.AndroidJUnit4
import androidx.test.platform.app.InstrumentationRegistry
import org.junit.Assert.assertEquals
import org.junit.Test
import org.junit.runner.RunWith

@RunWith(AndroidJUnit4::class)
class MyBroadcastReceiverInstrumentedTest {

    @Test
    fun testOnReceive() {
        val context = InstrumentationRegistry.getInstrumentation().context
        val intent = Intent("android.intent.action.BOOT_COMPLETED")
        val receiver = MyBroadcastReceiver()
        receiver.onReceive(context, intent)
        // Add verification code
        assertEquals("android.intent.action.BOOT_COMPLETED", intent.action)
    }
}

```

You can test **"android.intent.action.BOOT_COMPLETED"** with **AndroidJUnit4**.