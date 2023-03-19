---  
layout: post  
title: "What is Reflection in kotlin and How to use it. for beginners"  
categories: kotlin
tags: kotlin
comments: true
showAds: true
description: Reflection is a technique that identifies classes of programs at runtime and is primarily implemented as an annotation. Incorrect use can lead to performance degradation.
---  

### **What is Reflection in Kotlin?**
- What is Kotlin Reflection?
- Accessing Reflection
- Creating Objects with Reflection
- Executing Functions of Objects Created with Reflection
- Accessing Variables of Objects Created with Reflection

---

## **Adding Dependencies (Libraries) for Using Reflection in Kotlin**

To use Reflection in Kotlin, you need to add the following dependencies:

If you are using Android Studio:

``` kotlin
implementation "org.jetbrains.kotlin:kotlin-reflect:{kotlin_version}"

```

If you are using InteliJ IDEA, the library is already included in the IDE, so you don't need to add it separately.

---

## **Creating a Custom Annotation Class for Reflection Use**

Create a class with the annotation to be used in Reflection as follows.

(In fact, this part works well without it, but it is an example because annotations use Reflection a lot.)

``` kotlin
// Create custom annotations.
annotation class CustomAnnotation

// Class with one main constructor.
class MyClass(val first: Int) {
    // Two member variables of the class.
    private val immutableSecret: Int = 2
    private var mutableSecret: Int = 3

    // A function to add two numbers.
    fun make(second: Int): Int {
        return first + second
    }

    // A private function to add two member variables.
    private fun secretMake(): Int {
        return immutableSecret + mutableSecret
    }
}

```

---

## **Accessing Reflection**

The object used to use Reflection in Kotlin is called a **reference object**.

There are two ways to create such a reference object, **using an instance** or **using a class**.

The following code obtains a KClass by using ::class and accesses the **object** reference.

``` kotlin
val myClass = MyClass(5)
// KClass is a generic type, so you need to handle inheritance with out.
val kClass: KClass<out MyClass> = myClass::class

```

> Note that ::class and ::class.java are different.
> 

You can also create it as follows without creating an instance object.

``` kotlin
val kClass: KClass<MyClass> = MyClass::class

```

You can use this KClass type Reflection reference object to check the following information:

``` kotlin
val myClass = MyClass(5)
// KClass is a generic type, so you need to handle inheritance with out.
val kClass: KClass<out MyClass> = myClass::class

// qualifiedName = MyClass
println("qualifiedName = ${kClass.qualifiedName}")
// isAbstract = false
println("isAbstract = ${kClass.isAbstract}")
// isCompanion = false
println("isCompanion = ${kClass.isCompanion}")
// isData = false
println("isData = ${kClass.isData}")
// isFinal = true
println("isFinal = ${kClass.isFinal}")
// typeParameters = []
println("typeParameters = ${kClass.typeParameters}")
// functions = [fun MyClass.make(kotlin.Int): kotlin.Int, fun MyClass.secretMake(): kotlin.Int, fun MyClass.equals(kotlin.Any?): kotlin.Boolean, fun MyClass.hashCode(): kotlin.Int, fun MyClass.toString(): kotlin.String]
println("functions = ${kClass.functions}")
// primaryConstructor = fun <init>(kotlin.Int): MyClass
println("primaryConstructor = ${kClass.primaryConstructor}")
// memberProperties = [val MyClass.first: kotlin.Int, val MyClass.immutableSecret: kotlin.Int, var MyClass.mutableSecret: kotlin.Int]
println("memberProperties = ${kClass.memberProperties}")
// annotations = [@CustomAnnotation()]
println("annotations = ${kClass.annotations}")

```

---

## **Creating Objects with Kotlin Reflection**

There are two types of classes in Reflection object creation.

1. With a default constructor
2. Without a default constructor

---

### **Creating Objects with Reflection When There Is a Default Constructor**

If there is a default constructor, an error occurs when you try to create an object with Reflection **without the default constructor**.

Let's look at an example of how an error occurs.

Note that you can create a Reflection object using the **createInstance** method.

``` kotlin
fun main() {
    val myClass = MyClass(5)
    // KClass is a generic type, so you need to handle inheritance with out.
    val kClass: KClass<out MyClass> = myClass::class
    val instance = kClass.createInstance()
    // Error: Exception in thread "main" java.lang.IllegalArgumentException: Class should have a single no-arg constructor: class MyClass
    println("instance = $instance")
}

```

MyClass has a primary constructor called **first**, but an **IllegalArgumentException** error occurs because we tried to create it with Reflection.

So, you can make it by giving a separate **primaryConstructor** using the **call** method as follows.

``` kotlin
fun main() {
    val kClass: KClass<MyClass> = MyClass::class
    val primaryConstructor = kClass.primaryConstructor
    // Force the primary constructor with call.
    val instance = primaryConstructor?.call(5)

    println("MyClass first = ${instance?.first}") // MyClass first = 5
}

```

> If there is no default constructor, you can create an object with Reflection without the call method.
> 

---

## **Executing Functions with Kotlin Reflection**

Let's use the object created above to execute functions in MyClass.

Note that MyClass has the following two functions:

