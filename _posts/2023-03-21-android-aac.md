---  
layout: post  
title: "What is an essential AAC for using Android MVVM?"
categories: android
tags: android 
comments: true
showAds: true
description: AAC came out to use the existing MVVM more easily and accurately. In other words, it is easy to think that AAC is not an architecture, but a tool for better use of MVVM.
---

AAC is a library collection that was created to simplify and improve the use of MVVM architecture. It is not an architecture itself, but rather a tool for using MVVM more efficiently and accurately. The blog post linked below explains what MVVM is and its background.

[What is Android MVVM architecture?](https://mmol93-developer.com/android/2023/03/17/android-mvvm/)

Android Architecture Components (AAC) is a library created to further simplify and improve the use of MVVM. Jetpack is commonly associated with AAC. The structure of AAC is briefly explained below.

### View

The View interacts with the user and displays the UI.

### ViewModel

The ViewModel receives data from the Repository and transforms it into the appropriate format for the UI.

### LiveData

LiveData detects the lifecycle of the View and updates the corresponding data.

### Repository

The Repository retrieves data from the local database (Room).

### Room

Room is an AAC that is used to create a local database.

The official AAC library released by Google in 2017 consists of the following:

1. Lifecycles (Easy handling lifecycles)
2. LiveData (Lifecycle aware observable)
3. ViewModel (Managing data in a lifecycle)
4. Room (Object mapping for SQLite)
5. Paging (Gradually loading information)
6. Data binding Navigation WorkManager

Each of the libraries has its own characteristics.

### Lifecycles

Lifecycles manage the lifecycle of each UI. It consists of two types: Lifecycle Owner and Lifecycle Observer.

### LiveData

LiveData is used as an observable pattern and manages data according to the Android Lifecycle. It automatically manages activity-related processing according to the lifecycle of Activity or Fragment.

### ViewModel

ViewModel is designed to store and manage UI-related data considering the lifecycle. It preserves data even when the configuration changes occur.

### Room

Room is a SQLite object mapping library that allows you to easily convert SQLite table data into Java objects.