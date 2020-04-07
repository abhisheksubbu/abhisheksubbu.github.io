---
title: Backend with NodeJS
layout: blog
category: [NodeJS, Deep Dive]
excerpt: In this blog, you will learn how to create backend for your application using NodeJS.
comments: true
---

<!--<div class="alert alert-warning alert-dismissible fade show" role="alert">
  <strong>INFORMATION: </strong> This blog post is under development. Feel free to read and understand the content available so far. I will keep adding more content for you guys.
  <button type="button" class="close" data-dismiss="alert" aria-label="Close">
    <span aria-hidden="true">&times;</span>
  </button>
</div>-->

In this short deep dive, you will learn how to create backend for your application using NodeJS.

#### 1. Introducing Node & its Architecture

NodeJS is an **open source**, **cross platform** **runtime environment** for executing **javascript** code outside of a browser. Mostly, Node is used to build backend services (also called as API [Application Programming Interface]). API's are the ones that power our Web Applications or Mobile Apps. The Web Applications or Mobile Apps are just the user interface aspect on which users interact. The user interface is only powerful if it can talk to a backend service and do operations like save data to database, send emails, etc.

Node is ideal choice for developing **highly scalable**, **data intensive** and **real time** services that power client applications.

##### Benefits of using Node

1. It is a great choice for prototyping and agile development.
2. Ideal for fast and highly scalable services. Companies like Uber, Netflix, Paypal use Node a lot.
- You can build a Node app twice as fast with fewer people.
- You just need few lines of code in Node.
- A Node app would have few files compared to other frameworks.
- Node apps can serve 2x requests per second.
- Node apps mostly have faster response time.
3. Node uses Javascript everywhere.
  - If you are a javacript frontend developer, you can add NodeJS to your resume and transition into a fullstack developer in no time without learning anything new.
4. Consistent codebase.
  - Using javascript on your backend and frontend makes the tech stack more consistent from a source code perspective [same naming conventions, same tools, same best practices].
5. Node has the largest ecosystem of open source libraries.
- NPM (Node Package Manager) has tons of libraries that the developer community contributes and maintains.
- For any feature that we want in our application, mostly there is going to be an NPM package which we can import into our application and configure [instead of developing things from scratch].
6. Node is high scalable because of its non-blocking asynchronous nature. It works on single thread but with asynchrony, it uses that thread very efficiently.

##### Remember This

> **Node is not an ideal choice for CPU intensive applications.**

#### Node Architecture

Initially, browsers were the only place where we could run our javascript code. Browsers have a Javascript Engine which takes the javascript code and converts it to computer understandable machine code and runs it.

![JS Engine](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/jsEngine.PNG)

Javascript engines on various browsers are different. That's probably the reason why the same javascript code behaves differently in different browsers.

- **Microsoft Edge** uses "**Chakra**" JS Engine
- **Mozilla Firefox** uses "**SpiderMonkey**" JS Engine
- **Google Chrome** uses "**v8**" JS Engine

The javascript engine that resides inside the browser is the runtime environment for javascript code. Till the year 2009, the only way to execute javsscript was in browsers. "[Ryan Dahl](https://en.wikipedia.org/wiki/Ryan_Dahl)" (the original developer of Node) took the "**v8**" JS engine and embedded it in a C++ program and named it as "Node".

![Node](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/node.PNG)

##### Understand Node and get it fundamentally RIGHT

- Node is not a programming language. So, don't compare it with other languages like C# or Ruby.
- Don't compare Node with ASP.NET or Rails or Django. ASP.NET/Rails/Django are frameworks for building web applications. Node is not a framework. It's just a runtime environment for executing javascript code.

#### 2. Preparing the Node environment in your machine

