---  
layout: post  
title: "What is annotation in kotlin? custom annotation, built in annotation - 2"  
categories: kotlin
tags: kotlin
comments: true
showAds: true
description: There are two ways to customize annotations in Kotlin using reflection and using code generation. Using reflection allows for custom annotations with reflection.
---  

### **Summary of frequently used annotations in Kotlin-2**
In this post, we will learn how to customize annotations following the previous post.

{% linkpreview "https://mmol93-developer.com/kotlin/2023/03/06/kotlin-annotation1/" %}

---

## Custom Annotation using Reflection
There are two ways to customize annotations in Kotlin:

1. **Using Reflection**
2. **Using Code Generation**

First, let's briefly explain what Reflection is.

> Reflection is a technique that analyzes the structure of a program (objects, functions, properties) at runtime. In other words, when the program is running, you can determine the internal structure of an object through instances, etc.
> 
> 
> However, since it is necessary to check it every time, it can cause performance degradation of the app.
> 

To use Reflection in Kotlin, you need to use KClass.

You can find information about **Kotlin Reflection** here.

{% linkpreview "https://mmol93-developer.com/kotlin/2023/03/09/kotlin-reflection/" %}

To use KClass in Kotlin, you need to add the following library:

``` kotlin
implementation "org.jetbrains.kotlin:kotlin-reflect:$kotlin_version"

```

Then, let's create an annotation class by specifying the annotation as follows:

``` kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class StringConstraint(
    val minLength: Int,
    val maxLength: Int
)

```

The **StringContraint** annotation class created above is an annotation class created to define the minimum and maximum length of a **String variable**.

Apply this custom annotation to the **data class** with property values.

``` kotlin
data class Data(
    // set field value with custom annotation@StringConstraint(10, 50)
    val title: String,
    @StringConstraint(100, 500)
    val contents: String
)

```

Now, write the code to check the validation using these two.

``` kotlin
object FieldValidator {
    data class ValidationResult(
        var isValid: Boolean = true,
        val invalidFieldNames: MutableList<String> = mutableListOf()
    )

    fun validate(data: Data): ValidationResult {
        val result = ValidationResult()

            // get property information with reflectiondata::class.declaredMemberProperties.forEach {
            val field = it
            val annotation = it.findAnnotation<StringConstraint>()

            if (annotation != null && field.getter.visibility == KVisibility.PUBLIC) {
                val fieldValue = field.getter.call(data) as String
                // set validation with annotation field value
                if (fieldValue.length !in annotation.minLength..annotation.maxLength) {
                    result.isValid = false
                    result.invalidFieldNames.add(field.name)
                }
            }
        }

        return result
    }
}

```

Then, write the following code and run it.

``` kotlin
fun main() {
    val data = Data(
        "transparency to the extreme",
        "Transparency means sharing more than just results. It means sharing enough to know the intent, context, and process. We believe that the extreme honesty we seek can only be achieved when transparency is a prerequisite."
    )

    val validationResult = FieldValidator.validate(data)

    // Validation: false  /  not validated Field: [title]
    println("Validation: ${validationResult.isValid}  /  not validated Field: ${validationResult.invalidFieldNames}")
}

```

---

## Summary

We have learned how to use custom annotations and Kotlin Reflection.

By the way, all the codes are as follows.

``` kotlin
import kotlin.reflect.KVisibility
import kotlin.reflect.full.declaredMemberProperties
import kotlin.reflect.full.findAnnotation

@Target(AnnotationTarget.PROPERTY)
annotation class StringConstraint(
    val minLength: Int,
    val maxLength: Int
)

data class Data(
// set field value with custom annotation@StringConstraint(10, 50)
    val title: String,
    @StringConstraint(100, 500)
    val contents: String
)

object FieldValidator {
    data class ValidationResult(
        var isValid: Boolean = true,
        val invalidFieldNames: MutableList<String> = mutableListOf()
    )

    fun validate(data: Data): ValidationResult {
        val result = ValidationResult()

            // get property information with reflectiondata::class.declaredMemberProperties.forEach {
            val field = it
            val annotation = it.findAnnotation<StringConstraint>()

            if (annotation != null && field.getter.visibility == KVisibility.PUBLIC) {
                val fieldValue = field.getter.call(data) as String
                // set validation with annotation field value
                if (fieldValue.length !in annotation.minLength..annotation.maxLength) {
                    result.isValid = false
                    result.invalidFieldNames.add(field.name)
                }
            }
        }

        return result
    }
}

fun main() {
    val data = Data(
        "transparency to the extreme",
        "Transparency means sharing more than just results. It means sharing enough to know the intent, context, and process. We believe that the extreme honesty we seek can only be achieved when transparency is a prerequisite."
    )

    val validationResult = FieldValidator.validate(data)

    // Validation: false  /  not validated Field: [title]
    println("Validation: ${validationResult.isValid}  /  not validated Field: ${validationResult.invalidFieldNames}")
}

```

---

In the next post, we will learn about the second way to customize annotations, which is:

> Insert logic into annotations using Code Generation at Compile Time
>