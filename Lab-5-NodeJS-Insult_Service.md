# Lab-5-NodeJS-Insult_Service.md

## Create a Project for the NodeJS insult Service  

### Clone the app

```bash

    git clone https://github.com/jeremyrdavis/insult-starter-nodejs.git

```

### Or download the project zip file

You can download the zip file from Github by opening https://github.com/jeremyrdavis/insult-starter-springboot
and choosing, "Download ZIP" from the green, "Clone or Download" button

![](./images/lab7/lab7-sb-01-download.png)  

### Rename the folder

Rename the folder from "insult-starter-nodejs" to "insult-nouns"

### Import the app into VS Code

Open Visual Studio Code, choose "Open," and navigate to the root folder of the project

### Update the package.json file

Change the name to "insult-nouns"  from "insult-starter-nodejs":

```json

{
  "name": "insult-service",
  "version": "1.0.0",

```

### Update the package-lock.json file

Change the name to "insult-service"  from "insult-starter-nodejs":

```json

{
  "name": "insult-service",
  "version": "1.0.0",

```

### Build and run

Open a Terminal from Visual Studio Code by choosing "Terminal -> New Terminal" from the menu.  Run npm install and npm start:

```bash

    npm install && npm start

```

## Get coding!

You may still be logged into OpenShift.  You can check by running the following command:

```bash

oc whoami

```

If the response is your username then you are still logged in.  If you are still logged in you can skip the next step.

#### Log back in to OpenShift

Nodeshift will build a Docker container and deploy it to OpenShift for us, but we need to be logged in first.  From your OpenShift console copy the login command by clicking on your name in the top right and choosing, "Copy Login Command."

![](./images/4-1/04-copy_login_command.png)  

Paste and enter the command into your terminal

![](./images/4-1/vscode-03-login.png)  


```bash

    npm install && npm run openshift

```
### Validating the deployment:  

1. Login to OpenShift Console - with your user name and password
2. Click on Project ‘red-hat-summit-2019’ if you are not already in that project
3. You should see 1 running pod and a url that you can access
4. Try the url

You should see:

![](./images/lab3/lab-03-njs-browser_verify_locally.png)  


![](./images/4-1/06-greeting_service.png)  

## Get coding!

### Create and fail a JUnit Test for our endpoint

We are of course practicing TDD in this tutorial so our first step will be to write a Unit Test.  Create a new file, "insult-test.js," in the "test" directory with the following content:

```javascript

const test = require('tape');
const supertest = require('supertest');

const app = require('../app');

test('test insult', t => {
  supertest(app)
    .get('/api/insult')
    .expect('Content-Type', /json/)
    .expect(200)
    .then(response => {
      t.not(response.body.content, null);
      t.end();
    });
});

````

```bash

    npm test

    ...

    total:     3
    passing:   2
    failing:   1

```

Your test should of course fail.  If it doesn't feel free to raise your hand and ask for help

### Stub out an insult method

Add the following method to app.js:

```javascript

app.use('/api/insult', (request, response) => {
  const name = request.query ? request.query.name : undefined;
  response.send({content: `Verily, ye be a malmsey-nosed, unmuzzled whey-face!`});
});

```

And re-run the tests, which this time, should pass:

```bash

    npm test

    ...

    total:     3
    passing:   3
    duration:  165ms

```
Of course we aren't actually doing anything.

## Pass our test

Let's create 2 new services to retrieve adjectives and a noun.  Create a "lib" folder in the project directory.  Add a file, "adjective-service.js" with the following content:

```javascript

const roi = require('roi');

const adjectiveService = process.env.ADJECTIVE_SERVICE || 'insult-adjectives';
const adjectivePort = process.env.ADJECTIVE_SERVICE_PORT || '8080';
const serviceUrl = `http://${adjectiveService}:${adjectivePort}/api/adjective`;

function parseResponse (response) {
  try {
    return JSON.parse(response.body).adjective;
  } catch (err) {
    console.error('Unable to parse adjective service response', response);
    console.error(err);
    return err.toString();
  }
}

module.exports = exports = {
  get: async function get () {
    return roi.get(serviceUrl).then(parseResponse);
  }
};

```

And an extremely similar "noun-service.js":

```javascript

const roi = require('roi');

const nounService = process.env.NOUN_SERVICE || 'insult-nouns';
const nounPort = process.env.NOUN_SERVICE_PORT || '8080';
const serviceUrl = `http://${nounService}:${nounPort}/api/noun`;

function parseResponse (response) {
  try {
    return JSON.parse(response.body).noun;
  } catch (err) {
    console.error('Unable to parse adjective service response', response);
    console.error(err);
    return err.toString();
  }
}

module.exports = exports = {
  get: async function get () {
    return roi.get(serviceUrl).then(parseResponse);
  }
};

```

### Update our App

Import the 2 services in App.js:

```javascript

const adj_service = require('./lib/adjective-service');
const noun_service = require('./lib/noun-service');

```

And create a method to call them and concatentate the results:

```javascript

app.use('/api/insult', (request, response) => {
  response.type('application/json');
  // call adjective and noun services
  return Promise.all([
    adj_service.get(),
    adj_service.get(),
    noun_service.get()
  ])
    // eslint-disable-next-line standard/object-curly-even-spacing
    .then(words => response.send({ insult: `Verily, ye be a ${words[0]}, ${words[1]} ${words[2]}!` }))
    .catch(e => console.error(`An unexpected error occurred: ${e}`));
});

```

We can now push to OpenShift:

```bash

npm run openshift

```