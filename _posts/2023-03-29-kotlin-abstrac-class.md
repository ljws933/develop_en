---  
layout: post  
title: "What is abstract class in Kotlin and how do I use it?"  
categories: kotlin
tags: kotlin
comments: true
showAds: true
description: In Kotlin, the abstract keyword has the following characteristics. An abstract class cannot directly create an object unlike a regular class. An abstract method is declared without an implementation.
---  

**In Kotlin, the abstract keyword has the following characteristics:**
- An abstract class cannot directly create an object unlike a regular class.
- An abstract method is declared without an implementation.

**Therefore, abstract is mainly used in the following situations:**

- When defining an abstract class or an abstract method.

---

## **Example of defining an abstract class and method**

An abstract class and method can only be used through inheritance, as shown below:

``` kotlin
// Declare an abstract class
abstract class Shape {
    // Declare an abstract method
    abstract fun draw()
}

// A subclass that inherits the Shape class
class Circle : Shape() {
    override fun draw() {
        println("Draw a circle.")
    }
}

// A subclass that inherits the Shape class
class Rectangle : Shape() {
    override fun draw() {
        println("Draw a rectangle.")
    }
}

fun main() {
    val circle = Circle()
    circle.draw()// Draw a circle.
    
    val rectangle = Rectangle()
    rectangle.draw()// Draw a rectangle.
}

```

In the above code, the Shape class is an abstract class and is inherited by the Circle and Rectangle classes.

Also, the draw() method in the Shape class is being overridden.

This way, abstract classes are very useful in implementing polymorphism in object-oriented programming.

---

## **Then, what is the difference between open class and abstract class?**

First of all, their similarities are:

- Both can be inherited and used for implementing polymorphism.

However, the biggest difference is that:

**An open class can also be a regular class, so it can be used as a standalone object.**

For example, open class can be used as follows:

``` kotlin
open class Animal {
// Unlike abstract, it can have a default valueopen fun makeSound() {
        println("The animal makes a sound")
    }
}

class Dog : Animal() {
    override fun makeSound() {
        println("Woof!")
    }
}

fun main() {
    // The Animal class can be used as a stand-alone object
    Animal().makeSound()  // The animal makes a sound
    Dog().makeSound()  // Woof!
}

```

As shown in the above code, since the open class Animal is also a regular class,

it can be used as a standalone object to create an object.

However, if an abstract class is used as shown above,

### **an error will occur.**

``` kotlin
// Declare an abstract class
abstract class Shape {
    // Declare an abstract method
    abstract fun draw()
}

fun main() {
    // error: Cannot create an instance of an abstract class
    val shape = Shape()
}

```

---

### Conclusion

**In Kotlin, the abstract keyword has the following characteristics:**

- An abstract class cannot directly create an object unlike a regular class.
- An abstract method is declared without an implementation.

The biggest difference between open class and abstract class is that

**an open class can also be a regular class, so it can be used as a standalone object, but abstract can only be inherited and overridden.**