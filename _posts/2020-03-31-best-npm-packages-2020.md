---
title: Best NPM Packages for Developers in 2020
layout: blog
category: [Productivity, Node]
excerpt: In this post, I will list out the top 10 best npm packages that you should use in your Node Projects
comments: true
---

## [1. Express](https://www.npmjs.com/package/express)

Express is the first choice for developers who build API's using NodeJS. It has best in class routing features, high performance, helper methods, support for templating engines and many more. The best part of using express is that it's easy and fast to work with.

##### Setup Steps

1. In your node project, install express by executing the command in shell/cmd

   ```console
   npm i express
   ```

2. In the app.js or index.js, write the following code to configure the root GET route.

   ```javascript
   const express = require('express');
   const app = express();

   app.get('/', function(req, res) {
     res.send('Hello World');
   });

   app.listen(3000);
   ```

## [2. Mongoose](https://www.npmjs.com/package/mongoose)

Mongoose is an object modelling wrapper over the MongoDB. It helps to define schema and model classes for NoSQL collections. Mongoose supports asynchrony with the help of callbacks and promises.

##### Setup Steps

1. In your node project, install mongoose by executing the command in shell/cmd

   ```console
   npm i mongoose
   ```

2. In the app.js or index.js, write the following code to configure the root GET route.

   ```javascript
   const mongoose = require('mongoose');
   mongoose
     .connect('mongodb://localhost/genreDatabase', {
       useNewUrlParser: true,
       useUnifiedTopology: true
     })
     .then(() => console.log('Connected to database...'))
     .catch(err => console.log('Error', err));

   const genreSchema = new mongoose.Schema({
     name: {
       type: String,
       required: true
     }
   });

   const Genre = mongoose.model('Genre', genreSchema);

   //Now, you can use Genre class to create/manipulate/delete objects

   app.listen(3000);
   ```

## [3. Nodemon](https://www.npmjs.com/package/nodemon)

Nodemon is a simple package that helps you to restart the NodeJS application when file changes are detected. This comes in handy as developers need not use "npm start" or "node index.js" everytime they need to restart the application.

##### Setup Steps

1. In your node project, install mongoose by executing the command in shell/cmd

   ```console
   npm i nodemon
   ```

2. When you want to run the application, just type the following command and the application will detect changes and auto-restart
   ```console
   nodemon index.js
   ```

## [4. Helmet](https://www.npmjs.com/package/helmet)

Helmet is an effective package that helps you secure your application by setting various HTTP headers. This package ensures that there is a certain set of protection set in your application by default. It's better than having no security. There are more options that you set in your helmet middleware configuration. Refer helmet docs for more information.

##### Setup Steps

1. In your node project, install mongoose by executing the command in shell/cmd

   ```console
   npm i helmet
   ```

2. In the app.js or index.js, write the following code to configure the helmet middleware.

   ```javascript
   const express = require('express');
   const helmet = require('helmet');

   const app = express();

   // attach helmet as a middleware
   app.use(helmet());
   ```

## [4. CORS](https://www.npmjs.com/package/cors)

CORS package helps provide the ability to configure the **Allow-Control-Allow-Origin** header inside the application code. This is much better than setting your headers with plain text in your application code. CORS is mostly used in conjuction with Express for ensuring that your api endpoints are allowed to use form known origins.

##### Setup Steps

1. In your node project, install mongoose by executing the command in shell/cmd

   ```console
   npm i cors
   ```

2. In the app.js or index.js, write the following code to configure the CORS middleware.

   ```javascript
   const express = require('express');
   const cors = require('cors');

   const app = express();

   // attach CORS as a middleware
   app.use(
     cors({
       origin: 'http://localhost:4200'
     })
   );

   app.get('/', function(req, res) {
     res.send(
       'This API endpoint on 3000 port can only be reached from localhost:4200'
     );
   });

   app.listen(3000);
   ```

## [5. Config](https://www.npmjs.com/package/config)

Config package comes handy in specifying default and environment specific configurations stored as JSON files. It can detect the process.env.NODE_ENV and pick the right configurations automatically.

##### Setup Steps

1. In your node project, install mongoose by executing the command in shell/cmd

   ```console
   npm i config
   ```

2. Create a json file named "development.json" [indicates the json file is for development environment]

   ```json
   {
     "appName": "My App",
     "mail": {
       "host": "server host url"
     }
   }
   ```

3. In the app.js or index.js, write the following code to read the configuration values.

   ```javascript
   const config = require('config');

   const appName = config.get('appName');
   const hostUrl = config.get('mail.host');
   ```

## [6. Debug](https://www.npmjs.com/package/debug)

Debug package helps us specify debug statements rather than polluting with console.log. You can choose to execute debug statements for a particular environment (like development) and not to execute it on production environments. You can also get creative with process variables and have the application log debug statements in a particular time period for us to understand the issue going on.

##### Setup Steps

1. In your node project, install mongoose by executing the command in shell/cmd

   ```console
   npm i debug
   ```

2. In the app.js or index.js, write the following code for debugging statements.

   ```javascript
   const debug = require('debug');
   if (process.env.NODE_ENV === 'development')
     debug('This is a debug statement');
   ```

## [7. Morgan](https://www.npmjs.com/package/morgan)

Morgan is an HTTP request logger middleware for NodeJS. It is usefull when you have amany middlewares and you want to know the sequence of the request flow through various middlewares in your application. Morgan can log the request pipeline flow in various formats.

##### Setup Steps

1.  In your node project, install mongoose by executing the command in shell/cmd

    ```console
    npm i morgan
    ```

2.  In the app.js or index.js, write the following code to log the HTTP request pipeline.

    ```javascript
    var express = require('express');
    var morgan = require('morgan');
    var app = express();

    app.use(morgan('combined'));

    app.get('/', function(req, res) {
      res.send('hello, world!');
    });
    ```

## [8. React](https://www.npmjs.com/package/react)

React is a popular javscript library for creating user interfaces. It provides ability to create components and render the components by interacting with DOM elements (for web) or native elements (for native environment).

##### Setup Steps

1.  In your node project, install mongoose by executing the command in shell/cmd

    ```console
    npm i react
    ```
