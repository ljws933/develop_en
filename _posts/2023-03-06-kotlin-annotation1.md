---  
layout: post  
title: "What is annotation in kotlin? custom annotation, built in annotation - 1"  
categories: kotlin
tags: kotlin
comments: true
showAds: true
description: Inline is the direct use of defined classes instead of calling a function or class in the body, which is usually used when creating a wrapping class. However, using inline classes is not meaningful when boxing processes occur frequently.
---  

### Understanding frequently used annotations in Kotlin
Annotations are means to add metadata (additional functionality) to code non-invasively.

---

## Types of annotations in Kotlin
1. Built-in annotations in Kotlin
2. Meta annotations for providing information about annotations
3. Custom annotations

These are the three types of annotations.

---

## Built-in Annotations in Kotlin

Kotlin already has many built-in annotations.

for example, 

> @Deprecated
> 

This annotation is used when you want to tell that a specific class, function, variable, etc. should not be used anymore.

```
@Deprecated("It is deprecated")
fun sum(a: Int, b: Int) = a + b

```

If you use the function that has @Deprecated, it will be expressed as follows.

![https://blog.kakaocdn.net/dn/b7ENgw/btr1dc0vCn0/9KLUBzwuHGkBOTSi1p678k/img.png](https://blog.kakaocdn.net/dn/b7ENgw/btr1dc0vCn0/9KLUBzwuHGkBOTSi1p678k/img.png)

---

> @JvmOverloads
> 

**@JvmOverloads** is a feature that automatically creates overloading functions as much as the default values set in the function or constructor parameters when using Kotlin Compiler.

For example, let's assume that one class has multiple constructors in Java:

``` kotlin
public class CustomView extends View {
// Instantiating Viewpublic CustomView(Context context) {
        super(context);
    }

// Inflate view using xmlpublic CustomView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

// Set View's default attributes with defStyleAttrpublic CustomView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

}

```

This can be expressed in Kotlin as follows:

``` kotlin
class CustomView : View {constructor(context: Context) : super(context)

    constructor(context: Context, attrs: AttributeSet) : super(context, attrs)

    constructor(context: Context, attrs: AttributeSet, defStyleAttr: Int) : super(context, attrs, defStyleAttr)

// ...
}

```

Using **@JvmOverloads**, it can be expressed more simply as follows:

``` kotlin
class CustomView @JvmOverloads constructor(context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0)
: View(context, attrs, defStyleAttr) {

// ...
}

```

This can be formalized as follows:

``` kotlin
class SubClassName @JvmOverloads constructor(parameter list) : SuperClassName(argument list)

```

---

> @Qualifier
> 

**@Qualifier** is an annotation that is mainly used in Hilt or Dagger2.

When providing dependencies using Hilt or Dagger2, it recognizes where dependencies go by looking at the value returned by the **@Provides** function.

However, if there are more than one functions that return objects of the same form within a class,

the compiler cannot know which function to provide.

At this time, **@Qualifier** is used.

For example, assuming that there is the following code:

``` kotlin
@Module
public class NameModule{
    @Provides
    // Function that returns String 1
    String provideName1(){
        return "Charles";
    }

    @Provides
    // Function that returns String 2
    String provideName2(){
        return "Runa";
    }
}

```

There are two functions that return strings within the same class.

If you call it as follows for dependency provision:

``` kotlin
class Main{
    @Inject
// Although it is named name1, the compiler does not know that this is the return value of provideName1().
    String name1;

    @Inject
    String name2;
    ...
}

```

An error occurs.

At this time, you can express it as follows using the **@Named** annotation, which performs the same function as **@Qualifier** and is provided by the javax.inject package by default:

``` kotlin
@Module
public class NameModule{
    @Provides
    @Named("me")
    String provideName1(){
        return "Charles";
    }

    @Provides
    @Named("you")
    String provideName2(){
        return "Runa";
    }
}

class Main{
    @Inject
    @Named("me")
    String name1;

    @Inject
    @Named("you")
    String name2;
    ...
}

```

### Customizing @Qualifier

You can customize @Qualifier as follows:

``` kotlin
public class WantQuailfierAutowired{

// Specify with the name myTarget@Qualifier("myTarget")
    private final Target taget;

// Call with the name myTargetpublic WantQuailfierAutowired(@Qualifier("myTarget") Target target){
        this.target = target;
    }
}

```

---

## Meta Annotations that Constrain Annotations

When defining an annotation, it is generally used as follows:

``` kotlin
// Simply put an annotation in front of the class name
annotation class MyAnnotation

```

However, you can add constraints to the annotation as follows:

This is called a Meta Annotation.

``` kotlin
@Target(AnnotationTarget.FUNCTION, AnnotationTarget.CLASS)
@Retention(AnnotationRetention.RUNTIME)
@Repeatable
@MustBeDocumented
annotation class MyAnnotation

```

The meaning of each Meta Annotation defined in the above code is as follows:

---

### **@Target**

It is used to specify where the annotation can be used.

The commonly used parameters are as follows.

- CLASS: Restrict to be used in class, interface, object, and annotation class.
- FUNCTION: Restrict to be used only in functions except constructors.
- FIELD: Restrict to be used only in fields, including backing fields.
- PROPERTY: Restrict the annotation to be used only in properties.
- TYPE: Restricts the annotation to be used anywhere.

In addition, as shown in the code above,

multiple Targets can be specified.

---

### **@Retention**

It is used to limit the scope of the annotation, and there are three parameters.

- SOURCE: It is useful only during compile time and is not included in the built binary. It is used when it is useful only during development, such as @suppress that suppresses warnings during development and is not needed in binary.
- BINARY: It is included in both compile time and binary, but it cannot be accessed through reflection.
- RUNTIME: It is included in both compile time and binary and can be accessed through reflection. If @Retention is not indicated in Custom Annotation, the default is RUNTIME.

---

### **@Repeatable**

Indicates whether an annotation can be used repeatedly for one element.

---

### **@MustBeDocumented**

Indicates whether the annotation can be included in the generated documentation.

It is mainly used when creating a library.

It can be used as follows:

``` kotlin
@MustBeDocumented
annotation class A_Annotation()

annotation class B_Annotation()

@A_Annotation
@B_Annotation
class TestClass()

```

Although there is **@MustBeDocumented** in the **A_Annotation()** class, there is no such annotation in the **B_Annotation()** class.

With Kotlin's official documentation tool with **dokka**, you can automatically generate official documentation.

Set the following in **build.gradle**:

``` kotlin
buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        classpath "org.jetbrains.dokka:dokka-gradle-plugin:0.9.14"
    }
}

apply plugin: 'org.jetbrains.dokka'

dokka {
    outputFormat = 'html'
    outputDirectory = "$buildDir/html"
}

```

Then, if you enter the following in the Terminal:

```
> ./gradlew dokka

```

The following document will be automatically generated:

![https://blog.kakaocdn.net/dn/Hu1HJ/btr2aIEHHGl/nusQoTnKu5HdfeTojIvpb1/img.png](https://blog.kakaocdn.net/dn/Hu1HJ/btr2aIEHHGl/nusQoTnKu5HdfeTojIvpb1/img.png)

If you look closely, you can see that only the @A_Annotation class with **@MustBeDocumented** has been documented.

---

## Basic Usage of Kotlin Annotations

### In Kotlin, annotations can be used with to apply multiple annotations.

You can also apply multiple annotations at once using [ ] instead of applying them one by one with @.

``` kotlin
// equivalent to @Synchronized, @Strictfp
@[Synchronized Strictfp]
fun test() {

}

```

### Defining Annotations in Kotlin

In Kotlin, you can define annotations by adding "annotation" in front of classes, interfaces, etc.

``` kotlin
// defining an annotation by adding "annotation"annotation class MyAnnotation(val text: String) {
// annotation parameter must always be defined with valcompanion object {

    }
}

// Calling the annotation using @
@MyAnnotation("testText")
fun myAnnotation() {

}

```

---

In part 2, we will discuss how to customize annotations and how to use them.

---

### Summary of Kotlin Annotations

- Annotations provide information to the compiler to check for syntax errors in the code.
- Annotations provide information to development tools or builders to automatically add code.
- Annotations provide information to execute specific functions at runtime.