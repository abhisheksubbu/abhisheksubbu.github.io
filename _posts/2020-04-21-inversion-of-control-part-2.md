---
title: Inversion of Control - Part 2
layout: blog
category: [Architecture, Deep Dive]
excerpt: In this blog post, we will explore the practical ways of achieving Inversion of Control using Dependency Injection patterns. We'll use simple JavaScript examples to understand how to implement IOC in real-world scenarios.
comments: true
---

In [Part 1](/2020/04/20/inversion-of-control-part-1), we understood what Inversion of Control (IOC) is and why it's important. We learned that IOC is a principle that says: **"Don't call me, We will call you"** - meaning, don't create objects directly, let a container or framework handle it.

Now, let's explore the **practical ways** to achieve IOC. The most common and recommended approach is **Dependency Injection**.

## What is Dependency Injection?

**Dependency Injection (DI)** is a design pattern that implements the IOC principle. It's a technique where one object supplies the dependencies of another object, rather than the object creating them itself.

Think of it like this: Instead of a chef going to the market to buy ingredients (creating dependencies), someone else brings the ingredients to the chef (injecting dependencies). The chef just focuses on cooking!

## The Problem Without Dependency Injection

Let's start with a simple example to understand the problem:

```javascript
// A class that sends emails
class EmailService {
    sendEmail(to, message) {
        console.log(`Sending email to ${to}: ${message}`);
    }
}

// A class that needs to send emails
class UserService {
    constructor() {
        // Creating the dependency directly - BAD!
        this.emailService = new EmailService();
    }

    registerUser(name, email) {
        // Some user registration logic
        console.log(`Registering user: ${name}`);
        
        // Using the email service
        this.emailService.sendEmail(email, "Welcome to our platform!");
    }
}

// Usage
const userService = new UserService();
userService.registerUser("John", "john@example.com");
```

**What's wrong here?**
- `UserService` is tightly coupled to `EmailService`
- If we want to test `UserService`, we can't easily replace `EmailService` with a mock
- If we want to change `EmailService` to `SmsService`, we have to modify `UserService`
- This violates the Dependency Inversion Principle (the "D" in SOLID)

## Solution: Dependency Injection

There are three main ways to inject dependencies:

### 1. Constructor Injection (Most Common)

This is the most popular and recommended way. Dependencies are provided through the constructor.

```javascript
// Email service
class EmailService {
    sendEmail(to, message) {
        console.log(`Sending email to ${to}: ${message}`);
    }
}

// SMS service (alternative implementation)
class SmsService {
    sendSms(to, message) {
        console.log(`Sending SMS to ${to}: ${message}`);
    }
}

// User service that accepts dependencies via constructor
class UserService {
    constructor(notificationService) {
        // Dependency is injected, not created!
        this.notificationService = notificationService;
    }

    registerUser(name, contact) {
        console.log(`Registering user: ${name}`);
        
        // Use the injected service
        if (this.notificationService.sendEmail) {
            this.notificationService.sendEmail(contact, "Welcome!");
        } else if (this.notificationService.sendSms) {
            this.notificationService.sendSms(contact, "Welcome!");
        }
    }
}

// Usage - We control what gets injected
const emailService = new EmailService();
const userService = new UserService(emailService);
userService.registerUser("John", "john@example.com");

// Easy to switch to SMS service
const smsService = new SmsService();
const userService2 = new UserService(smsService);
userService2.registerUser("Jane", "1234567890");
```

