---
title: Logger Framework in Salesforce
layout: blog
category: [Salesforce, Architecture]
excerpt: Logging errors in Salesforce environment is a practice that almost every organization and developer follows. In this blog, I will introduce a logging framework that will ensure that logging is done consistently.
comments: true
---

Right from the simplest to most complex, there are many approaches to logging. In this blog, I will introduce a framework for logging errors/warning in APEX language in Salesforce.

## Need for such a framework
- Propose a consistent way for developers to log errors/warnings in Apex.
- Reduce coupling by embracing OOPS concepts.
- Be able to deploy enhancements to the Logger without having the logger classes locked by runtime.
- Be able to switch to different logger implementations easily in future.

## Improvements that I aimed in this Framework
- My focus was to use the Dependency Inversion principle in Apex and ensure that I create a framework with highest levels of Cohesion and lowest levels of Coupling.

## UML Diagram
![UML Diagram](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/LoggerFramework.png)

## Explanation of UML Elements
- **ILogger** - This interface specifies the contract of LogMessage method that the framework exposes.
- **MockLogger** - This is the concrete implementation of ILogger and implements the LogMessage method. This concrete implementation is created only for framework testing purpose only. The actual users of the framework are expected to create their concrete implementation by implementing the ILogger interface.
- **ApexLogRequest** - This is a request class and it encapsulates all the data that needs to be provided for the framework for logging it.
- **LogLevel** - This is an enumeration that contains the log levels that the framework supports.
- **DIContainer** - This is the class that acts as the dependency inversion container. It's responsibility is to load the service registrations from a Custom Metadata and make it available for use.
- **LoggerInjector** - This class is responsible for getting the service registrations loaded by DIContainer and return the instance of the logger instance.
- **Client** - This element is only for demonstration purpose. It shows how a client should create the log request and use the Logger Framework to log that message.

#### DI_Container\_\_mdt

##### Field Structure

| Field Name                 | Type     |
| -------------------------- | -------- |
| Logger\_\_c                | Text(255) |

##### Values in DI_Container\_\_mdt custom metadata type

| DI Container Name      | Logger                |
| ---------------------- | --------------------- |
| ServicesRegistration   | MockLogger            |


The source code of this project is present in [my GitHub page](https://github.com/abhisheksubbu/ApexLoggerFramework){:target="\_blank"}.

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.