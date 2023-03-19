---  
layout: post  
title: "Will Android Kotlin recycle if it keeps making the same variables?"  
categories: kotlin
tags: kotlin
comments: true
showAds: true
description: Creates a function that generates a variable each time a function is called, and continues to call the function to determine the memory address value of the variable. This allows you to determine if the same variable is recycled or if you are assigned a new memory address when you create it.
---  

### Will Kotlin recycle the same variable if it is repeatedly created?
1. Create a function that generates variables every time it is called.
2. Check the memory address value of the variable created while continuously calling the function.
3. Check whether the same variable is reused or a new address is assigned through the address value.

---

## Writing Test Code

Basically, since Kotlin **does not have pointers** like C,

**It is not possible to obtain the address value where the variable is stored...**

However, in case of **collections**, it is possible to output the address value,

So, as in the example below, we create a variable (list) every time a button is clicked.

``` kotlin
binding.freeButton.setOnSingleClickListener {
    val testValue = listOf(0, 1, 2)

    Timber.d("clicked time: ${Calendar.getInstance()}")
}

```

---

## Confirm the Variable Address through Test and Debugging

Looking at the results through debugging,

- **First call result**

![https://blog.kakaocdn.net/dn/bXaXVa/btrVDjDgQFH/9aLTjLdttyNZsZhExkUn61/img.png](https://blog.kakaocdn.net/dn/bXaXVa/btrVDjDgQFH/9aLTjLdttyNZsZhExkUn61/img.png)

- **Second call result**

![https://blog.kakaocdn.net/dn/cwNzVA/btrVA1wnAHg/gLjYDuoKdtwKISNrIjLcE1/img.png](https://blog.kakaocdn.net/dn/cwNzVA/btrVA1wnAHg/gLjYDuoKdtwKISNrIjLcE1/img.png)

- **Third call result**

![https://blog.kakaocdn.net/dn/c6M6Zn/btrVBokysb3/QExZH7VCKMPP0DOPXdYdFk/img.png](https://blog.kakaocdn.net/dn/c6M6Zn/btrVBokysb3/QExZH7VCKMPP0DOPXdYdFk/img.png)

It can be seen that **every time you click the button, a new space is allocated to the memory**.

Therefore, if you need to define a variable every time you call it in the function,

you need to check whether you need to call it every time you click the button.