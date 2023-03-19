---  
layout: post  
title: "Defining and using Kotlin inline classes and functions"  
categories: kotlin
tags: kotlin
comments: true
showAds: true
description: Inline is the direct use of defined classes instead of calling a function or class in the body, which is usually used when creating a wrapping class. However, using inline classes is not meaningful when boxing processes occur frequently.
---  

### What is Variance?
- A concept that explains the relationship between types when the base type is the same but the type arguments are different.
- In the code below, List is the base type, and `<String>`, `<Any>` are the type arguments.

```
List<String>, List<Any>

```

- However, even though String is a subtype of Any, `List<String>` is not a subtype of `List<Any>`.
- Therefore, the following code produces an error:

``` kotlin
fun addStringList(list: MutableList<Any>) {
    list.add("text")
}

fun addNumberList() {
    val numbers = mutableListOf(1, 2, 3)
    // Error occurs here
    addStringList(numbers)
}

```

- If the above code did not produce an error, unintended type arguments could be added to the list.
- Thus, although it may look like there is a subtype relationship, **types that actually have no relationship with each other are called invariant.**

⇒ This **invariance** only occurs in **mutable collections.**

Therefore, **immutable collections do not experience invariance.**

``` kotlin
fun addNumberList() {
    val numbers =mutableListOf(1, 2, 3)
    // Error occurs here
    addStringMutableList(numbers)

    // Success with List<Any>
    val anyList =listOf<Any>(0, "2")
}

```

---

## Why does covariance and invariance occur differently depending on whether a collection is immutable or mutable?

- **Firstly, there are three types of generic types:**
1. Generic type producers that provide operations that only return an operation of type T and do not provide operations that receive an input of type T. = **i.e., when the generic type only acts as a producer.**
2. Generic type consumers that only receive input of type T and never return output of type T. = **i.e., when the generic type only acts as a consumer.**
3. Types that are not included in either of the above two categories. = If all sub-values can be inserted, it is impossible to maintain a subtype relationship without breaking type safety, which is why it is difficult to use mutable collections.

For example:

``` kotlin
fun test() {
    val stringNode = TreeNode("Hello")
    // Error occurs due to covariance
    val anyNode: TreeNode<Any> = stringNode
    // There is no error in the action of receiving values itself
    anyNode.addChild(123)
}

```

> So why are immutable types like List<T> covariant?
> 

Immutable collections **only produce T type values and do not consume them.**

Therefore, if it is defined as List<Any>, it just returns Any.

Thus, most immutable collections maintain covariance.

---

## Assigning covariance and contravariance using in and out projections

- As seen in the example above, even though Int inherits from Number and Int's superclass is Number, there is no inheritance relationship in generics. = **invariance**
- In general use, Int inherits from Number. = **covariance**

---

### Out Projections

Out projections can be used to change from **invariance to covariance.**

``` kotlin
class Rectangle<out T: Number>(val width: T, val height: T) {
}

fun main(args: Array<String>) {
    // Declare Double for generics
    val derivedClass = Rectangle<Double>(10.5, 20.5)
    // Rectangle<Double> is recognized as a subclass of Rectangle<Number>
    // So no error occurs
    val baseClass : Rectangle<Number> = derivedClass
}

```

### In Projections

In is the opposite of out.

That is, the two types can be reversed.

For example, in the previous example,

Rectangle<Double> was a subclass of Rectangle<Number>,

but using in projections,

Rectangle<Number> can be a subclass of Rectangle<Double>.

``` kotlin
class Rectangle<in T: Number>(val width: T, val height: T) {
}

fun main(args: Array<String>) {
    val baseClass = Rectangle<Number>(10.5, 20.5)
    // Number class can be put into Double
    val derivedClass : Rectangle<Double> = baseClass
}

```

This is called **contravariance**.

---

### Summary

- Variance: A concept that explains the relationship between types when the base type is the same but the type arguments are different.
- Why covariance and invariance occur differently depending on whether a collection is immutable or mutable is due to the role of generics.
- Covariance and contravariance can be assigned freely using in and out projections.

![https://blog.kakaocdn.net/dn/Tk18r/btrZJDyj6XZ/RKnDddleJ8JAMpKR0Coswk/img.png](https://blog.kakaocdn.net/dn/Tk18r/btrZJDyj6XZ/RKnDddleJ8JAMpKR0Coswk/img.png)
