---  
layout: post  
title: "Usage of Scope functions apply, with, let, also, run"  
categories: kotlin
tags: kotlin
comments: true
showAds: true
description: It explains the internal appearance of each Scope function and the difference between apply and with, presents use and application examples, and also explains precautions when using Scope functions.
---  

### **Usage of Scope functions apply, with, let, also, run**
- Internal structure of each Scope function (including differences between apply and with)
- Examples and applications of usage
- Precautions to take when using Scope functions

---

# **Internal Structure of Standard Extension Functions (Scope Functions)**

```
inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    return receiver.block()
}

inline fun <T> T.also(block: (T) -> Unit): T {
    block(this)
    return this
}

inline fun <T> T.apply(block: T.() -> Unit): T {
    block()
    return this
}

inline fun <T, R> T.let(block: (T) -> R): R {
    return block(this)
}

inline fun <T, R> T.run(block: T.() -> R): R {
    return block()
}

```

---

### **with**

> Internal structure
> 

`inline fun <T, R> with(receiver: T, block: T.() -> R): R { return receiver.block() }`

- with is not used as an extension function
- Instead, it is an **explicit receiver** that receives T and must be explicitly passed as a parameter
    - Therefore, this or it is not used
- When returning, it implicitly passes the receiver it executed block on as the extension function of T and returns it
- In other words, it returns the applied object

---

### **apply**

> Internal structure
> 

`inline fun <T> T.apply(block: T.() -> Unit): T { block() return this }`

- also is used as an extension function
- The receiver is implicitly passed as (this)
- Returns the object itself (this) that was applied

---

### **let**

> Internal structure
> 

`inline fun <T, R> T.let(block: (T) -> R): R { return block(this) }`

- It is an extension function that passes the **explicit** receiver (it) as a parameter
- Returns the result of executing block

---

### **also**

> Internal structure
> 

`inline fun <T> T.also(block: (T) -> Unit): T { block(this) return this }`

- It is an extension function that passes the **explicit** receiver (it) as a parameter
    - This is the biggest difference from apply
- Therefore, the object can be specified using this
- Returns the object itself that was extended

---

### **run**

> Internal structure
> 

`inline fun <T, R> T.run(block: T.() -> R): R { return block() }`

- The receiver is implicitly passed as (this)
- Returns the result of executing block
    - The original object is not affected
    - This is the difference from apply

---

# Usage of apply, with, let, also, run

### **apply**

- Used when you want to extend and specify the result

``` kotlin
fun main() {
    val me = Person().apply {
        name = "Nishant"
        age = 19
        gender = 'M'
    }
    println(me)
}
```


### **with**

- Used for almost the same purpose as apply

``` kotlin
fun main() {
    val me = with(Person()) {
        name = "Nishant"
        age = 19
        gender = 'M'
    }
    println(me)
}
```


### **let**

- Useful when you want to use the value in multiple places
- Can also be used for nullPointCheck using ?:(elvis operator)

``` kotlin
fun main() {
    val age = readLine()?.toIntOrNull()

    age?.let {
        println("You are $it years old")
    } ?: println("Wrong input!")
}
```

### **also**

- Useful when you want to use the applied value in a limited place ( { } )

``` kotlin
fun main() {
    Person(
        name = "Nishant",
        age = 19,
        gender = 'M'
    ).also {
        println(it)
    }
}
```

### **run**

- Useful when you don't want to specify one by one with it like let
- That is, when you want to use the internal values of the value in multiple places

``` kotlin
fun main() {
    val me = Person(
        name = "Nishant",
        age = 19,
        gender = 'M'
    )
    me.run {
        println("My name is $name and I am $age years old.")
    }
}
```
---

### **Summary of Scope Functions apply, with, let, also, run**

Summarizing, we have:

- **apply**
    - Used when you want to extend and specify the result
- **with**
    - Used for almost the same purpose as apply
- **let**
    - Useful when you want to use the value in multiple places
    - Can also be used for nullPointCheck using ?:(elvis operator)
- **also**
    - Useful when you want to use the applied value in a limited place ( { } )
- **run**
    - Useful when you don't want to specify one by one with it like let
    - That is, when you want to use the internal values of the value in multiple places

