---  
layout: post  
title: "What is Solid Design in Android"  
categories: android
tags: android 
comments: true
showAds: true
description: Let's use DebugView to test the analogics of Firebase for debugging. This allows users to see when they used the app, which screen they saw, and which button they clicked on. It even tells you when you deleted the app.
---
## What is Solid Design?
* S: Single Accountability Principle (SRP) = Single Accountability Principle
* O : OPENING/CLOSING PRINCIPLES (OCP)
* L : Liskov Replacement Principle (LSP) = Liskov Replacement Principle
* I : Interface Segmentation Principle = Interface Separation Principle
* D : Dependency Inversion Principle (DIP) = Dependency Inversion Principle

SOLID design allows you to create code that's easy to follow and easy to change.

However, it is not necessary to apply the SOLID design when creating all apps, and you can use it appropriately for your project.

---
# S: Single Accountability Principle (SRP)

* It is a part of the SOLID design that corresponds to S.
* The module should handle only one thing.

There is a source file as below.

``` kotlin
class Order {
    
    fun sendOrderUpdateNotification() {
        // sends notification about order updates to the user.
    }

    fun generateInvoice() {
        // generates invoice
    }

    fun save() {
        // insert/update data in the db
    }
}
```

The Order Class above has multiple responsibilities.

A source file (module) called Oder contains functions that have multiple roles.

Therefore, it can be changed as follows.

``` kotlin
data class Order(
    val id: Long,
    val name: String,
    // ... other properties. 
)
class OrderNotificationSender {

    fun sendNotification(order: Order) {
        // send order notifications
    }
}
class OrderInvoiceGenerator {

    fun generateInvoice(order: Order) {
        // generate invoice
    }
}
class OrderRepository {

    fun save(order: Order) {
        // insert/update data in the db.
    }
}
```

When used, it can be used by delegating responsibility to one class as follows

<img src="https://user-images.githubusercontent.com/127634758/225625504-73e2f3fa-f960-4c89-9e6a-19b29e99b2f1.png" width="500" alt="">

``` kotlin
class OrderFacade(
    // 1 class parameter has 1 responsibility
    private val orderNotificationSender: OrderNotificationSender,
    private val orderInvoiceGenerator: OrderInvoiceGenerator,
    private val orderRepository: OrderRepository
) {
     fun sendNotification(order: Order) {
        // sends notification about order updates to the user.
        orderNotificationSender.sendNotification(order)
    }

    fun generateInvoice(order: Order) {
        // generates invoice
        orderInvoiceGenerator.generateInvoice(order)
    }

    fun save(order: Order) {
        // insert/update data in the db
        orderRepository.save(order)
    }
}
```

* However, giving a single responsibility to all classes in this way will cause class files to grow exponentially.
* That's why it's better to apply the SOLID design properly than to apply it completely…

---

# O - OCP(Open/Closed Principle)

* It is a part of the SOLID design that corresponds to O.
* The Open/Closed Principle definition is as follows.
* Software artifacts must be opened for expansion but closed for modification.
* That means the operation of the software artifact should be expandable without modification.

### Example of violating OCP

``` kotlin
enum class Notification {
    PUSH_NOTIFICATION, EMAIL
}
class NotificationService {

    fun sendNotification(notification: Notification) {
        // Perform other tasks according to the value of the notification.
        when (notification) {
            Notification.PUSH_NOTIFICATION -> {
                // send push notification
            }

            Notification.EMAIL -> {
                // send email notification
            }
        }
    }
}

```

It doesn't seem to be a problem to execute, nor does it seem to violate OCP.

However, what happens if I add a new Notification here?

Assuming you need to add NotificationServiceSMS, it will be as follows


``` kotlin
enum class Notification {
    PUSH_NOTIFICATION, EMAIL, SMS
}
class NotificationService {

    fun sendNotification(notification: Notification) {
        when (notification) {
            Notification.PUSH_NOTIFICATION -> {
                // send push notification
            }

            Notification.EMAIL -> {
                // send email notification
            }

            Notification.SMS -> {
                // send sms notification
            }
        }
    }
}
```

Maybe if you noticed it here,

You must modify Notification Service each time you add a new value.

Therefore, it can be seen as a violation of OCP.

This can be resolved in the following ways:


``` kotlin
// Create an interface, not a class.
interface Notification {
    fun sendNotification()
}

```

And create a class file for each Notification.


``` kotlin
// Inherit the Notification interface.
class PushNotification : Notification {
    // Implement each function in a redefined form.
    override fun sendNotification() {
        // send push notification
    }
}

class EmailNotification : Notification {
    override fun sendNotification() {
        // send email notification
    }
}

```

Create a class file called NotificationService to integrate it.

``` kotlin
class NotificationService {

    fun sendNotification(notification: Notification) {
        notification.sendNotification()
    }
}
```

This will allow you to add new features by simply adding the file as follows without modifying the relevant Notification file.


``` kotlin
class SMSNotification : Notification {
    override fun sendNotification() {
        // send sms notification
    }
}

```

By using the interface in this way, you can use common parts and also use them redefine as needed.

OCP is also very difficult to keep 100%, so it is important to always be conscious and use it properly when writing code.

---

