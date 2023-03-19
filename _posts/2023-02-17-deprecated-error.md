---  
layout: post  
title: "[Solved] Deprecated Gradle features were used in this build, making it incompatible with Gradle 8.0"  
categories: android
tags: android 
comments: true
showAds: true
description: There are times when focus automatically goes to editText on Android Kotlin. At this time, the focus is automatically prevented.
---

### Deprecated Gradle features were used in this build, making it incompatible with Gradle 8.0. How to resolve it
- Occurred during build
- Occurred when running command line in terminal

---

## Solution

Actually, I think Android Studio already suggests a solution.

> You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.
> 

In other words, ignore compatibility with Gradle and run by giving the option '--warning-mode all'.

You can do the following:

Open Android Studio's Preferences on a Mac.

Then go to **Build, Execution, Deployment -> Compiler**

There is a Command line Options and you can put the following code:

```
--warning-mode=all --stacktrace

```

![https://blog.kakaocdn.net/dn/bc9Ubu/btrZJcNe5mM/OH6kOnKcW9URduWh0LlVNk/img.png](https://blog.kakaocdn.net/dn/bc9Ubu/btrZJcNe5mM/OH6kOnKcW9URduWh0LlVNk/img.png)

After that,

Press file -> Invalidate Caches / Restart

Reset the cache and restart it.

You can see that it works normally.