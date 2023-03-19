---  
layout: post  
title: "Automatically publish a license using LicenseTools Plugin"  
categories: android
tags: android 
comments: true
showAds: true
description: LicenseTools Plugin is an open source created by Cookpad that automatically generates license information for your library, prints it out to a file in HTML format, and displays it on the screen.
---

### What is LicenseToolsPlugin
- Automatically generates license information for the libraries in use
- The generated file is output as an html file
- License is disclosed by displaying the output html file on the screen
- An open source created by a company called Cookpad in Japan

### LicenseToolsPlugin GitHub address

{% linkpreview "https://github.com/cookpad/LicenseToolsPlugin" %}

---

### Declaration of necessary Dependency

``` kotlin
// Declare in the project level gradle
buildscript {
  repositories {
    maven {
      url "<https://plugins.gradle.org/m2/>"
    }
  }
  dependencies {
    classpath "gradle.plugin.com.cookpad.android.plugin:plugin:1.2.6"
  }
}

```

```
// Declare in the app level gradle
apply plugin: "com.cookpad.android.plugin.license-tools"

```

---

## How to use

## How to use

### Creating a Licenses.yml file

1. Enter the following in the Android Studio terminal
    - ./gradlew updateLicenses
2. If successful, you can see that the ./app/licenses.yml file has been created
    - You need to set the view to Project rather than Android to see it.
    - If failed, try ./gradlew checkLicenses first and try again.

![https://blog.kakaocdn.net/dn/dbhUyi/btrUfQQEIel/AkcH2qSaQCHwor1o2Q6cnk/img.png](https://blog.kakaocdn.net/dn/dbhUyi/btrUfQQEIel/AkcH2qSaQCHwor1o2Q6cnk/img.png)

### Checking the generated file

1. Enter the following in the Android Studio terminal
    - ./gradlew checkLicenses
2. If successful, you can see the BUILD SUCCESSFUL message.

### Creating the licenses.html file

1. Enter the following in the Android Studio terminal
    - ./gradlew generateLicensePage
2. If successful, you can see that the ./app/src/main/assets/licenses.html file has been created.
    - You need to set the view to Project rather than Android to see it.

---

## Issues and Precautions that may arise during LicenseToolsPlugin setup

- Process of creating license files
    1. Create the licenses.yml file through updateLicenses.
    2. ./gradlew generateLicensePage creates a licenses.html file by referring to licenses.yml.
    3. Therefore, most problems arise due to the contents of licenses.yml.
- When the error "Could not generate the copyright statement" occurs
    - Add skip: true to the corresponding artifact.
        - This means that the license generation of the library will not be done when generating.
    - Or put a value in copyrightHolder.
        - If it was first created, copyrightHolder contains meaningless values such as #COPYRIGHT_HOLDER#.
        - However, you can delete this and put the name of the company or individual who actually owns or created the library, such as Google.
- LicenseToolsPlugin does not set up all libraries.
    - Check which libraries were automatically set up as html through the app after outputting.

---

## Displaying the generated html file

- Simply showing the generated html file is the end.
- It will look like the following html.

<img src="https://blog.kakaocdn.net/dn/tjzDe/btrUh4a00sw/DtTE0fjTu8wuku5z1YnGyk/img.jpg" width="450" alt="LicenseToolsPlugin">