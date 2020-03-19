---
title: Customizing Registration Handler Apex Code in Salesforce Open ID Connect Auth Provider
layout: blog
category: [Salesforce]
excerpt: When you play with Salesforce Single Sign-On, it is most probable that you will come across something called the “Registration Handler” class. In this blog, we will take a deep dive into understanding what a RegistrationHandler class is, how to create it, how to customize it and finally use it for production.
comments: true
---

When you play with Salesforce Single Sign-On, it is most probable that you will come across something called the “Registration Handler” class. In this blog, we will take a deep dive into understanding what a RegistrationHandler class is, how to create it, how to customize it and finally use it for production.

## What is a Registration Handler?

A **Registration Handler** is nothing but a normal apex class that implements **Auth.RegistrationHandler interface**.

## Why is Registration Handler needed?

It is needed for **running the authorization code** after authentication happens. For example: When you do single sign-on with Google, the authentication is done by Google Identity server and the results of the authentication (which is user data, tokens, etc) are passed back into the Relying Party (which is your application) to do further actions of registering/logging in the user. The code at the Relying Party (which is your application) that receives the user data/tokens from Identity Servers is called Registration Handler. Using the Registration Handler apex code, you can check if the user is really authorized to use the application and you either register the user or log him in.

## When is a Registration Handler used?

When you are designing a single sign-on feature (using Auth Providers) in Salesforce, a Registration Handler apex class is used.

## What is the need of implementing Auth.RegistrationHandler interface in the Registration Handler apex class?

Any apex class that implements the Auth.RegistrationHandler interface makes the Force.com platform aware that this apex class is a Registration Handler and not just a simple functional apex class. The Force.com platform knows when information is received from authentication providers, it needs to use specific methods in this Registration Handler class for creating & updating user data as appropriate.

## What are the specific methods in the Registration Handler that must be defined?

1. **global User createUser(ID portalId, Auth.UserData userData)** – The Force.com platform routes the control flow into this method of your Registration Handler class when it gets the user information from the IDP (Identity Provider). This new user data would be most probably not there in the User table or sometimes it may also represent an existing user record in the database. It is very important to check for user existence in this method and do necessary actions accordingly. The use of this method is usually referred to as “User Creation/Linking” flow.
2. **global void updateUser(ID userId, ID portalId, Authe.UserData userData)** – The Force.com platform routes the control flow into this method of your Registration Handler class when it gets an already registered user information from the IDP (Identity provider). You can use this method to update user record in Salesforce with latest information obtained from the IDP. The use of this method is usually referred to as the “**Existing User Linking**” flow.

## What are the possible information that I can get from the IDP?

The Auth.UserData object can provide you with the following information.

1. **ID** – An identifier from the third party for the authenticated user, such as the Facebook user number or the Salesforce user ID
2. **First Name** – The first name of the authenticated user, according to the third party.
3. **Last Name** – The last name of the authenticated user, according to the third party.
4. **Full Name** – The full name of the authenticated user, according to the third party.
5. **Email** – The email address of the authenticated user, according to the third party.
6. **Link** – A stable link for the authenticated user such as https://www.facebook.com/MyUsername.
7. **Username** – The username of the authenticated user in the third party.
8. **Locale** – The standard locale string for the authenticated user.
9. **Site Login URL** – The site login page URL passed in if used with a site; null otherwise.
10. **Attribute Map** – A map of data from the third party, in case the handler has to access non-standard values. For example, when using Janrain as a provider, the fields Janrain returns in its accessCredentials dictionary are placed into the These fields vary by provider.

## What you should not do with user data received from IDP?

The Auth.UserData argument that you receive in the createUser/updateUser method should not be stored anywhere in your Salesforce account. It violates the Data Privacy rules and standards (as outlined by GDPR [General Data Protection Regulation]). This data should be used in real-time in your Registration Handler code for creating a user record or updating a user record only. A working implementation of a Registration Handler can be found in the Social Sign-On blog for you to try out and understand the flow.
