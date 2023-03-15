---  
layout: post  
title: "Using DebugView with Firebase Analytics."  
categories: android
tags: android 
comments: true
showAds: true
description: Let's use DebugView to test the analogics of Firebase for debugging. This allows users to see when they used the app, which screen they saw, and which button they clicked on. It even tells you when you deleted the app.
---
## Using DebugView with Firebase Analytics
* Firebase Setting
* Dependency for Firebase DebugView
* Example code for Firebase DebugView
* Firebase DebugView Test

# Firebase Setting
#### 1. Access Android Studio

#### 2. Click [Tool] -> [Firebase] at the top

<img src="https://user-images.githubusercontent.com/127634758/225017424-d4855ca7-0f64-447a-8929-2e3dce52c759.png" width="350">

#### 3. Select [Analytics] from the menu -> Get started with Google Analytics [Kotlin]

<img src="https://user-images.githubusercontent.com/127634758/225017966-b30246d0-4ad3-4032-ab2e-a60b68af4ac4.png" width="350">

#### 4. After that, you can follow the instructions for firebase.

------------

# Dependency for Firebase DebugView

``` kotlin
implementation 'com.google.firebase:firebase-analytics:21.2.0'
```

---

# Example code for Firebase DebugView

* Code Story
It is an app with a layout structure that has Main Activity and a fragment called First Fragment in it.

> MainActivity

``` kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private lateinit var analytics: FirebaseAnalytics

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        Timber.plant(MyTimber())

        // init Analytics
        analytics = Firebase.analytics

        // send screen_view event
        analytics.logEvent(FirebaseAnalytics.Event.SCREEN_VIEW) {
            param(FirebaseAnalytics.Param.SCREEN_NAME, this@MainActivity.javaClass.simpleName)
            param("test", "test")
        }
}
```

> FirstFragment

``` kotlin
class FirstFragment : Fragment() {
    private var binding: FragmentFirstBinding by autoCleared()
    private lateinit var analytics: FirebaseAnalytics

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        binding = FragmentFirstBinding.inflate(inflater, container, false)

        // init Analytics
        analytics = Firebase.analytics

        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // send select_content event
        binding.analyticsButton.setOnClickListener {
            analytics.logEvent(FirebaseAnalytics.Event.SELECT_CONTENT) {
                param(FirebaseAnalytics.Param.SCREEN_NAME, this@FirstFragment.javaClass.simpleName)
                param("test", "test")
                param(FirebaseAnalytics.Param.ITEM_NAME, "test item name");
            }
        }
    }
}
```

> completed Layout

<img src="https://user-images.githubusercontent.com/127634758/225019625-c354f8a0-5f7b-4d00-ab63-a30a35bdb87e.png" width="350" alt="">

----

# Firebase DebugView Test

The Analytics test uses something called DebugView.

There are tests using adb, but DebugView is intuitive and reliable.

Using the code written above, you can obtain the following results

1. Event to output MainActivity name when MainActivity is opened (screen_view)
2. Send a click event when the SEND ANALYTICS button is pressed (select_content)

<img src="https://user-images.githubusercontent.com/127634758/225021082-b452e21b-df26-49ff-8895-e194445b00e5.png" width="350" alt="Firebase DebugView Test">

---

# Test DebugView with Analytics in Firebase Summary
* To set up Firebase
  * Set up and define in Android Studio
  * To define the dependencies required for Firebase
* To write code for use in Debug View
  * Define events in Main Activity and Fragment
* Test DebugView (debug)
  * When a specific screen is opened = screen_view
  * When a specific click is made = select_content