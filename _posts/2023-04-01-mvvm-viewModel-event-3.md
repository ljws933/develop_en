---  
layout: post  
title: "How can I get viewModel event with MVVM - 3"
categories: android
tags: android 
comments: true
showAds: true
description: There is many way to get viewModel event. liveData, EventWrapper, SingleLiveData etc. Let you know how to use, how to get these event with MVVM architecture.
---

### **How to receive viewModel events in MVVM**
1. Handling events with SharedFlow and Sealed classes
2. Handling events with SharedFlow, Sealed classes, and Lifecycle
3. Handling events with EventFlow, Sealed classes, and Lifecycle

**In the previous blog post,** we looked at the following:

1. Handling events with LiveData only
2. Wrapping EventFlow in LiveData for handling events
3. Handling events with SingleLiveData
4. Handling events with StateFlow and SharedFlow

{% linkpreview "https://mmol93-developer.com/android/2023/03/26/mvvm-viewmodel-event-1/" %}

{% linkpreview "https://mmol93-developer.com/android/2023/03/30/mvvm-viewmodel-event-2/" %}

---

## **Handling events with SharedFlow and Sealed classes**

One issue with using SharedFlow is that you need to define variables for all SharedFlow data.

For example, you need to define multiple variables in the viewModel like this:

> SharedFlowEventViewModel.kt
> 

```kotlin
private val _someData = MutableSharedFlow<Int>()
val someData = _someData.asSharedFlow()

private val _otherData = MutableSharedFlow<String>()
val otherData = _otherData.asSharedFlow()

private val _theOtherData = MutableSharedFlow<Boolean>()
val theOtherData = _theOtherData.asSharedFlow()

private val _eventFlow = MutableSharedFlow<SharedFlowEvent>()
val eventFlow = _eventFlow.asSharedFlow()

```

And you need to perform the same number of collects to receive the SharedFlow.

> Activity
> 

```kotlin
lifecycleScope.launch {
    launch {
        sharedFlowEventViewModel.someData.collect {

        }
    }

    launch {
        sharedFlowEventViewModel.otherData.collect {

        }
    }

    launch {
        sharedFlowEventViewModel.eventFlow.collect {

        }
    }

    launch {
        sharedFlowEventViewModel.theOtherData.collect {

        }
    }
}

```

To solve this issue, you can create a class that sends events and a sealed class that bundles the events, so that you can receive all sharedFlows with a single collect.

> SharedFlowEventViewModel.kt
> 

```kotlin
class SharedFlowEventViewModel : ViewModel() {
    private val _eventFlow = MutableSharedFlow<SharedFlowEvent>()
    val eventFlow = _eventFlow.asSharedFlow()

    // Emit each event.
    fun emitSomeData() {
        emitEvent(SharedFlowEvent.SomeData(1))
    }

    fun emitOtherData() {
        emitEvent(SharedFlowEvent.OtherData("abc"))
    }

    fun emitTheOtherData() {
        emitEvent(SharedFlowEvent.TheOtherData(true))
    }

    // Put the event in the flow.
    private fun emitEvent(sharedFlowEvent: SharedFlowEvent) {
        viewModelScope.launch {
            _eventFlow.emit(sharedFlowEvent)
        }
    }

    // Put received data in each event.
    sealed class SharedFlowEvent {
        data class SomeData(val number: Int): SharedFlowEvent()
        data class OtherData(val text: String): SharedFlowEvent()
        data class TheOtherData(val ox: Boolean): SharedFlowEvent()
    }
}

```

Then, you can trigger and receive events in the Activity as follows:

> Activity
> 

```kotlin
// Receive only one event and perform different actions depending on the event.
lifecycleScope.launch {
    sharedFlowEventViewModel.eventFlow.collect {
        when (it) {
            is SharedFlowEventViewModel.SharedFlowEvent.OtherData -> {
                Timber.d("sharedFlowEventViewModel is collected: $it")
            }
            is SharedFlowEventViewModel.SharedFlowEvent.SomeData -> {
                Timber.d("sharedFlowEventViewModel is collected: $it")
            }
            is SharedFlowEventViewModel.SharedFlowEvent.TheOtherData -> {
                Timber.d("sharedFlowEventViewModel is collected: $it")
            }
        }
    }
}

binding.sharedSealedButton.setOnClickListener {
    sharedFlowEventViewModel.emitSomeData()
    sharedFlowEventViewModel.emitOtherData()
    sharedFlowEventViewModel.emitTheOtherData()
}

```

With this method, you can receive multiple flows with a single collect for one event.

---

## **Handling events with SharedFlow, Sealed classes, and Lifecycle**

The method described above has a problem where it keeps collecting even when the user goes to the background, which is not necessary.

For example:

1. Collect specific flow.
2. User goes to the background while the app is still running.
3. In this state, there is no need to collect the data.

In such cases, you can use the `repeatOnLifecycle` function of the LifecycleOwner to create a function that turns on when the lifecycle is in the Start state and turns off when it's not.

> LifecycleExt.kt
> 

```kotlin
fun LifecycleOwner.repeatOnStarted(block: suspend CoroutineScope.() -> Unit) {
    lifecycleScope.launch {
        lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED, block)
    }
}

```

