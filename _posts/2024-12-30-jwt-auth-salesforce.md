---
title: JWT Authentication in Salesforce
layout: blog
category: [Salesforce, JWT, Auth, Security]
excerpt: In this blog, I explain how to do JWT Auth into Salesforce. This is more secure and preferred over OAuth for services that do auth with Salesforce.
comments: true
---

For years, developers and admins have relied on OAuth as the go-to authentication protocol in Salesforce, enabling secure access and integration between applications. OAuth remains a robust and versatile option, particularly for scenarios where users need to explicitly grant permissions, such as when accessing Salesforce via frontend applications.

However, when it comes to backend services, there’s a growing need for a more secure and efficient approach to authentication. Enter [JWT (JSON Web Token) authentication](https://jwt.io/) - a technique that eliminates the need for user interaction while offering enhanced security.

While OAuth is by no means obsolete and still serves many essential use cases, JWT is particularly suited for backend services that require seamless and secure authentication to Salesforce.

In this blog, we’ll explore how you can leverage JWT to securely authenticate backend services and retrieve access tokens from Salesforce, simplifying your integrations and fortifying your security.

# Steps to follow

## 1. Connected App in Salesforce

Create a Connected App with the preferred name and email. Following are the settings you should configure properly.

- Selected OAuth Scopes
  - Manage user data via APIs (api)
  - Perform requests at any time (refresh_token, offline_access)
  - Access unique user identifiers (openid)
- Callback URL
  - Can be anything (https://www.salesforce.com just works fine) - This only needs to be proper if OAuth is done from client UI
- Digital Certificate
  - Upload the .cert or .pem that you get from the IT/Networks team.

> IT/Networks team will have a key file also. Salesforce only needs the certificate. Consuming application is what needs the key file.

- Require Secret for Refresh Token Flow = true
- Require Secret for Web Server Flow = true

On the Connected App (Manage), make sure that only the user used for integration is only allowed. Either allow by Profile pr Permission Set. Permission Set is better as you can assign this to a user and user with that permission set will only be allowed. It's much better than allowing all users under a Profile.

## 2. Consuming Application should have the following for making JWT auth call into Salesforce

1. `CLIENT_ID`(Consumer Key of the Connected App from Salesforce)
2. `USERNAME`(Username of the Salesforce User with which Auth needs to be done)
3. `LOGIN_URL`(For Sandboxes, it will be https://test.salesforce.com and For Production, it will be https://login.salesforce.com )
4. Private key OR JKS Bundle from IAM team

## 3. Algorithm for the Consuming Application to initiate JWT auth call into Salesforce

- JWT Token has to be created by the consuming application (as shown in line 10 to 21)
  - Consuming application should decide the minimum expiration to be used in the JWT payload.
- Make an HTTP POST to `${LOGIN_URL}/services/oauth2/token` with the params that constitute of
  - **grant_type** = `'urn:ietf:params:oauth:grant-type:jwt-bearer'`
  - **assertion** = the JWT token created previously
- Access token will be received in the response.

Below shown is a simple NodeJS Application that create a JWT Assertion, initiates HTTP Post to Salesforce Auth endpoint with the JWT grant_type and JWT assertion & finally receives an access token which is used to run a simple SOQL query.

```javascript
const path = require("path");
const fs = require("fs");
const jwt = require("jsonwebtoken");
const axios = require("axios");

// Salesforce details
const CLIENT_ID = "xyz";
const USERNAME = "sampleuser@tesorg.com";
const LOGIN_URL = "https://test.salesforce.com"; // Use 'https://login.salesforce.com' for production

// Path to the private key file
const PRIVATE_KEY_PATH = path.join(__dirname, "test.key"); // Replace with your private key path

// Create the JWT token for Salesforce authentication
const createJwtToken = () => {
  const privateKey = fs.readFileSync(PRIVATE_KEY_PATH, "utf8");
  const payload = {
    iss: CLIENT_ID,
    sub: USERNAME,
    aud: LOGIN_URL,
    exp: Math.floor(Date.now() / 1000) + 60 * 3, // Token expiration in 3 minutes
  };
  return jwt.sign(payload, privateKey, { algorithm: "RS256" });
};

// Authenticate and retrieve the access token and instance URL
const authenticateWithJwt = async () => {
  const token = createJwtToken();
  const params = new URLSearchParams();
  params.append("grant_type", "urn:ietf:params:oauth:grant-type:jwt-bearer");
  params.append("assertion", token);
  try {
    const response = await axios.post(
      `${LOGIN_URL}/services/oauth2/token`,
      params,
      {
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
      }
    );
    // Return both the access token and the instance URL from the response
    return {
      accessToken: response.data.access_token,
      instanceUrl: response.data.instance_url,
    };
  } catch (error) {
    console.error(
      "Error authenticating with JWT:",
      error.response ? error.response.data : error.message
    );
    throw new Error("Authentication failed");
  }
};

// Execute an SOQL query
const executeSoqlQuery = async (accessToken, instanceUrl) => {
  const query = "SELECT Id, Name FROM Account LIMIT 2"; // Replace with your SOQL query
  try {
    const response = await axios.get(
      `${instanceUrl}/services/data/v52.0/query`,
      {
        headers: {
          Authorization: `Bearer ${accessToken}`,
        },
        params: {
          q: query,
        },
      }
    );
    console.log("SOQL Query Result:", response.data.records);
  } catch (error) {
    console.error(
      "Error executing SOQL query:",
      error.response ? error.response.data : error.message
    );
  }
};

// Main function to initiate the JWT flow and run the query
const main = async () => {
  try {
    // Get the access token and instance URL
    const { accessToken, instanceUrl } = await authenticateWithJwt();
    // Pass both accessToken and instanceUrl to the SOQL query function
    await executeSoqlQuery(accessToken, instanceUrl);
  } catch (error) {
    console.error("Error:", error.message);
  }
};

main();
```

### Response received via sample NodeJS app with Authenticated User

```json
SOQL Query Result: [
  {
    attributes: {
      type: 'Account',
      url: '/services/data/v52.0/sobjects/Account/001cX000005sdVyQAI'
    },
    Id: '001cX000005wdVyQAI',
    Name: 'Parker Harris'
  },
  {
    attributes: {
      type: 'Account',
      url: '/services/data/v52.0/sobjects/Account/001cX000005fe0cQAA'
    },
    Id: '001cX000005we0cQAA',
    Name: 'Marc Benioff'
  }
]
```

### Response received via sample NodeJS app with Non-Authenticated User

```json
Error authenticating with JWT: {
  error: 'invalid_app_access',
  error_description: 'user is not admin approved to access this app'
}
```

This confirms JWT Test with successful with a valid certificate-key combination & username.

> Make sure not to expose KEY and ensure that it is stored securely in a key store.
