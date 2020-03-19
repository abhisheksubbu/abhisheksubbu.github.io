---
title: Understanding Microservices Architecture Part 1
layout: blog
category: [Architecture]
excerpt: Development Community around the world is pushing the software development approach to shift from Monolithic to Microservices architecture. Most of us accept this shift without understanding the WHY, WHAT, WHEN and HOW of Microservices. In this blog, we will try to understand What is essential for an organization to shift from Monolithic to Microservices Architecture...
comments: true
---

Development Community around the world is pushing the software development approach to shift from Monolithic to Microservices architecture. Most of us accept this shift without understanding the WHY, WHAT, WHEN and HOW of Microservices. In this blog, we will try to understand

1. What is essential for an organization to shift from Monolithic to Microservices Architecture
2. The differences between Monolithic and Microservices Architecture

**Essential Ingredients to adopting Microservices** In order to successfully develop software in Microservices realm, you need to get these three things right.

1. Organization
2. Process
3. Architecture

**Organization** – There is a strong consensus in the development community that instead of having large development teams, small & autonomous teams (not more than 12, ideally 6 to 8) that can work independently is highly desired.

**Process** – There is also a strong consensus that you should be developing using Agile processes with Continuous Delivery/Deployment.

**Architecture** – On a very high level, you have to make either of these 2 decisions. Monolithic Architecture or Microservices Architecture.

## Monolithic v/s Microservices

Imagine an online store, its modules and their interactions. In a monolithic architecture, the functionalities like Catalog, Orders and Reviews would be layered and modularized, clearly separated. But when it comes to packaging, everything has to be put together as a single application that talks to the database.

![Monolithic](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/monolithic.png)

Well, there are reasons why organizations choose Monolithic Architecture.

- Its simple to Develop
- Its simple to Test
- Its simple to Deploy
- Its simple to Scale

So, what’s the downside of Monolithic Architecture?

- Every successful application (in monolithic form) keeps growing every time something is deployed.
- As it grows in years,
  - It becomes difficult to stick to Agile Processes (make changes to the application & evolve).
  - It also becomes difficult to deploy changes to the application using Continuous Delivery because a small change impacts a greater part of the application.
  - It becomes difficult to maintain independent, autonomous teams because every team is dependent on other team’s code or functionality.
  - Teams have to struggle with complex merging tasks in source control.
  - All these problems lead us to a situation that we call as the “Monolithic Hell”.

Well, the solution to solve all these problems is to adopt Microservices Architecture. Chris Richardson explains the meaning/definition of microservices in a simple & neat way.
![Functional Decomposition](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/functional-decomposition.png)

The X-AXIS (Horizontal Duplication) refers to the running multiple copies of your application behind a load balancer. The Z-AXIS refers to running copies of the same application on different servers and some routing logic controls which server responds to the requests. Both the X-AXIS and Z-AXIS approaches is how we can scale in Monolithic Architecture. The Y-AXIS seems much interesting as it refers to running different modules separately on servers. Each module on a server can be horizontally and vertically scaled separately. For us to put different modules on separate servers, we have to functionally decompose our application’s monolithic logic. This way of functional decomposition leads us to Microservices Architecture. Thus, the modules which we saw in a Monolithic Architecture becomes standalone, deployable & independently scalable services in Microservices Architecture. Each service will have its own database (which helps that service to be standalone). Sitting in front of these services is an API gateway (which acts like a Facade) that exposes the meaningful business operations to the clients (like browser, apps).

![Microservices](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/microservices.png)

## Benefits of Microservices Architecture

- The small team owns the responsibility of a specific service (like Catalog Service Team). This is how modern organizations wants their teams to be.
- This small team can independently make changes to their service [helps them follow Agile processes easily] and allows them to deploy these changes (using Continuous Integration or Continuous Delivery)
- Each service can have its own technology stack. So, organizations can adopt new technologies faster.

## Drawbacks of Microservices Architecture

- Complexity of developing a distributed system
  - Implementing inter-process communication [how a service talks to other service without direct calls]
  - Handling partial failures
- Complexity of implementing business transactions that span multiple databases for data consistency
- Testing a distributed system is more complex
- Complexity involved in deploying and operating a distributed system since there are multiple services running
- Managing the development and deployment of functionalities that involve multiple services

However, the good news is that solutions (like Docker, Kubernetes, Publisher-Subscriber) exists for removing these drawbacks. 🙂 We do understand that Monolithic Architecture and Microservices Architecture has their own benefits and drawbacks. But a key takeaway that we can understand is that the benefits of Microservices Architecture outweigh the drawbacks of large monolithic applications. In the next blog, we will deep dive into [Event Sourcing in Microservices Architecture](/understanding-microservices-architecture-part-2/).
