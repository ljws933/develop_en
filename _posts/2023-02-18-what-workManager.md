---  
layout: post  
title: "Why do I need Android WorkManager and where should I use it?"  
categories: android
tags: android 
comments: true
showAds: true
description: WorkManager is an element created by Google to help with background processing in Android's Jetpack. As it is a recent technology, many problems and bugs have been fixed, and it is safe to say that most background tasks in recent Android can be done with WorkManager.
---

### Where and how should the Android Kotlin WorkManager be used?
- Why is WorkManager needed?
- Differences with other background processing technologies
- Where can WorkManager be used?
- How to use WorkManager

WorkManager is a component created by Google to assist with background processing in Android's Jetpack.

As it is a recently developed technology, it has fixed various issues and bugs.

It is safe to say that most background tasks in Android can be done using WorkManager.

---

## **Why is WorkManager needed?**

Firstly, it is important to understand

**why apps running in the background on Android are automatically terminated**.

---

### **Memory usage in Android**

Although the Android kernel is based on the Linux kernel,

the biggest difference is that it does not have **Swap Space**.

**What is Swap Space?**

- Used when RAM is full
- Moves inactive data in memory to Swap Space
- Swap Space cannot replace RAM capacity increase
    - Because Swap Space is located on hard drives with slow access times

![https://blog.kakaocdn.net/dn/Ghacm/btrWVKlHfC2/WVRWgBoYgJKTu2A8LKSx50/img.png](https://blog.kakaocdn.net/dn/Ghacm/btrWVKlHfC2/WVRWgBoYgJKTu2A8LKSx50/img.png)

Since (by default) Android does not have Swap Space,

it uses an **OOM killer** to forcibly terminate processes.

The OOM killer forcibly terminates processes based on their **Visible state** and **amount of consumed memory**,

to secure memory.

---

All processes have their own oom_adj score assigned by the activity manager.

This score is determined by the app's state (Foreground, Background, Service, etc.).

And the OOM killer terminates processes based on the following conditions.

> When free memory space is less than x, clean up processes with oom_adj values greater than y!
> 

Therefore,

**The higher the oom_adj value, the more easily the process is cleared by the OOM killer.**

In other words,

> The less memory an app consumes, the less likely the process is to be forcibly terminated.
> 

---

The second thing to understand is

**understanding of application states.**

Basically, if an app wants to do something continuously in the background,

You should use the "Service" component.

> What is a service?
> 

There are several reasons why you should use the service.

1. Notify the system that the process has a long-running task.
2. Assign an appropriate oom_adj score when using the service.
3. You can execute operations separately in a separate process using the service.
4. Services are one of the four major components of Android apps.

However, there is one problem here.

Doing work in a separate process consumes a lot of battery power!!!

Therefore, Google introduced the "Doze mode" and developed it more and more.

(= Puts a tremendous amount of restrictions on background operations)

Official description link for Doze mode

{% linkpreview "https://developer.android.com/training/monitoring-device-state/doze-standby?hl=en" %}

Specifically, the following restrictions have been added to apps that can be registered in the app market.

- August 2018: All newly released apps must have API 26 (Oreo 8.0) or higher.
- November 2018: Existing apps must also have API 26 (Oreo 8.0) or higher.
- After 2019: The targetSdkVersion requirement will be improved every year. Whenever Android releases a new version each year, all apps must target that API level or higher.

Therefore, the job API was created to replace the service.

---

## Differences in other background processing

### Appearance of JobScheduler

``` java
ComponentName service = new ComponentName(this, MyJobService.class);
JobScheduler mJobScheduler = (JobScheduler)getSystemService(Context.JOB_SCHEDULER_SERVICE);
JobInfo.Builder builder = new JobInfo.Builder(jobId, serviceComponent)
 .setRequiredNetworkType(jobInfoNetworkType)
 .setRequiresCharging(false)
 .setRequiresDeviceIdle(false)
 .setExtras(extras).build();
mJobScheduler.schedule(jobInfo);

```

**JobScheduler** schedules started jobs.

At the appropriate time, the system starts **MyJobService** and performs the contents inside **onStartJob()**.

However, **JobScheduler** can only be used in API 21 or higher.

Afterwards, due to too many bugs, it is replaced by **JobDispatcher**.

However, since there are devices that **JobDispatcher** does not work properly,

It was then replaced by **JobIntentService**.

However, **JobIntentService** also had a problem that it does not work exactly at the desired timing.

So **WorkManager** was introduced.

---

### Appearance of WorkManager

When you look at the above content, developers face issues where they must consider various things such as Android version and device model to create a background service.

To solve these problems, **WorkManager** was created.

**WorkManager** has the following characteristics.

- Provides system-based background processing APIs to simplify implementation.
    - This API allows jobs to be performed in the background even when the app is not in the foreground.
- Developers do not have to worry about background processing.
- Therefore, it can replace most of the service functions such as Service and JobIntentService.
- Short tasks and long tasks can both be used.
- Trigger is possible by network status, battery status, etc.
- Can be used even in the foreground.
- Can be used with FCM.

**WorkManager** has the following components.

![https://blog.kakaocdn.net/dn/TMPVe/btrZKZgrXN8/HWCfcIXhTVhgpajRCcDhH0/img.png](https://blog.kakaocdn.net/dn/TMPVe/btrZKZgrXN8/HWCfcIXhTVhgpajRCcDhH0/img.png)

Configuration of WorkManager

- **WorkManager**
    - Puts the job to be processed in its own queue and manages it.
    - Since it is implemented as a singleton, you can receive the instance of WorkManager using getInstance() and use it.
- **Worker**
    - An abstract class that inherits the processing code of the background task that needs to be processed. The code is written by overriding the doWork() method of this class.
        - doWork()
            - Once the job is completed, you must return one of the values of the enum defined within the Worker class called **Result**. There are three values: SUCCESS, FAILURE, RETRY. Depending on the value returned, WorkManager decides whether to finish the job, retry it, or define it as a failure and stop it.
- **WorkRequest**
    - The individual job that will actually be requested through WorkManager.
    - It contains information on how to process this job, such as the work to be done, whether it should be repeated, and the work execution conditions and constraints.
    - Depending on whether it is repeated or not, it is divided into onTimeWorkRequest and PeriodicWorkRequest.
    - onTimeWorkRequest
        - Represents a job request for a job that will not be repeated, that is, a job that will be executed only once.
    - PeriodicWorkRequest
        - Represents a job request for a job that will be executed multiple times.