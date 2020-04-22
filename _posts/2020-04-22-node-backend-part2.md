---
title: Backend with NodeJS - Part 2
layout: blog
category: [NodeJS, Deep Dive]
excerpt: In this blog, we will improve the NodeJS API that we created in the Part 1 from in-memory data storage to persistant storage using MongoDB & Mongoose.
comments: true
---

> **Important:** Before you start reading this blog, I strongly urge you to read the [Backend with NodeJS - Part 1]({{site.baseurl}}/node-backend/) blog so that you will better understand the context.

As you are aware, the criminal information is currently stored in-memory within the codebase. This is not desired from a real-world perspective. In this blog, we will persist the criminal information in MongoDB rather than managing it in-memory.

1. Open the NodeJS project
2. Install the `mongoose` package
   ```console
     npm i mongoose
   ```
3. Reference the mongoose package in the app.js and connect to the mongodb locally
   ```javascript
   const mongoose = require('mongoose');
   mongoose
     .connect('mongodb://localhost/criminals')
     .then(() => console.log('Connected to database...'))
     .catch((err) => console.log('Connection to database failed.', err));
   ```
4. Create the schema of the Criminal object [this is like the table/document definition] and register the schema to the model class

   ```javascript
   const criminalSchema = mongoose.Schema({
     name: {
       type: String,
     },
     type: {
       type: String,
     },
   });

   const Criminal = mongoose.model('Criminal', criminalSchema);
   ```

5. Modify the `GET()` and `POST()` implementations by referencing the `Criminal` model class

   ```javascript
   app.get('/api/criminals', async (req, res) => {
     res.send(await Criminal.find());
   });

   app.post('/api/criminals', async (req, res) => {
     const newCriminal = new Criminal({
       name: req.body.name,
       type: req.body.type,
     });
     const result = await newCriminal.save();
     res.send(result);
   });
   ```

6. Run the app.js using the command 'node app.js`
7. Open Postman and do a GET request to `http://localhost:3000/api/criminals`, you should get an empty array as response.
8. Now, do a POST request to `http://localhost:3000/api/criminals` with the request body as shown below. You should get the response back with the created criminal record that has an `id` value. This confirms that POST is working.
   ![criminals-post](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/criminals-post.png)
9. Now, do a GET request to `http://localhost:3000/api/criminals` and you should see the newly created criminal record being shown in the response.
   ![criminals-get](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/criminals-get.png)

Just to make sure that this record is persisted in MongoDB, we can check the record using MongoDB Compass.
![mongo-compass-criminals](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/mongodb-compass-criminals.png)

In the next blog, we will structure this project in a much better way rather than having everything written in app.js. Stay tuned...
