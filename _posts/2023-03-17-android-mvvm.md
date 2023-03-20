---  
layout: post  
title: "what is Android MVVM architecture and why it is needed?"
categories: android
tags: android 
comments: true
showAds: true
description: With the advent of ViewModel, which shows the status of View, complete separation between View and ViewModel became possible. Therefore, in MVVM architecture, ViewModel does not work on UI. VIewModel only updates the data, and View only shows the data.
---

### **Android MVVM Architecture**
- Comparison of MVC vs MVP vs MVVM
- The reason for the emergence and necessity of MVVM
- Method of implementing MVVM

---

## **MVC: Activity and Fragment are responsible for View and Controller**

In the MVC pattern, the View and Controller are connected as shown below.

![https://blog.kakaocdn.net/dn/cSMa5Y/btr4jKTnasG/YEB8Zdvkvc3KrySdA7lQY0/img.png](https://blog.kakaocdn.net/dn/cSMa5Y/btr4jKTnasG/YEB8Zdvkvc3KrySdA7lQY0/img.png)

In other words, the Controller is implemented in the Activity or Fragment.

When using this architecture, the following problems can arise:

- If the lifecycle of the Activity changes, it affects the View or Controller implemented inside the Activity.
- As the scale increases, there are many cases where you have to change multiple parts when changing one place.
- If you modify the View, you have to modify the Controller as well.
- If you modify the Controller, you have to modify the Model as well.

To solve these problems, an architecture that pursues **loose coupling** has emerged.

- MVP
- MVVM

---

## **MVP: The appearance of Presenter that loosely couples with View**

In the MVP architecture, unlike the MVC pattern where the Controller is defined in the View,

Presenter is introduced and takes over the role of the Controller.

![https://blog.kakaocdn.net/dn/H4NTy/btr4gUP1cBG/bYfqTzrbLaysLvuo5mvjt0/img.png](https://blog.kakaocdn.net/dn/H4NTy/btr4gUP1cBG/bYfqTzrbLaysLvuo5mvjt0/img.png)

In other words, it is as follows:

> View handles the user interface, and Model handles the data.
> 

---

However, the MVP pattern also causes the following problems:

- As the role of the Presenter grows, the Presenter class becomes complex.
    - As a result, the maintenance of the Presenter becomes difficult.
- The MVP pattern is difficult to manage the state change between View and Model.
    - If you change the state of the Model according to the event that occurs in the View, you need to handle a lot of work in the Presenter to reflect it in the View.

To solve this, the **MVVM pattern** emerged.

---

## **MVVM: The emergence of ViewModel that observes the state of View**

With the emergence of ViewModel, which can know the state of View, complete separation between View and ViewModel became possible.

![https://blog.kakaocdn.net/dn/lJrrY/btr4jnYfIJa/USxVAXWOQjY9aEobn9CG3k/img.png](https://blog.kakaocdn.net/dn/lJrrY/btr4jnYfIJa/USxVAXWOQjY9aEobn9CG3k/img.png)

Therefore, ViewModel does not perform UI-related work in the MVVM architecture.

ViewModel only updates data, and View only shows the data.

This provides the following benefits:

- Reactive programming is possible through state observation.
- The amount of code written has been greatly reduced and UI can be easily changed through viewBinding and dataBinding introduced in ViewModel.
- Complete independence between View and ViewModel is achieved.
    - This makes testing easier.

---

### **Reactive programming is possible through state observation**

In the MVVM architecture, the View subscribes to the Publisher that recognizes the lifecycle, called LiveData, in Activity and Fragment.

As a result, the View performs a behavior that reacts to changes when the data changes.

This allows users to change the data whenever they enter a specific input and display or internally process the changed data on the UI.

---

### **Significant reduction in code and easy UI changes through viewBinding and dataBinding**

In general, there are projects that use both viewBinding and dataBinding in one project, and projects that use only one.

However, you should know their characteristics well, and their characteristics are as follows.

**Characteristics of viewBinding**

- Generates a binding class that maps to the XML file at compile time
- The binding class can reference the View
- Faster performance than the findViewById() method
- Only analyzes the XML layout file when the binding class is created, so unnecessary XML file analysis does not occur at runtime
- Excellent build time performance and memory efficiency

**Characteristics of dataBinding**

- Performs data binding between View and XML file at runtime
- Provides a function that updates the UI by writing expressions directly in the XML file
- Can easily handle UI-related tasks
- Slower performance than the findViewById() method
- Performance degradation may occur as the complexity of data binding increases

---

### **Optimization of memory usage is possible**

One of the biggest differences between the web and mobile is that mobile has a limit on the resources that can be used.

Therefore, you should always think about memory optimization and remove unused elements as much as possible to make them available elsewhere.

If you use the MVVM pattern, you can observe the state of the View and delete it when it is not in use.

However, data is not deleted immediately when the View dies, because it costs money to recreate the data when the View is called again.

Therefore, as shown in the figure below, the data remains in the viewMocelScope for a certain period of time even if the View dies.

![https://blog.kakaocdn.net/dn/bVQMXo/btr4hzx850y/3kUMOV06RW2zzuK9MXae8k/img.png](https://blog.kakaocdn.net/dn/bVQMXo/btr4hzx850y/3kUMOV06RW2zzuK9MXae8k/img.png)
