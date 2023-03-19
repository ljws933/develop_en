---  
layout: post  
title: "How to get Notification authorization on Android 13 and what changes."  
categories: android
tags: android 
comments: true
showAds: true
description: Let's use DebugView to test the analogics of Firebase for debugging. This allows users to see when they used the app, which screen they saw, and which button they clicked on. It even tells you when you deleted the app.
---

### **Changes in Notification in Android 13**
- POST_NOTIFICATIONS (Notification Permission) can be added from Target SDK API 33 or higher
- If an app with Target SDK API 32 or lower is installed on an Android 13 device, the Notification permission request popup will appear **automatically** when registering a Notification Channel
- If an app with Target SDK API 33 or higher is installed on an Android 13 device, the Notification permission request can be displayed at the **developer's desired timing**
- If an app with Target SDK API 33 or higher is installed on an Android 12 or lower device, it can be used without requesting the Notification permission as before
- If an API 32 app is updated to 33 and the user has already agreed to the notification permission, it will be allowed by default after the update, but there are exceptions
    - There may be cases where it is not automatically allowed depending on the device and you need to request permission again
    - Therefore, it is recommended to include a check to see if the Notification permission is granted.

## **Cautions for Notification Permission Request Popup**

- For apps with **API 32 or lower**, if the user presses the Don't allow button when the system automatically displays the permission popup, the popup will not appear even if the app is restarted.
- For apps with **API 33 or higher**, if the app displays the permission popup, it can continue to display the permission popup until the user presses the Don't allow button **twice**.
- If the user can no longer display the popup by pressing the Don't allow button, they must go to the app settings and grant permission.

---

## **Registering the App as Needing Notification Permission**

> AndroidManifest.xml


``` kotlin
<manifest ...>
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
    <application ...>
        ...
    </application>
</manifest>

```

1. Add the above permission to AndroidManifest.xml.
2. Perform "Sync project with gradle files".

---

## **Change compileSdk to 33**

- The compileSdk in the app-level build.gradle must be changed to 33
    - The targetSdk also needs to be changed to 33

``` kotlin
android {
    compileSdk 33

    defaultConfig {
        applicationId "com.example.selftest"
        minSdk 23
        targetSdk 33
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

```

---

## **Defining Notification**

- Define the Notification Channel and Builder for the Notification
    - From SDK version 26 or higher, a channel must be defined to use Notification
    - Send Notification using the Builder

``` kotlin
fun makeAlarmNotification(context: Context, messageBody: String) {
    Timber.d("make notification")
    // Open MainActivity when notification is clickedval intent = Intent(context, MainActivity::class.java)
    intent.flags = Intent.FLAG_ACTIVITY_CLEAR_TASK or Intent.FLAG_ACTIVITY_NEW_TASK
    val pendingIntent = PendingIntent.getActivity(
        context, 0, intent,
        PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
    )

    // Define the Notification channel idval channelId = "channel1"
    val channelName = "channel name"

        // Define the Notificationval notificationBuilder = NotificationCompat.Builder(context, channelId)
        .setSmallIcon(R.drawable.ic_bottom_menu_search)
        .setContentTitle("notification title")
        .setContentText(messageBody)
        .setAutoCancel(false)// Prevent it from being deleted completely
        .setSound(null)
        .setContentIntent(pendingIntent)
        .setPriority(NotificationCompat.PRIORITY_DEFAULT)
        .setOngoing(true)// Keep the alarm on

        // Create the Notification using the defined content and channelval notificationManager =
        context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

        // A channel must be specified when creating a notification for Android SDK 26 or higherif (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        val channel = NotificationChannel(
            channelId,
            channelName,
            NotificationManager.IMPORTANCE_DEFAULT
        )
        notificationManager.createNotificationChannel(channel)
    }

// Show the Notification
    notificationManager.notify(100, notificationBuilder.build())
}

```

---

## **Getting Notification Permission**

To display a Notification in Android 13 or higher, you must request and obtain permission.

Notification permissions can be obtained in the following ways.

> FirstFragment.kt
> 

Display the Notification permission request popup when the button is pressed.

``` kotlin
class FirstFragment : Fragment() {
    companion object {
        const val DENIED = "denied"
        const val EXPLAINED = "explained"
    }

    private var binding: FragmentFirstBinding by autoCleared()
        // Create an Activity Callback object for permission requestsprivate 
        val registerForActivityResult = registerForActivityResult(ActivityResultContracts.RequestMultiplePermissions()) { permissions ->
            val deniedPermissionList = permissions.filter { !it.value }.map { it.key }
            when {
                deniedPermissionList.isNotEmpty() -> {
                    val map = deniedPermissionList.groupBy { permission ->
                        if (shouldShowRequestPermissionRationale(permission)) DENIED else EXPLAINED
                    }
                    map[DENIED]?.let {
                        // When permission is simply denied
                    }
                    map[EXPLAINED]?.let {
                        // When the permission request is completely blocked (usually open the app details screen)
                    }
                }
                else -> {
                    // When all permissions are allowed
                }
            }
    }

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        binding = FragmentFirstBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding.button.setOnClickListener {
            val action = FirstFragmentDirections.actionFirstFragmentToThirdFragment()
            findNavController().navigate(action)
        }

        // Request Notification permission when the button is clicked
        binding.freeButton.setOnSingleClickListener {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
                registerForActivityResult.launch(
                    arrayOf(Manifest.permission.POST_NOTIFICATIONS)
                )
            }
            makeAlarmNotification(requireContext(), "notification")
        }
    }
}

```

This method can be used not only in Fragment but also in Activity.

---

## **Summary**

- For **existing apps**, the window for requesting permission automatically appears when registering a Notification Channel.
    - However, if permission is not granted at this stage, permission cannot be requested twice.
- If you have set SDK 33 or higher as the target, you must include a feature that requests permission.