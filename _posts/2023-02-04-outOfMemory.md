---  
layout: post  
title: "Android OutOfMemoryError: How to resolve Java heap space errors"  
categories: android
tags: android 
comments: true
showAds: true
description: Information on the cause and solution of OutOfMemoryError about Java heap space error in Android Studios.
---

## Resolving OutOfMemoryError: Java heap space error in Android
- Causes of OutOfMemoryError: Java heap space
- Solutions for OutOfMemoryError: Java heap space

---

## Causes of OutOfMemoryError: Java heap space

This error is not often encountered when working on personal projects.

The reason is that it occurs when there is insufficient space in the Heap area.

It occurs when there are large files (Res, Raw) in the project.

Therefore, one solution is to optimize these files, but it is not easy as it takes a lot of time and there is much to do.

However, there is a simple way to solve this problem, which is to increase the Heap area.

---

## Solutions for OutOfMemoryError: Java heap space

Increasing the Heap area can solve the OutOfMemoryError: Java heap space problem, and the following is the method:

- Look for the gradle.properties file.

![https://blog.kakaocdn.net/dn/NsjEG/btrX3txexQm/9t1Ln7DeKPAsr0ca3pGW7k/img.png](https://blog.kakaocdn.net/dn/NsjEG/btrX3txexQm/9t1Ln7DeKPAsr0ca3pGW7k/img.png)


<br>
- Find **jvmargs** and change it to the following:

```
org.gradle.jvmargs=-Xms4096m -Xmx10248m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8

```

![https://blog.kakaocdn.net/dn/bxkCF3/btrX18ABzOD/UHBQg8X3fLDi0d5lMR77Mk/img.png](https://blog.kakaocdn.net/dn/bxkCF3/btrX18ABzOD/UHBQg8X3fLDi0d5lMR77Mk/img.png)

To increase the effect properly, both Xms and Xmx need to be increased.

After this, run the project again to confirm that it is working properly.