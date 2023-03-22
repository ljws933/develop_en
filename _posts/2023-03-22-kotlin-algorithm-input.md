---  
layout: post  
title: "Solving Algorithm Problems with Kotlin - Getting Input"  
categories: kotlin
tags: kotlin
comments: true
showAds: true
description: When coding to solve algorithm problems, the most important and basic thing is receiving input values. In this blog, we will look at how to receive input values and which method is the most effective.
---  

### Solving Algorithm Problems with Kotlin - Getting Input
- Comparison of Scanner and BufferedReader
- Receiving various input values using BufferedReader

When coding to solve algorithm problems, the most important and basic thing is receiving input values.

In this blog, we will look at how to receive input values and which method is the most effective.

---

## Scanner VS BufferedReader

### When using Scanner

When there is the following input value,

> abc
> 
> 123

The output can be printed with the following code if Scanner is used:

``` kotlin
fun main(args: Array<String>) = with(Scanner(System.`in`)) {

    val name = next()
    val age = nextInt()

// name: abc, age: 123
    println("name: $name, age: $age")

}

```

For input values separated by spaces, such as:

> 10 20 30 40 50
> 

The following code can be used to receive values **separately by space**:

``` kotlin
fun main(args: Array<String>) = with(Scanner(System.`in`)) {
    val arr = ArrayList<Int>()
    repeat (5) {
        arr.add(nextInt())
    }
// 10 20 30 40 50
    println("Contents Of Array: $arr")
}

```

Now, let's see how to receive input values using **BufferedReader**.

---

### When using BufferedReader

> abc
> 
> 123

The value can be received with the following code:

``` kotlin
fun main(args: Array<String>) = with(System.`in`.bufferedReader()) {
    val name = readLine()
    val age = readLine().toInt()

// name: abc, age: 123
    println("name: $name, age: $age")
}

```

---

The basic usage of both methods is similar.

However, BufferedReader is mainly used because of the following effect:

### **BufferedReader is faster.**

---

## Applying BufferedReader

Let's take a look at how to use BufferedReader for each case, including the ones shown above.

### **When the given input value is one line and separated by space**

> 10 20 30

``` kotlin
fun main(args: Array<String>) = with(System.`in`.bufferedReader()) {
    val nums = readLine().split(" ").map { it.toLong() }

// [10, 20, 30]
    println(nums)
}

```

### **When the given input value is multiple lines and separated by space - 1**

> 2 (Number of given lines)
> 
> x y (Given value is always 2)
> 
> x y


``` kotlin
fun main(args: Array<String>) {
    val n = readln().toInt()
    val dots = ArrayList<Dot>()
    repeat(n) {
        val (x, y) = readln().split(" ").map { it }
        dots.add(Dot(x, y))
    }
// [Dot(x=x, y=y), Dot(x=x, y=y)]
    println(dots)
}
data class Dot(val x: String, val y: String)

```

### **When the given input value is multiple lines and separated by space**

> 2 (Number of given lines)
> 
> a b c (Given value is fluid)
> 
> d e f


``` kotlin
fun main(args: Array<String>) = with(System.`in`.bufferedReader()) {
    val lines = readLine().toInt()
    val data = ArrayList<String>()

    for(i in 0 until lines){
        data.add(readLine().split(" ").map { it }.toString())
    }

// [[a, b, c], [d, e, f]]
    println(data)
}

```