**On Windows Operating System**
- Navigate to [NodeJS](https://nodejs.org/en/) website and download the LTS version of node
- Install the downloaded node executable

**On Ubuntu Linux**
- Run the commands one by one in the Terminal
```console
sudo apt-get install curl
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install nodejs
```
- Finally check the installed versions by executing the following commands
```console
node -v
npm -v
```

**General softwares to install**
- Install [Visual Studio Code](https://code.visualstudio.com) [if not available in your machine]
- Install [PostMan](https://www.postman.com/) for troubleshooting API endpoints

Installing node in your machine will ensure that **Node** and **NPM** are both setup properly.
That's it. We are done with preparing the environment.

#### 3. Creating the first program in Node

> Note that I am on Windows machine and the steps outline Windows Terminal commands

- Open the "[Terminal](https://www.microsoft.com/en-us/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)" [I am using the Powershell in Windows Terminal]
- "cd" in to the directory where you want your Node project to be created.
  ```console
  cd D:\NodeProjects
  ```
- use `mkdir` command to create a directory for your node project
  ```console
  mkdir my-first-node-project
  ```
- `cd` into the newly created folder
  ```console
  cd .\my-first-node-project\
  ```
- type the command `npm init --yes` and press Enter. This will create a package.json and initialize your project.
  ```console
  npm init --yes
  ```
- type the command `code .` and press Enter. This will open Visual Studio Code and load the current node project folder.
  ```console
  code .
  ```
- create a "new file" in Visual Studio Code named "app.js"
- type the following code in "app.js"
  ```console
  console.log('Hello World');
  ```
- In Visual Studio Code, press `` Ctrl + ` `` . This will open the Terminal inside Visual Studio Code
- Here, type the following command and press Enter. You should see "**Hello World**" displayed in the terminal console.
  ```console
  node app.js
  ```
  ![Node Hello World](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/node-hello-world.PNG)

#### 4. Setting up Express Package

- Switch back to the Terminal and type the following command to install "**express**" package
  ```console
  npm i express
  ```
- Switch back to Visual Studio Code. You will be able to see a "node_modules" folder with express dependencies in it. Great !!
- Delete the `console.log('Hello World');` line and type the following code and save.

  ```javascript
  const express = require('express');
  const app = express();

  app.get('/', (req, res) => {
    res.send('Hello World');
  });

  const port = process.env.POT || 3000;
  app.listen(port, () => {
    console.log(`Listening to port ${port} ...`);
  });
  ```

- In Visual Studio Code Terminal, type the following command and press Enter. You should see "**Listening to port 3000**" displayed in the terminal console.
  ```console
  node app.js
  ```
- Now, open the browser and type "**http://localhost:3000/**" and press Enter. You should see "**Hello World**" display in the browser page.

  ![Hello World Express](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/hello-world-express.PNG)

Now, you have successfully set up Express with Node and the "/" endpoint of your API is running and returning a response. Awesome :)

Ok. Let's get a little serious and move away from the "Hello World" examples.

> Let's develop an API to manage a list of criminals [I mean ficticious criminals] with in-memory data management

- Edit the code in "app.js" to add the following
  1. Add a "/api/criminals" GET endpoint that returns a list of criminals
  2. Add a "/api/criminals" POST endpoint that takes a criminal record and adds it to the criminal records.

```javascript
const express = require('express');
const app = express();

app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

const ficticiousCriminals = [
  { name: 'Mr.Pereira', type: 'Smuggler' },
  { name: 'Benjamin Bruno', type: 'Smuggler' }
];

app.get('/api/criminals', (req, res) => {
  res.send(ficticiousCriminals);
});

app.post('/api/criminals', (req, res) => {
  const newCriminal = {
    name: req.body.name,
    type: req.body.type
  };
  ficticiousCriminals.push(newCriminal);
  res.send(ficticiousCriminals);
});

const port = process.env.POT || 3000;
app.listen(port, () => {
  console.log(`Listening to port ${port} ...`);
});
```

- Open PostMan and do a **GET** request to "**/api/criminals**" endpoint. You should get the 2 criminals in the response.

![Get Criminals](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/get+criminals.PNG)

- In PostMan, do another **POST** request to "**/api/criminals**" endpoint with a request body as JSON with new criminal's name & type. You should now get the criminal list with 3 criminals.

![Post Criminal](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/post+criminals.PNG)

An API development this fast & easy is the power of NodeJS.
I believe this deep dive was usefull for you. In the next deep dive, we will focus on moving the in-memory data to a persistant data storage.
