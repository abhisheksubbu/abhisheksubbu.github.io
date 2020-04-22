---
title: Inversion of Control - Part 1
layout: blog
category: [Architecture, Deep Dive]
excerpt: In this blog post, we will understand the basic principles of Inversion of Control and Dependency Injection. Also, we will see how it is very important for developers to adopt IOC.
comments: true
---

> **Inversion of Control** is a **principle** in software engineering by which the **control of objects** or portions of a program is **transferred** to a **container or framework**.

Let's disect the defintion word by word.

- **Principle** is a law/rule, if followed, will yield good results. For example, our parents often tell us that "Do good to others" and always stick to the principle. What happens if we follow a principle? It makes us a good human being, society benefits, etc. It is absolutely not necessary to follow a principle, if you don't want good results. Also, a principle never tells you how to achieve it. You can achieve this however you want. This means that you have the freedom to choose the ways to adhere to the principle.

- **Principle** in software engineering - this principle only applies in the world of software engineering (that is, every programming language which is used to create softwares has the option to embrace this principle).

- **What does the principle state?** - it says anything related to **controlling of objects** should be handled by a **container**/**framework**

- What does **controlling of objects** mean? - creating objects by writing `new Class()` gives life to the objects and is the point where you start controlling it.

- Why should **controlling of objects** be transferred to a **container** and not have it managed in the main program? - If you manage the creation of objects without a container, then you are coupling the responsibility of object creation in various parts of the program and it becomes a maintainance headache.

- So, what is this **container** or how does it look like? - Well, the principle doesn't state how the container should look like. It just says that the creation of objects should be abstracted to a black box (or in other words a container/framework). It gives you the full freedom to create your own container or use a third-party container/framework.

In a nutshell, the principle of Inversion of Control (IOC) says:

> Don't call me, We will call you

It means that don't call the classes directly for creating objects. Instead, the container/framework will create/manage the objects and will give it to the part of the program that needs that object. In other words, **don't call concretions, instead, rely on the abstractions**.

This would remind us of the "**D**" in **SOLID** principles.

For those who are hearing SOLID for the first time, we will briefly introduce it so that you don't lose the flow of thoughts. Every letter in the SOLID is itself a principle.

- **S**ingle Responsibility Principle - One class should have only one responsibility. For example, a Tax class should only calculate the tax and return the tax rate. It is not the responsibility of the Tax class to set any properties of the cart items.

- **O**pen Closed Principle - Open for Extension & Closed for Modification. It means that any piece of code that was released should not be modified (from a signature perspective) because it can break any integrations of use by a client. So, always extend the methods and add features. Do not modify existing things.

- **L**iskov Substitution Principle - It states that sub-classes should be substitutable for parent classes. To make this more clear, if the main program uses a parent class reference, then we should be able to create a sub-class that inherits the parent class and change the parent class reference by sub-class in the main program. Even after this, the program should not break.

- **I**nterface Segregation Principle - It states that you should break the interfaces into small, manageable ones instead of having a single huge interface. This allows the clients to use those interfaces that they are interested in. For example, if you break a single ICommerce interface into ITax interface and ICart interface. Then, it will help the clients to consume ITax interface if they are only interested to use the Tax logic that we expose. So, you are not forcing the client to use ICommerce interface that contains methods of tax calculations and cart methods.

- **D**ependency Inversion Principle - It states that depend only on abstractions and do not depend on concretions.

In Part 2, we will understand the "Ways of achieving IOC".