1. A general fun function
2. A private function

Normally, only function 1 can be called, but if you create an instance object with Reflection, you can also call private function 2.

---

### **Calling a Private Function from a Reflection Object**

When you use functions in the kClass object created above, you can get all the functions in the class.

So, you can access private functions as follows:

``` kotlin
fun main() {
    val kClass: KClass<MyClass> = MyClass::class
    val primaryConstructor = kClass.primaryConstructor
    val instance = primaryConstructor?.call(5)

    // Find a function with the name "secretMake" from all the functions in the kClass.
    val secretMake: KFunction<*>? = kClass.functions.find { it.name == "secretMake" }
    // Since it is a private function, it is forcibly set to be accessible.
    secretMake?.isAccessible = true
    // Call the function.
    println("secretMake() = ${secretMake?.call(instance)}")
}

```

---

### **Calling a Function with Parameters from a Reflection Object**

So, what if you need parameters for a function?

You can call it as follows:

``` kotlin
fun main() {
    val kClass: KClass<MyClass> = MyClass::class
    val primaryConstructor = kClass.primaryConstructor
    val instance = primaryConstructor?.call(5)

    // Call the function with call.
    println("make() = ${instance?.call("make", 10)}")
}

```

---

## Accessing and Modifying Variables with Kotlin Reflection

Finally, let's take a look at accessing member properties within a class object.

For member properties, the way to access them is slightly different depending on whether they are `val` or `var`.

More specifically, while both can be obtained in the same way, the code for modifying or inserting values is slightly different.

---

### Obtaining Member Property Values

You can obtain member property values using `memberProperties`.

Since `MyClass` has only private member variables, you need to set `isAccessible` just like with functions.

The following code uses reflection to `get` the member variables of the class:

``` kotlin
fun main() {
    val kClass: KClass<MyClass> = MyClass::class
    val primaryConstructor = kClass.primaryConstructor
    val instance = primaryConstructor?.call(5)

    val immutableSecret = kClass.memberProperties.find { it.name == "immutableSecret" }
    immutableSecret?.isAccessible = true
// immutableSecret = 2
    println("immutableSecret = ${immutableSecret?.get(instance!!)}")

    val mutableSecret = kClass.memberProperties.find { it.name == "mutableSecret" }
    mutableSecret?.isAccessible = true
// mutableSecret = 3
    println("mutableSecret = ${mutableSecret?.get(instance!!)}")
}

```

`val` uses `KProperty`, while `var` uses `KMutableProperty`, so the reference classes are different.

Therefore, for modifying `var` variables, you can use smart casting.

``` kotlin
fun main() {
    val kClass: KClass<MyClass> = MyClass::class
    val primaryConstructor = kClass.primaryConstructor
    val instance = primaryConstructor?.call(5)

    val immutableSecret = kClass.memberProperties.find { it.name == "immutableSecret" }
    immutableSecret?.isAccessible = true
// immutableSecret = 2
    println("immutableSecret = ${immutableSecret?.get(instance!!)}")

    val mutableSecret = kClass.memberProperties.find { it.name == "mutableSecret" }
    mutableSecret?.isAccessible = true
// mutableSecret = 3
    println("mutableSecret = ${mutableSecret?.get(instance!!)}")

// treat mutableSecret as a KMutableProperty1.if (mutableSecret is KMutableProperty1) {
        mutableSecret.setter.call(instance, 10)
// mutableSecret = 10
        println("mutableSecret = ${mutableSecret.get(instance!!)}")
    }
}

```

Unfortunately, even with reflection, you cannot modify `val` variables.

---

## Caution When Using Kotlin Reflection

When using Android Studio, there is a possibility of code obfuscation, such as when using `minifyEnabled` as shown below:

``` kotlin
android {
    ..
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    ..
}

```

Using `minifyEnabled` changes the names of classes and functions defined in the code to random names.

(e.g. `a class`, `fun b`, etc.)

However, when using reflection, classes and functions that use reflection should not be obfuscated.

As seen above, this is because they are distinguished by name.

In this case, you can use `minifyEnabled` and define `proguard` to specify the packages that should not be obfuscated.

As shown in the code above, the file `'proguard-rules.pro'` contains the locations, and the location of the file is as follows:

![https://blog.kakaocdn.net/dn/bzoIik/btr2OQ91PIA/GC3ISpommINi7bGdaUTKL0/img.png](https://blog.kakaocdn.net/dn/bzoIik/btr2OQ91PIA/GC3ISpommINi7bGdaUTKL0/img.png)

For example, if all data classes in the `data` package use reflection:

Adding the following `-keep` statement to `proguard-rules.pro` will prevent obfuscation of all classes and functions in the package:

> -keep class com.sample.activity.data.* { *;}
> 

In addition to this method of specifying packages at once, you can use the `@Keep` annotation to prevent obfuscation of individual classes or functions.

`@Keep data class TestData(val blogName: String)`



---

### **Summary**
**It is a technology used to inspect a program's classes at runtime.**

In other words, you can understand the internal structure of an object through instances and so on while the program is running.

Annotations are a representative example.

However, if you use it excessively, you may cause **performance degradation** because you have to inspect it every time you call a function or create an object.
