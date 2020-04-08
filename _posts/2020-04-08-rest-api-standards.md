---
title: REST API Standards
layout: blog
category: [Best Practices]
excerpt: In this blog post, I will explain what REST standards are. Additionally, we will list out what API's need to have for it to be RESTful.
comments: false
---

**REST** stands for Representational State Transfer.

#### Features

- **It is stateless by nature** - It means that every request-response handshake is not aware of what happened in the previous request-response cycle.
- **Works with HTTP protocol** - It means that REST API's interact with HTTP requests with all HTTP verbs and generate an HTTP response.

#### HTTP Verbs

1. **GET** - A request with GET verb does not modify any resources on the server. It simply fetches/queries the resource from the server and sends it back as a response. The response can simply object or a complex record(s).
2. **POST** - A request with POST verb creates an object(s) on the server. It is used for INSERT operations. The response is the object that was created on the server.
3. **PUT** - A request with PUT verb updates the object as a whole (all properties) on the server. It is used for UPDATE operations. The response is the object that was updated on the server.
4. **PATCH** - A request with PUT verb updates certain properties of the object on the server. It is used for UPDATE operations on specific properties of the object and not the whole object need not be touched. The response is the object that was updated on the server.
5. **DELETE** - A request with DELETE verb deletes the object from the server. It is used for DELETE operations. The response of the object that was deleted from the server.

#### Expectations for RESTful HTTP Responses

- If the response was generated without any errors, then the object should be sent back in the HTTP response with 200 [OK] status code.
- If the request contained any invalid data due to which the API could not generate a valid response, then the HTTP response should have 400 [Bad Request] status code.
- If the request was valid but the resources that should exist in server does not exist due to which the response couldn't be generated, then an HTTP response with the error message with 404 [Not Found] status code should be sent back.

Read more about [HTTP Status Codes](https://www.restapitutorial.com/httpstatuscodes.html){:target="\_blank"}

> Just because something returns a JSON response, it cannot be called RESTful

#### API Best Practices

- Every API endpoint should be readable and the URL should express it's intent very clearly.
- API endpoints should only work on the necessary data from the request body. Remember that an attacker can send more data in the request body.
- API should always validate the access permissions, completeness of the incoming data and then process the response. Any validation error should be thrown back to the client with appropriate status codes.
- API endpoints should be asynchronous in nature so that it does not block the threads.
- API endpoints should be protected by [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing).
- API endpoints should be obey transactions. It should not put the data into an inconsistent state for any reason.
- Any sensitive data (like username, password, tokens, salt, etc) should not be stored in API codebase or configuration files. This makes the API vulnerable to be hacked/ get compromised.
- Every API endpoint should be load tested for performance benchmarking.
- Un-necessary data that the client doesn't need should not be there in the response.
- API implementations should be exception handled properly and logged appropriately.
- There has to be a mechanism in API implementation to switch ON/OFF critical logging areas for quicker troubleshooting without having to deploy code/configuration files for starting the debugging process.
- Every API endpoint should be UNIT & INTEGRATION tested.
- Every API endpoint should be versioned. This enables us to rollout changes to API without affecting consumers who are using the old version. It also enables old consumers or new consumers to on-board into new API version without impacting their environment.
- API documentation has to be available for every version that the API exists [like [Swagger documentation]({{site.baseurl}}/asp-net-core-2-2-web-api-with-swagger/){:target="\_blank"}].