**Benefits:**
- Dependencies are explicit and clear
- Easy to test (you can inject mock objects)
- Easy to swap implementations
- Dependencies are required (can't forget to provide them)

### 2. Setter Injection

Dependencies are provided through setter methods or properties.

```javascript
class UserService {
    constructor() {
        this.notificationService = null;
    }

    // Setter method to inject dependency
    setNotificationService(service) {
        this.notificationService = service;
    }

    registerUser(name, contact) {
        console.log(`Registering user: ${name}`);
        
        if (this.notificationService) {
            if (this.notificationService.sendEmail) {
                this.notificationService.sendEmail(contact, "Welcome!");
            }
        }
    }
}

// Usage
const userService = new UserService();
const emailService = new EmailService();
userService.setNotificationService(emailService);
userService.registerUser("John", "john@example.com");
```

**When to use:**
- When dependencies are optional
- When you need to change dependencies at runtime
- Less common than constructor injection

### 3. Interface Injection (Using Abstractions)

This is about depending on abstractions (interfaces) rather than concrete classes. In JavaScript, we can achieve this using duck typing or by defining clear contracts.

```javascript
// Define what a notification service should look like (contract)
// In JavaScript, we don't have interfaces, but we can define the expected shape

// Email service implementing the contract
class EmailService {
    sendEmail(to, message) {
        console.log(`Email to ${to}: ${message}`);
    }
    
    // This method makes it compatible with the contract
    notify(contact, message) {
        this.sendEmail(contact, message);
    }
}

// SMS service implementing the same contract
class SmsService {
    sendSms(to, message) {
        console.log(`SMS to ${to}: ${message}`);
    }
    
    // Same method name - same contract!
    notify(contact, message) {
        this.sendSms(contact, message);
    }
}

// User service depends on the abstraction (any service with 'notify' method)
class UserService {
    constructor(notificationService) {
        // We don't care if it's EmailService or SmsService
        // We only care that it has a 'notify' method
        this.notificationService = notificationService;
    }

    registerUser(name, contact) {
        console.log(`Registering user: ${name}`);
        // Use the abstraction - we don't know or care about the implementation
        this.notificationService.notify(contact, "Welcome!");
    }
}

// Usage - both work the same way!
const emailService = new EmailService();
const userService1 = new UserService(emailService);
userService1.registerUser("John", "john@example.com");

const smsService = new SmsService();
const userService2 = new UserService(smsService);
userService2.registerUser("Jane", "1234567890");
```

**Benefits:**
- Complete decoupling from concrete implementations
- Easy to add new implementations (just follow the contract)
- Follows the Dependency Inversion Principle perfectly

## Real-World Example: E-commerce Application

Let's see a more practical example:

```javascript
// Payment abstraction (contract)
class PaymentProcessor {
    processPayment(amount) {
        throw new Error("This method must be implemented");
    }
}

// Credit card implementation
class CreditCardProcessor extends PaymentProcessor {
    processPayment(amount) {
        console.log(`Processing $${amount} via Credit Card`);
        return { success: true, transactionId: "CC123" };
    }
}

// PayPal implementation
class PayPalProcessor extends PaymentProcessor {
    processPayment(amount) {
        console.log(`Processing $${amount} via PayPal`);
        return { success: true, transactionId: "PP456" };
    }
}

// Order service - depends on PaymentProcessor abstraction
class OrderService {
    constructor(paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }

    createOrder(items, totalAmount) {
        console.log(`Creating order with ${items.length} items`);
        
        // Process payment using injected processor
        const result = this.paymentProcessor.processPayment(totalAmount);
        
        if (result.success) {
            console.log(`Order created! Transaction ID: ${result.transactionId}`);
            return { orderId: "ORD789", transactionId: result.transactionId };
        }
        
        throw new Error("Payment failed");
    }
}

// Usage - easy to switch payment methods
const creditCardProcessor = new CreditCardProcessor();
const orderService1 = new OrderService(creditCardProcessor);
orderService1.createOrder(["item1", "item2"], 100);

// Switch to PayPal - no changes to OrderService needed!
const paypalProcessor = new PayPalProcessor();
const orderService2 = new OrderService(paypalProcessor);
orderService2.createOrder(["item3"], 50);
```

## Testing with Dependency Injection

One of the biggest benefits of DI is easy testing:

```javascript
// Mock payment processor for testing
class MockPaymentProcessor extends PaymentProcessor {
    processPayment(amount) {
        // Simulate payment without actually charging
        console.log(`[TEST] Mock payment for $${amount}`);
        return { success: true, transactionId: "TEST123" };
    }
}

// Test the OrderService without real payment processing
const mockProcessor = new MockPaymentProcessor();
const testOrderService = new OrderService(mockProcessor);
testOrderService.createOrder(["test-item"], 25);
// No real money is charged during testing!
```

## Dependency Injection Container (Advanced)

For larger applications, you might use a DI Container that manages all dependencies automatically:

```javascript
// Simple DI Container
class DIContainer {
    constructor() {
        this.services = {};
    }

    // Register a service
    register(name, factory) {
        this.services[name] = factory;
    }

    // Get a service (creates it if needed)
    get(name) {
        if (!this.services[name]) {
            throw new Error(`Service ${name} not found`);
        }
        
        const factory = this.services[name];
        return factory(this); // Pass container for nested dependencies
    }
}

// Usage
const container = new DIContainer();

// Register services
container.register('emailService', () => new EmailService());
container.register('userService', (container) => {
    const emailService = container.get('emailService');
    return new UserService(emailService);
});

// Get services - dependencies are automatically resolved
const userService = container.get('userService');
userService.registerUser("John", "john@example.com");
```

## Key Takeaways

1. **Constructor Injection** is the most common and recommended approach
2. **Dependency Injection** makes your code:
   - More testable
   - More flexible
   - Less coupled
   - Easier to maintain

3. **Always depend on abstractions**, not concrete implementations
4. **Let someone else** (container, framework, or caller) create and inject dependencies

## Summary

Inversion of Control is achieved primarily through **Dependency Injection**, where:
- Objects receive their dependencies from outside (injected)
- Objects don't create their own dependencies
- This makes code more flexible, testable, and maintainable

Remember: **"Don't call me, We will call you"** - let the dependencies come to you, don't go and get them yourself!

In Part 3, we'll explore more advanced topics like DI containers, service locators, and how popular frameworks implement IOC.