You can use this function in the Activity as follows:

> Activity
> 

```kotlin
repeatOnStarted {
    sharedFlowEventViewModel.eventFlow.collect {
        when (it) {
            is SharedFlowEventViewModel.SharedFlowEvent.OtherData -> {
                Timber.d("sharedFlowEventViewModel is collected: $it")
            }
            is SharedFlowEventViewModel.SharedFlowEvent.SomeData -> {
                Timber.d("sharedFlowEventViewModel is collected: $it")
            }
            is SharedFlowEventViewModel.SharedFlowEvent.TheOtherData -> {
                Timber.d("sharedFlowEventViewModel is collected: $it")
            }
        }
    }
}

```

---

## **Processing Events with EventFlow & Sealed class Lifecycle**

Using the above method, we were unable to handle the following case:

1. Collect specific data using flow in the app.
2. While collecting, the user moves to the home screen.
3. The collection stops suddenly.
4. When you go back to the app, you need to either handle an error or collect from the beginning.

**The fact that the task stops completely when the user goes to the home screen was a big problem.**

Therefore, when collecting in the background, we created an **EventFlow that temporarily holds the data**.

> EventFlow.kt
> 

```kotlin
interface EventFlow<out T> : Flow<T> {

    companion object {
// Maximum number of stored events emittedconst val DEFAULT_REPLAY: Int = 3
    }
}

interface MutableEventFlow<T> : EventFlow<T>, FlowCollector<T>

@Suppress("FunctionName")
fun <T> MutableEventFlow(
    replay: Int = EventFlow.DEFAULT_REPLAY
): MutableEventFlow<T> = EventFlowImpl(replay)

fun <T> MutableEventFlow<T>.asEventFlow(): EventFlow<T> = ReadOnlyEventFlow(this)

private class ReadOnlyEventFlow<T>(flow: EventFlow<T>) : EventFlow<T> by flow

// By inheriting FlowCollector, you can control the data before emitting it to sharedFlow.private class EventFlowImpl<T>(
    replay: Int
) : MutableEventFlow<T> {

    private val flow: MutableSharedFlow<EventFlowSlot<T>> = MutableSharedFlow(replay = replay)

    @InternalCoroutinesApi
    override suspend fun collect(collector: FlowCollector<T>) = flow
        .collect { slot ->
// If the collected data has not been consumed, emit the data.if (!slot.markConsumed()) {
                collector.emit(slot.value)
            }
        }

    override suspend fun emit(value: T) {
        flow.emit(EventFlowSlot(value))
    }
}

private class EventFlowSlot<T>(val value: T) {

// To ensure stability in multiple threads, use AtomicBooleanprivate val consumed: AtomicBoolean = AtomicBoolean(false)

// Check whether it has been collected at least once.fun markConsumed(): Boolean = consumed.getAndSet(true)
}

```

The newly created **MutableEventFlow** works as follows:

1. Collected data is automatically saved for the replay value.
2. If it was collected while the user was using the app, check if it has already been collected using the markConsumed method.
3. If the user is not using the app, the data is not collected and remains stored in sharedFlow. (The stored data may contain data that has already been collected depending on the timing.)
4. When the user starts using the app again, already collected values are checked to see if they have been collected before, and only values that have never been collected are emitted again.

You can use it in viewModel as follows:

> EventFlowViewModel.kt
> 

```kotlin
class EventFlowViewModel: ViewModel() {
// Change this part to use MutableEventFlow.private val _eventFlow = MutableEventFlow<SharedFlowEvent>()
    val eventFlow = _eventFlow.asEventFlow()

// Trigger each eventfun emitSomeData() {
        emitEvent(SharedFlowEvent.SomeData(1))
    }

    fun emitOtherData() {
        emitEvent(SharedFlowEvent.OtherData("abc"))
    }

    fun emitTheOtherData() {
        emitEvent(SharedFlowEvent.TheOtherData(true))
    }

// Add the event to the flow.private fun emitEvent(sharedFlowEvent: SharedFlowEvent) {
        viewModelScope.launch {
            _eventFlow.emit(sharedFlowEvent)
        }
    }

// Add the received data to each event.sealed class SharedFlowEvent {
        data class SomeData(val number: Int): SharedFlowEvent()
        data class OtherData(val text: String): SharedFlowEvent()
        data class TheOtherData(val ox: Boolean): SharedFlowEvent()
    }
}

```

The usage in Activity or Fragment is the same.

> Activity
> 

```kotlin
repeatOnStarted {
    eventFlowViewModel.eventFlow.collect {
        when (it) {
            is EventFlowViewModel.SharedFlowEvent.OtherData -> {
                Timber.d("EventFlowViewModel is collected: $it")
            }
            is EventFlowViewModel.SharedFlowEvent.SomeData -> {
                Timber.d("EventFlowViewModel is collected: $it")
            }
            is EventFlowViewModel.SharedFlowEvent.TheOtherData -> {
                Timber.d("EventFlowViewModel is collected: $it")
            }
        }
    }
}

```

By defining EventFlow in this way, we have added the ability to handle previously emitted data while optimizing.