---
title: Understanding Microservices Architecture Part 2
layout: blog
category: [Architecture]
excerpt: In the previous blog of “Understanding Microservices Architecture – Part 1“, we did a detailed study about monolithic v/s microservices architecture, the benefits and drawbacks of each architecture and the solution approaches to it. I highly recommend you to read the “Understanding Microservices Architecture – Part 1” blog before diving into this blog as it...
comments: true
---

In the previous blog of “[Understanding Microservices Architecture – Part 1](/understanding-microservices-architecture/)“, we did a detailed study about monolithic v/s microservices architecture, the benefits and drawbacks of each architecture and the solution approaches to it. I highly recommend you to read the “[Understanding Microservices Architecture – Part 1](/understanding-microservices-architecture/)” blog before diving into this blog as it would help you connect to the context of what we are going to learn here. Well, we understood that Microservices Architecture is the way forward. Now, here comes a lots of questions in our mind.

- How to design & deploy these services?
- How should the services communicate?
- How do the client applications communicate with all these services?
- How do we correctly partition the monolithic application into services?
- How do we deal with distributed data management problems?

The solution here is to **adopt EVENT BASED approach** into microservices to solve the interaction issues. With respect to data management, we can either choose “**Shared Database**” approach or “**Database per service**” approach. With “Shared Database” approach, you have 1 single database with many tables in it. Different services might talk to one of more tables. This is pretty simple to achieve and ACID principles help maintain data consistency. But the biggest problem with this approach is that individual services have tight coupling with the database. This means that if we design a set of microservices, then every microservices would have to talk to this single database. If a team working on order service decides to update the database schema, then they have to go and talk to other service team and figure out if this schema update will break their service or not. Thus, things slow down and team start losing their autonomy/independent nature. As autonomy degrades, sticking to agile model becomes painful. The recommended approach in Microservices is for each service to have its own database. Therefore, each service gets it own private database which has all the information for that service to operate. For example, Order Service has an Order Database, Customer Service has a Customer Database and so on. This ensures **Loose Coupling**. But it’s more complex to get this working from a TRANSACTIONAL scope. Right? Now, we get confused on how a TRANSACTION is defined in Microservices Architecture because different services with their own database have to communicate and co-ordinate for completing a business transaction. The solution to this TRANSACTIONAL confusion is to **adopt EVENT DRIVEN ARCHITECTURE**. This means that when something significant happens in a service, that service has to raise an event. This event can be observed by other services and act upon that. With this, you can achieve eventual consistency in a transaction. For example, when a customer places an order, the following actions takes place.

1. Order Service creates an Order in PENDING state and emits an event “Order Created”
2. The Customer Service listens to the “Order Created” event and performs the credit check. If the credit check is successful, the customer service emits a “Credit Reserved” event. If the credit check failed, then it emits a “Credit Check Failed” event.
3. The Order Service listens to “Credit Reserved” and “Credit Check Failed” events and accordingly updates the state of the Order to COMPLETE or FAILED.

Here, we have 3 ACID transactions happening on each service and finally contributes to ACID of the entire order transaction becoming consistent. When we say “EMIT AN EVENT”, the question arises

- where is the event created?
- is the event persisted as a record?
- What if the order state got updated and event failed to fire?

The way to tackle this problem is by using “**EVENT SOURCING**” technique. In this technique, a separate table for Event is maintained where the Id, type of event, event data are all stored. Publisher services write event records to this table. Subscriber services read the event records from this table based on Event Type and carry out their actions. A benefit of Event Sourcing is that the entire history of a transaction from start till end can be traced from this Event Table.
![Event Sourcing](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/event-sourcing.png)

If there is a need to replay/retry from a particular step, you can do that easily because every step/event of the entire transaction was recorded with its data. Benefits of Event Sourcing

- It solves the data consistency issues in a Microservice/NoSQL based architecture.
- It provides reliable event publishing mechanism. You can mine this event data and even predict what events can occur in future and raise user notifications/alerts.
- It eliminates the Object Relational Mapping problem because the event is recorded with its own data.
- Because each state change is represented by an event, it helps us build a reliable audit log and create temporal queries to understand what the record state was back in time.

## Drawbacks of Event Sourcing

- It requires an application rewrite to fit that into microservices architecture.
- It makes breaking down of business logic into weird pieces and thus there is a learning curve to adapt to this unfamiliar style of programming.
- Once an event is published, these event records stay there forever (for historical needs). Sometimes some events might have been because of your bad design decisions.
- We have to handle duplicate events by detecting duplication.
- As the event store grows, querying the event table can be challenging.

That was a lot of learning. Isn’t it? In the Part 3, we will understand the querying complexities involved in Microservices and how we can solve this problem.
