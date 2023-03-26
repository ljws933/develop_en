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
1. Sending signals directly to the Broadcast Receiver using ADB
2. Using Instrumented Unit Test to test on an emulator

Both methods have their pros and cons, let's take a look at how to use them.

---

## **Testing Android Broadcast Receiver with ADB**

To test with ADB, you need to create a test environment as follows:

1. Connect the Android device to your computer.
2. Use ADB to ensure that the device is connected.
    - Enter the `adb devices` command and check if the connected device is displayed.
3. Open the ADB shell to send broadcast events.
    - Enter the `adb shell` command.
4. Send a broadcast event.
    - Use the `am broadcast` command to generate a broadcast intent.
    - For example, you can simulate a boot complete event by entering the `am broadcast -a android.intent.action.BOOT_COMPLETED` command.
5. Verify event reception.
    - Run an app that checks broadcast reception to confirm that the event has been sent.
    - For example, to confirm the boot complete event, restart the device, log in, and run the app.


### **Modifying androidManifest**

Register `<receiver>` in the androidManifest and set `exported` to `true`.

``` kotlin
<receiver
    android:name=".mybroadcast.MyReceiver"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>

```

You also need to define a receiver filter for testing.

You can define it as follows:

``` kotlin
<receiver
    android:name=".broadcast.MyReceiver"
    android:exported="true">
    <intent-filter>

        <action android:name="com.my.packageName" />

    </intent-filter>

```

### **Note**

System commands like `android.intent.action.BOOT_COMPLETED` cannot be called from external sources on Android 26 (Oreo) or later.

Therefore, you can only test by custom actions or by specifying the package and class name, as follows:

```
adb shell am broadcast -n your.package.name/your.package.name.braodcastClassName -e ACTION_BUTTON "action1"

```

Note that:

- `n` is the package name/broadcast class name.
- `e` is the accompanying data. (In this case, the action name is "ACTION_BUTTON" and the value is "action1".)

---

## **Testing with AndroidJUnit4**

Before testing Broadcast Receiver, please make sure that you have defined the filter in androidManifest as mentioned above.

Create a file in the androidTest package, and you can easily test it as follows:

``` kotlin
@RunWith(AndroidJUnit4::class)
class AlarmBroadCastTest {
    private val context = InstrumentationRegistry.getInstrumentation().context
    private lateinit var receiver: AlarmReceiver

    @Before
    fun setup() {
        receiver = AlarmReceiver()
        val intentFilter = IntentFilter().apply {
            addAction(context.packageName)
        }
        // Register BroadcastReceiver
        InstrumentationRegistry.getInstrumentation().targetContext.registerReceiver(
            receiver,
            intentFilter
        )
    }

    @Test
    fun test_MY_PACKAGE_REPLACED_OnReceive() {
        val intent = Intent(context.packageName)
        // Send signal to BroadcastReceiver
        context.sendOrderedBroadcast(intent, null)
        // Wait for onReceive() in Receive class
        Thread.sleep(2000)
        // Check if the variable in the receiver is set to true
        assertTrue(receiver.broadcastCalled)
    }

    @After
    fun tearDown() {
        // Unregister BroadcastReceiver
        InstrumentationRegistry.getInstrumentation().targetContext.unregisterReceiver(receiver)
    }
}

```

Note that the receiver is defined as follows:

``` kotlin
override fun onReceive(context: Context?, intent: Intent?) {
    println("Broadcast is called")

    // broadcastCalled is initially false.
    // It will be set to true only when onReceiver is called.
    broadcastCalled = true
}

```

You can check the `broadcastCalled` variable to see if the Broadcast Receiver has been called.