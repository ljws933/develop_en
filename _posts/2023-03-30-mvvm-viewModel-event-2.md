---  
layout: post  
title: "How can I get viewModel event with MVVM - 2"
categories: android
tags: android 
comments: true
showAds: true
description: There is many way to get viewModel event. liveData, EventWrapper, SingleLiveData etc. Let you know how to use, how to get these event with MVVM architecture.
---

### **Ways to Receive ViewModel Events in MVVM**
1. Process events with SingleLiveData
2. Process events with StateFlow and SharedFlow
3. Process events with SharedFlow and Sealed class
4. Process events with SharedFlow, Sealed class, and LifeCycle
5. Process events with EventFlow, Sealed class, and LifeCycle

---

## **Receiving Data Events with SingleLiveData**

The previous method of receiving viewModel events using **EventWrapper** was very tricky because it required wrapping the liveData variable and input/output data every time.

Therefore, SingleLiveData was used to obtain a **pre-wrapped viewModel**.

### **Defining SingleLiveData**

It is the same as using **EventWrapper**.

```kotlin
abstract class SingleLiveData<T> {
    // Wrap as an event
    private val liveData = MutableLiveData<Event<T>>()

    protected constructor()

    protected constructor(value: T) {
        liveData.value = Event(value)
    }

    protected open fun setValue(value: T) {
        liveData.value = Event(value)
    }

    protected open fun postValue(value: T) {
        liveData.postValue(Event(value))
    }

    // Get the value that is already stored = the same as before
    fun getValue() = liveData.value?.peekContent()

    // Get only the first value that was entered
    fun getNotHandledValue() = liveData.value?.getContentIfNotHandled()

    // Observe the value that is already stored. = the same as before
    fun observePeek(owner: LifecycleOwner, onResult: (T) -> Unit) {
        liveData.observe(owner) { onResult(it.peekContent()) }
    }

    // Observe only the first value that was entered.
    fun observe(owner: LifecycleOwner, onResult: (T) -> Unit) {
        liveData.observe(owner) { it.getContentIfNotHandled()?.let(onResult) }
    }
}

```

### **Defining MutableSingleLiveData**

Define MutableSingleLiveData to use SingleLiveData.

```kotlin
class MutableSingleLiveData<T> : SingleLiveData<T> {

    constructor() : super()

    constructor(value: T) : super(value)

    public override fun postValue(value: T) {
        super.postValue(value)
    }

    public override fun setValue(value: T) {
        super.setValue(value)
    }
}
```

### **Defining SingleViewModel**

The viewModel is defined in the same way as before.

```kotlin
class SingleViewModel : ViewModel() {
    private val _someData = MutableSingleLiveData<Int>()
    val someData: SingleLiveData<Int> = _someData

    fun setData() {
        if (_someData.getNotHandledValue() == null) {
            _someData.setValue(0)
        } else {
            _someData.setValue(_someData.getNotHandledValue()!! + 1)
        }
    }
}

```

With EventWrapper and SingleLiveData, you can choose whether to receive a liveData that is initialized every time or a maintains its data.

---

## **Receiving viewModel Events with SharedFlow**

However, there is a problem when using only liveData with clean architecture.

In clean architecture, each layer is separated, and viewModel is located in the Presenters layer.

That means it should not have anything to do with a specific platform.

That is, it should be maintained without any code that is **import android.xxx**.

This is also mentioned in a blog post by a Google employee.

> Don’t let ViewModels (and Presenters) know about Android framework classe
> 

{% linkpreview "https://medium.com/androiddevelopers/viewmodels-and-livedata-patterns-antipatterns-21efaef74a54" %}

However, the LiveData used so far is a technology that belongs to the Android package.

```kotlin
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
```

However, if you use sharedFlow, you can eliminate this package.

Then implement sharedFlowViewModel as follows.

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
// androidx has been replaced with kotlinx package.
import kotlinx.coroutines.flow.MutableSharedFlow
import kotlinx.coroutines.flow.asSharedFlow
import kotlinx.coroutines.launch

class SharedFlowViewModel : ViewModel() {
    private val _someData = MutableSharedFlow<Int>()
    val someData = _someData.asSharedFlow()

    fun setData() {
        viewModelScope.launch {
            _someData.emit(1)
        }
    }
}
```

This way, you can configure the viewModel without depending on the Android package.

Next, define it in Activity or Fragment as follows.

```kotlin
lifecycleScope.launch {
    sharedFlowViewModel.someData.collect {
        Timber.d("sharedViewModel data observed: $it")
        openSecondActivity()
    }
}

binding.sharedFlowButton.setOnSingleClickListener {
    sharedFlowViewModel.setData()
}

```

With sharedFlow, you cannot directly retrieve the value with get or value, but you can only receive the emitted value with collect.

If you need to directly retrieve the value, you can use **stateFlow**.
