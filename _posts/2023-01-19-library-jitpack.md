---  
layout: post  
title: "Create and deploy Android libraries as jitpack"  
categories: android
tags: android 
comments: true
showAds: true
description: Learn how to use Android Studios to create your own library, use it in your local environment, and distribute it through jitpack.
---

### Creating an Android library and distributing it with jitpack
- How to create your own library using Android Studio
- How to use the created library in a local environment
- How to distribute the created library with jitpack

## **Creating a library**

### 1. Create a project with an empty activity using the usual method.

![https://user-images.githubusercontent.com/69494230/214218295-cc793065-2dfb-4eae-9ee3-bab2e596efd9.gif](https://user-images.githubusercontent.com/69494230/214218295-cc793065-2dfb-4eae-9ee3-bab2e596efd9.gif)

### 2. Create a new module for the library.
- Create a new module at the top level.
- Make sure the package name is the same as the project's package name.
- Remember the name of the newly created module.

![https://user-images.githubusercontent.com/69494230/214218540-295e95f8-1b4b-4ce0-9dfc-6830fb8b3584.gif](https://user-images.githubusercontent.com/69494230/214218540-295e95f8-1b4b-4ce0-9dfc-6830fb8b3584.gif)

### 3. Add a new file to the new module.
- This file will contain the definition of the library.

![https://user-images.githubusercontent.com/69494230/214219460-3136fd5d-cd6f-41d0-ac9a-c3e59b26ec73.gif](https://user-images.githubusercontent.com/69494230/214219460-3136fd5d-cd6f-41d0-ac9a-c3e59b26ec73.gif)

### 4. Insert simple functionality into the file.

```
object TestLibrary {
    fun showToast(context: Context, message: String) {
        Toast.makeText(context, message, Toast.LENGTH_SHORT).show()
    }
}

```

### 5. Add the library that was just added to the main module's build.gradle.
  - The following is how to add it.
  
* Click File > Project Structure.

![https://user-images.githubusercontent.com/69494230/214242563-e9c653fc-13b5-4958-839c-d865e1c27ed2.png](https://user-images.githubusercontent.com/69494230/214242563-e9c653fc-13b5-4958-839c-d865e1c27ed2.png)

* Click Dependencies > app > +.

![https://user-images.githubusercontent.com/69494230/214242739-4786133d-9fe3-4113-a68f-869b0cfc6c69.png](https://user-images.githubusercontent.com/69494230/214242739-4786133d-9fe3-4113-a68f-869b0cfc6c69.png)

* Click Module Dependency.

![https://user-images.githubusercontent.com/69494230/214242963-d1c1e363-dca9-4be9-87b3-59b10fee7c7c.png](https://user-images.githubusercontent.com/69494230/214242963-d1c1e363-dca9-4be9-87b3-59b10fee7c7c.png)

* Add the newly created library.

![https://user-images.githubusercontent.com/69494230/214243059-3f5b5ef9-ad60-4e36-aae4-cfa3c25f8959.png](https://user-images.githubusercontent.com/69494230/214243059-3f5b5ef9-ad60-4e36-aae4-cfa3c25f8959.png)

* Execute Sync Project with gradle files.
- Note that mine looks like this:

```
dependencies {
    ...
    implementation project(path: ':PermissionCatcherLibrary')
}

```

---

- Once you have done this, you can use the code of the newly created module in the original main module file.

1. Using the created library (locally)
- You can use the locally added library in the default MainActivity as follows:

```
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        TestLibrary.makeToast(this, "Library test")
    }
}

```

---

## **Setting up for distribution of the library**

- As it is now, it can only be used locally and cannot be distributed.
- To distribute it, the following settings are required.

**1. Set the targetSDK to the latest version.**

- To distribute the initial library with jitpack, the targetSDK must be set to the latest version.
    - The latest targetSDK as of now is 33, so change it to that.
    - This needs to be done for both the main app and the library's build.gradle.

```
android{
    compileSdk 33
    .....
          defaultConfig {
              minSdk 26
              targetSdk 33
              ....
     }
}

```

**2. Set up the necessary settings in the library's build.gradle.**

### **Modify the Project-level gradle**

Add the following code to the beginning.

```
buildscript {
    dependencies {
        classpath 'com.github.dcendents:android-maven-gradle-plugin:2.1'
    }
}

```

---

### **Modify the App-level gradle**

Modify **plugins** as follows.

```
plugins {
    id 'com.android.library'
    id 'org.jetbrains.kotlin.android'
    id 'maven-publish'
}

```

**Add publishing information here.**

- Add the following code directly below dependencies

```
afterEvaluate {
    publishing {
        publications {
            // Creates a Maven publication called "release".
            release(MavenPublication) {
                // Applies the component for the release build variant.
                from components.release

                // You can then customize attributes of the publication as shown below.
                groupId = 'your github name'  =  Example: mmol93
                artifactId = 'name of the library to be released'  =  Example: TestLibrary
                version = 'version'  =  Example: 1.0.0
            }
            // Creates a Maven publication called “debug”.
            debug(MavenPublication) {
                // Applies the component for the debug build variant.
                from components.debug

                groupId = 'your github name'  =  Example: mmol93
                artifactId = 'name of the library to be released'  =  Example: TestLibrary
                version = 'version'  =  Example: 1.0.0
            }
        }
    }
}

```

---

## **Uploading the project to Github**

- To distribute with jitpack, the following steps are required on Github:
    1. Upload to Github - we will omit the explanation of this part.
    2. Release

### Release on Github

You can release using **Create a new release** as shown in the following video.

![https://user-images.githubusercontent.com/69494230/214467488-815afb54-01c0-41f0-9e78-a3bca1802d9a.gif](https://user-images.githubusercontent.com/69494230/214467488-815afb54-01c0-41f0-9e78-a3bca1802d9a.gif)

You can see that the release is activated as shown below.

![https://user-images.githubusercontent.com/69494230/214487380-aea7fe12-be58-4735-b867-c82069b7cda1.png](https://user-images.githubusercontent.com/69494230/214487380-aea7fe12-be58-4735-b867-c82069b7cda1.png)

The code below is added to **setting.gradle**.

```
 maven { url 'https://jitpack.io' }
```

And you can add **dependency** that you just created.