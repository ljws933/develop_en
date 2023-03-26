---  
layout: post  
title: "How can I get viewModel event with MVVM - 1"
categories: android
tags: android 
comments: true
showAds: true
description: There is many way to get viewModel event. liveData, EventWrapper, SingleLiveData etc. Let you know how to use, how to get these event with MVVM architecture.
---

### **Ways to receive viewModel events in MVVM**
1. Handle events using only LiveData
2. Wrap EventFlow in LiveData to handle it
3. Handle events using SingleLiveData
4. Handle events using StateFlow and SharedFlow
5. Handle events using SharedFlow and Sealed class
6. Handle events using SharedFlow & Sealed class & LifeCycle
7. Handle events using EventFlow & Sealed class Lifecyle

---

## **Project Setup**

I'll briefly explain how the project is set up.

The UI is set up as follows:

![https://blog.kakaocdn.net/dn/wSTpa/btr53FCi2ri/nWqeBeadLiaDSLnaKsH7m0/img.png](https://blog.kakaocdn.net/dn/wSTpa/btr53FCi2ri/nWqeBeadLiaDSLnaKsH7m0/img.png)

The following actions are executed:

1. Click each button.
2. Insert data into each LiveData or flow variable.
3. Perform observe on the variable of step 2 and open an empty activity when it is observed.
4. Return to the previous activity and check the value stored in variable 2.

---

## **Handle Data Using Only LiveData**

### **ViewModel Configuration**

``` kotlin
class LiveDataViewModel: ViewModel() {
    private val _someData = MutableLiveData<Int>()
    val someData: LiveData<Int> = _someData

    fun setData() {
        if (_someData.value == null) {
            _someData.value = 0
        } else {
            _someData.value = _someData.value!! + 1
        }
    }
}

```

### **Activity for Testing**

``` kotlin
class FirstActivity : AppCompatActivity() {
    private lateinit var binding: ActivityFirstBinding
    private val liveDataViewModel: LiveDataViewModel by viewModels()
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityFirstBinding.inflate(layoutInflater)
        setContentView(binding.root)

        lifecycleScope.launch {
            liveDataViewModel.someData.observe(this@FirstActivity) {
                Timber.d("data observed: $it")
                startActivity(Intent(this@FirstActivity, SecondActivity::class.java))
            }
        }

        binding.liveDataButton.setOnClickListener {
            liveDataViewModel.setData()
        }
    }

    override fun onResume() {
        super.onResume()
        Timber.d("liveDataViewModel Data: ${liveDataViewModel.someData.value}")
    }
}

```

### **Test Results**

![https://blog.kakaocdn.net/dn/bmg1ZG/btr5RZ2OD6S/CtojHX1K1WZVyKx0mBFLgk/img.png](https://blog.kakaocdn.net/dn/bmg1ZG/btr5RZ2OD6S/CtojHX1K1WZVyKx0mBFLgk/img.png)

Even though we returned to the activity, we can see that the **existing data remains**.

> To prevent data from remaining, we'll use EventWrapper.
> 

---

## **Apply EventWrapper to LiveData**

There is a way to prevent data from being observed again when returning to the screen or to initialize data by defining the EventWrapper class. This was once a method used by Google.

The EventWrapper class mentioned here can be defined in the following way.

``` kotlin
/**
 * Used as a wrapper for data that is exposed via a LiveData that represents an event.
 */
open class Event<out T>(private val content: T) {

    var hasBeenHandled = false
        private set // Allow external read but not write

    /**
     * Returns the content and prevents its use again.
     */
    fun getContentIfNotHandled(): T? {
        return if (hasBeenHandled) {
            null
        } else {
            hasBeenHandled = true
            content
        }
    }

    /**
     * Returns the content, even if it's already been handled.
     */
    fun peekContent(): T = content
}

```

The ViewModel that uses this EventWrapper can be defined as follows.

``` kotlin
class EventLiveDataViewModel: ViewModel() {
    private val _someData = MutableLiveData<Event<Int>>()
    val someData: LiveData<Event<Int>> = _someData

    fun setData() {
        if (_someData.value?.getContentIfNotHandled() == null) {
            _someData.value = Event(0)
        } else {
            _someData.value = Event(_someData.value?.getContentIfNotHandled() as Int + 1)
        }
    }
}

```

And the activity to be tested can be defined as follows. (This part will be omitted from now on)

``` kotlin
class FirstActivity : AppCompatActivity() {
    private lateinit var binding: ActivityFirstBinding
    private val liveDataViewModel: LiveDataViewModel by viewModels()
    private val eventLiveDataViewModel: EventLiveDataViewModel by viewModels()
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityFirstBinding.inflate(layoutInflater)
        setContentView(binding.root)

        lifecycleScope.launch {
            liveDataViewModel.someData.observe(this@FirstActivity) {
                Timber.d("data observed: $it")
                startActivity(Intent(this@FirstActivity, SecondActivity::class.java))
            }
        }

        lifecycleScope.launch{
            eventLiveDataViewModel.someData.observe(this@FirstActivity) {
                Timber.d("data observed: ${it.getContentIfNotHandled()}")
                startActivity(Intent(this@FirstActivity, SecondActivity::class.java))
            }
        }

        binding.liveDataButton.setOnClickListener {
            liveDataViewModel.setData()
        }

        binding.liveEventDataButton.setOnClickListener {
            eventLiveDataViewModel.setData()
        }
    }

    override fun onResume() {
        super.onResume()
        Timber.d("liveDataViewModel Data: ${liveDataViewModel.someData.value}")
        Timber.d("eventLiveDataViewModel Data: ${eventLiveDataViewModel.someData.value?.getContentIfNotHandled()}")
    }
}

```

When you click the eventLiveDataViewModel button, the following log is output.

![https://blog.kakaocdn.net/dn/20RRk/btr5Uj7KAeu/dtaVP362fPh7WqD849zBm1/img.png](https://blog.kakaocdn.net/dn/20RRk/btr5Uj7KAeu/dtaVP362fPh7WqD849zBm1/img.png)

Null is output when the screen is first opened.

After being observed once, even if you go to a different activity and return, the value is **null again**.

This is because the value is returned as null once it is observed, thanks to the EventWrapper.

If you use peekContent instead of getContentIfNotHandled to get the value, you can use it just like you would with a normal LiveData.

---

### **Summary of Ways to Receive viewModel Events in MVVM**

1. Handle events using only LiveData
2. Wrap EventFlow in LiveData to handle it

In the next posting, we'll take a look at SingleLiveData and SharedFlow.