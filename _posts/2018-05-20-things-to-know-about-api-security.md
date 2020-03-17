---
title: Things to know about API Security
layout: blog
category: [Security]
excerpt: Enterprises having greatly embraced SOA (Service Oriented Architecture) which helps to decouple different subsystems in such a way that we can expose & reuse these without having to break & build existing systems. In this context, securing an API is an absolute must for enterprises. In this blog, I would like to share few things...
---

Enterprises having greatly embraced SOA (Service Oriented Architecture) which helps to decouple different subsystems in such a way that we can expose & reuse these without having to break & build existing systems. In this context, securing an API is an absolute must for enterprises.

In this blog, I would like to share few things that I learnt in my quest to secure an API from an enterprise perspective.

OWASP (Open Web Application Security Project) is a worldwide not-for-profit charitable organization focused on improving the security of software. This organization was setup in 2001. OWASP provide lots of materials on Application Security under Free and Open Software License.

**OWASP sets forth few guidelines for REST API Security.**

1. **Secure REST services must only provide HTTPS endpoints.** This protects authentication credentials in transit, for example passwords, API keys or JSON Web Tokens. It also allows clients to authenticate the service and guarantees integrity of the transmitted data.

2. **Non-public REST services must perform access control at each API endpoint.** Web services in monolithic applications implement this by means of user authentication, authorisation logic and session management.

3. **Ensure JWTs (JSON Web Tokens) are integrity protected by either a signature or a MAC.** Do not allow the unsecured JWTs: {“alg”:”none”}. In general, signatures should be preferred over MACs for integrity protection of JWTs.

4. **API keys needs to be used in Public REST services** to avoid the risk of being farmed leading to excessive bills for bandwidth or compute cycles. API keys can reduce the impact of denial-of-service attacks.

   1. Require API keys for every request to the protected endpoint.
   2. Always pass API Keys in the Request Header & not in the payload. This helps to use API Keys for other REST endpoints consuming different payloads.
   3. Return 429 “Too Many Requests” HTTP response code if requests are coming in too quickly.
   4. Revoke the API key if the client violates the usage agreement.
   5. Do not rely exclusively on API keys to protect sensitive, critical or high-value resources.

5. **A REST request or response body should match the intended content type in the header.** Otherwise this could cause misinterpretation at the consumer/producer side and lead to code injection/execution.

6. **Avoid exposing management endpoints via Internet.** If management endpoints must be accessible via the Internet, make sure that users must use a strong authentication mechanism, e.g. multi-factor.

7. **Respond with generic error messages – avoid revealing details of the failure unnecessarily.** Do not pass technical details (e.g. call stacks or other internal hints) to the client

8. **Passwords, security tokens, and API keys should not appear in the URL**, as this can be captured in web server logs, which makes them intrinsically valuable.

   - https://example.com/controller/<id>/action?apiKey=a53f435643de32 – NOT GOOD
   - https://example.com/resourceCollection/<id>/action – GOOD

9. **Write audit logs before and after security related events.** Consider logging token validation errors in order to detect attacks. Take care of log injection attacks by sanitizing log data beforehand.

10. **When designing REST API, don’t just use 200 for success or 404 for error.** Here is a selection of security related REST API status codes. Use it to ensure you return the correct code.

<table><tbody><tr><td>
  <strong>Status code</strong>
  </td><td>
  <strong>Message</strong>
  </td><td>
  <strong>Description</strong>
  </td></tr><tr><td>
  200
  </td><td>
  OK
  </td><td>
  Response
  to a successful REST API action. The HTTP method can be GET, POST, PUT, PATCH
  or DELETE
  </td></tr><tr><td>
  201
  </td><td>
  Created
  </td><td>
  The
  request has been fulfilled and resource created. A URI for the created
  resource is returned in the Location header
  </td></tr><tr><td>
  202
  </td><td>
  Accepted
  </td><td>
  The
  request has been accepted for processing, but processing is not yet complete
  </td></tr><tr><td>
  400
  </td><td>
  Bad Request
  </td><td>
  The
  request is malformed, such as message body format error
  </td></tr><tr><td>
  401
  </td><td>
  Unauthorized
  </td><td>
  Wrong
  or no authentication ID/password provided
  </td></tr><tr><td>
  403
  </td><td>
  Forbidden
  </td><td>
  It’s
  used when the authentication succeeded but authenticated user doesn’t have
  permission to the request resource
  </td></tr><tr><td>
  404
  </td><td>
  Not
  Found
  </td><td>
  When a
  non-existent resource is requested
  </td></tr><tr><td>
  406
  </td><td>
  Unacceptable
  </td><td>
  The
  client presented a content type in the Accept header which is not supported
  by the server API
  </td></tr><tr><td>
  405
  </td><td>
  Method
  Not Allowed
  </td><td>
  The
  error for an unexpected HTTP method. For example, the REST API is expecting
  HTTP GET, but HTTP PUT is used
  </td></tr><tr><td>
  413
  </td><td>
  Payload
  too large
  </td><td>
  Use it
  to signal that the request size exceeded the given limit e.g. regarding file
  uploads
  </td></tr><tr><td>
  415
  </td><td>
  Unsupported
  Media Type
  </td><td>
  The
  requested content type is not supported by the REST service
  </td></tr><tr><td>
  429
  </td><td>
  Too
  Many Requests
  </td><td>
  The
  error is used when there may be DOS attack detected or the request is
  rejected due to rate limiting
  </td></tr></tbody></table>

Hope this helps. Enjoy !!