# L - Liskov Substitution Principle(LSP)

It is a part of the SOLID design that corresponds to L.
To be defined as follows.
If the datatype S is a subtype of datatype T, it must be possible to replace the object of datatype T with the object of datatype S without changing the properties of the necessary program.

To put it more simply,

When used in the same way, the child object should be able to use the ability of the parent object as it is.

(I will explain this in detail in another post.)

Moving on to the example, we're going to create the following code.

### a service for collecting and sorting garbage

``` kotlin
interface Waste {
    fun process()
}
Waste interface를 상속받는 클래스 구현

class OrganicWaste : Waste {
    override fun process() {
        println("Processing Organic Waste")
    }
}
class PlasticWaste : Waste {
    override fun process() {
        println("Processing Plastic Waste")
    }
}
```

Implement a class using these two

``` kotlin
class WasteManagementService {

    fun processWaste(waste: Waste) {
        waste.process()
    }
}

```

Defining a function that uses it

In the same way, processWaste of waste management service is called, 

but it satisfies the Liskoff substitution principle because it is doing the same process without affecting each other.

``` kotlin
fun main() {
    val wasteManagementService = WasteManagementService()

    var waste: Waste

    waste = OrganicWaste()
    wasteManagementService.processWaste(waste) // Output: Processing Organic Waste

    waste = PlasticWaste()
    wasteManagementService.processWaste(waste) // Output: Processing Plastic Waste
}

```

---

# I - Interface Isolation Principle (ISP)

It is a part of the SOLID design that corresponds to I.
The definition is as follows.
It should not be forced to implement unnecessary methods in a class that inherits unused interfaces.

### a bad example

``` kotlin
interface OnClickListener {
    fun onClick()
    fun onLongClick()
}

```

If the interface is defined in this way, both onClick() and onLongClick() should always be defined as follows

``` kotlin
class CustomUIComponent : OnClickListener {
    override fun onClick() {
        // handles onClick event.
    }

    // left empty as I don't want the [CustomUIComponent] to have long-click behavior.
    override fun onLongClick() {

    }
}

```

To solve this problem, we can just divide the interface.

``` kotlin
interface OnClickListener {
    fun onClick()
}
interface OnLongClickListener {
    fun onLongClick()
}

```

And only the necessary interfaces are inherited and used.

``` kotlin
class CustomUIComponent : OnClickListener {
    override fun onClick() {
        // handle single-click event
    }
}

```

---

# D - Dependency Inversion Principle (DSP)

It is a part of the SOLID design that corresponds to D.
The definition is as follows.
* The project has independent modules.
* Submodules refer to (dependent) parent modules and
* A parent module cannot be referenced (dependent) on a child modules
* This is called the "Dependency Inversion Principle".
* Simply put, "We rely only on elements that don't change easily, but the parent module can't rely on the child module."Ida.

### Example code

``` kotlin
class ClassA {
    fun doSomething() {
        println("Doing something")
    }
}

class ClassB {
    fun doIt() {
        // ClassB 는 ClassA가 반드시 있어야만 사용할 수 있다 = 즉, ClassA에 대해 의존성을 가진다
        val classA = ClassA() 
        classA.doSomething()
    }
}
```

Assume that you have a program that is dependent on


``` kotlin
// Notification to send an email
class EmailNotification {
    fun sendNotification(message: String) {
        println("Sending email notification with message \"$message\"")
    }
}

// The part where Notification is sent.
class NotificationService {

    fun sendNotification(message: String) {
        // // Must rely on Email Notification in this part
        val emailNotification = EmailNotification()
        emailNotification.sendNotification(message)
    }
}

// Execution function
fun main() {
    val notificationService = NotificationService()
    
    // Output: Sending email notification with message "Happy Coding"
    notificationService.sendNotification("Happy Coding") 
}
```

So far, there is no problem with dependencies, but there is a problem that other types of Notifications cannot be sent.
You can remove dependencies in the following ways and have Notification Service send email as well as other types of Notification:

### Entire code

``` kotlin

interface Notification {
    fun sendNotification(message: String)
}
class EmailNotification : Notification {
    override fun sendNotification(message: String) {
        println("Sending email notification with message \"$message\"")
    }
}
class SmsNotification : Notification {
    override fun sendNotification(message: String) {
        println("Sending sms notification with message \"$message\"")
    }
}
fun main() {
    val message = "Happy Coding"
    val notificationService = NotificationService()
    var notification: Notification

    notification = EmailNotification()
    notificationService.notification = notification
    notificationService.sendNotification(message)
    // Output: Sending email notification with message "Happy Coding"

    notification = SmsNotification()
    notificationService.notification = notification
    notificationService.sendNotification(message)
    // Output: Sending sms notification with message "Happy Coding"
}

```

This independent dependence and OCP can be satisfied at the same time.

### Define a line in a SOLID design

* SRP: Each software module should have only one role.
* OCP: Software systems should be easy to change. Instead of changing the existing code, it should be designed to change the system behavior by adding new code.
* LSP: To build a software system with interchangeable parts, you must follow a contract to exchange those parts with each other.
* ISP: Software designers should avoid relying on unused items.
* DIP: Code implementing higher-level policies should not rely on lower-level details.