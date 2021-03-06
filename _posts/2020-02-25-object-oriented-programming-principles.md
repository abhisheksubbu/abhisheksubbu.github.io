---
title: Object Oriented Programming Principles
layout: blog
category: [Fundamentals]
excerpt: In this post, we will learn the history, advent and features/principles of Object Oriented Programming Principles (OOPS). As an aspiring developer/senior developer/architect, you need to very well understand these principles and apply it to real world problems.
comments: true
---

## History

When programming languages were introduced into the industry, they were usually Procedural in nature. Procedural means line by line execution from top to bottom.

For example, the procedure for making a phone call is:

1. Pick up the phone
2. Unlock the phone
3. Open the dialer app
4. Type the phone number
5. Hit the call button

In the above procedure, the steps/logic needs to be run from top to bottom (line by line) manner. You cannot make a call if you skip any steps or jump between steps randomly. This is the nature of procedural programming. To point out, programming languages like “C” are procedural in nature.

Speaking about Procedural approach, it doesn’t mean that Procedural Programming Languages are bad in any way. They are useful in terms of device programming (like the 8085/8086 Microprocessor assembly language we used to code back in college days). Just that for real world scenarios, procedural approach seems to introduce a lot of complexity.

### So what is the problem with Procedural/Structural Programming Languages?

1. **Data types & Variables**
   Every data needs to be in its own variables and you as a programmer should know what variables are created for what purpose and use it. So, a person’s first name, last name, date of birth and many other data needed to be in its own variables. There was no way of wrapping up a set of variables as the variables of a Person entity. “C” program introduced the concepts of Structs and Unions which solved the problem of wrapping up associated variables in a unit. This could only be visualized as a user defined data type. But it failed with respect to data access. Even though structs and unions were there, the data was very much accessible all over.
2. **Data Access**
   Procedural programs didn’t have the concept of modules. Everything was methods/functions. Therefore restricting the access to data in a unit was a painful task.
3. **Responsibilities were all scattered**
   Because everything is procedural, there was no efficient way to separate the responsibilities of a system/program in a modularized way. The only way to separate was to have separate functions. Having separate functions didn’t necessarily separate the responsibilities in a true sense.

### Object Oriented perspective had to be brought

Representing a real world problem in a concise, modularized way was the need of the hour and thus Object Orientation was born. “C++” programming language brought in the first glimpse of Object Oriented Concepts and it was a huge success in the programming community.

The community started getting into C++ really deep and new object oriented programming languages like Java and .NET started to emerge .

### Principles/Features of Object Orientation

1. Class
2. Object
3. Encapsulation
4. Abstraction
5. Inheritance
6. Polymorphism

#### CLASS

- Class is a user defined prototype/blueprint of a real world entity
- Class constitutes of properties & methods
- They wrap up a single responsibility in to a unit/class
- Example:
  - Computer can be a class that contains lots of components as its properties and can do specific methods like running an installed program, etc.

#### OBJECT

- Object is the instance of a Class
- It keeps the values of properties & references to methods inside the object
- Object can be managed in memory
- Objects, by nature, do not share any data to another object.
- Example:
  - An HP All-in-One PC is an Object of the Computer Class.

#### ENCAPSULATION

- Wrapping up the data into a single unit is called as Encapsulation
- Class is the encapsulated form
- The encapsulated form (class) can expose/not expose data for outside access using access modifiers.
- Example:
  - The processor, motherboard, hard disk, CD drive, ethernet adapters are all encapsulated into the All-in-One PC. It only exposes what we as users need to use.

#### ABSTRACTION

- Hiding the complexities and only exposing the necessary data/methods for outside access is called Abstraction
- It is achieved by declaring private properties and exposing data with public methods.
- Example:
  - Even though there are millions of Integrated Chips on the Network Interface Controller Board, it exposes a single NIC port to which users can plug in their ethernet cable to access internet.

#### INHERITANCE

- It is the mechanism by which classes can re-use its parent class implementations
- It helps to model real world use-cases like Man is a Human. A Man gets all the characteristics of a Human.
- Inheritance helps in re-usability of code
- Example:
  - A PC is a Computer. A Laptop is a Computer. Both the PC and Laptop inherit the properties of a Computer.

#### POLYMORPHISM

- Poly means Many. Morph means Forms. Polymorphism brings in the ability for an object to behave differently in different situations.
- Example:
  - Certain keys in the keyboard can be configured to do different actions as per the shortcut settings set in the specific application. The “Up Arrow” key can help you scroll a word application to go upwards. The same “Up Arrow” key can also accelerate a bike in the “Road Rash” game installed in that machine.

### Exercise for the Brain

- Try to identify the Object Oriented features in a
  - Bus Station
  - Web Page
  - Workplace
  - HR Department
