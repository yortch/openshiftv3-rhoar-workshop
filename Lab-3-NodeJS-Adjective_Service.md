# Lab-3-NodeJS-Adjective_Service.md

## Create a Project for the Spring Boot Twitter Service  

### Clone the app

```bash

git clone https://github.com/jeremyrdavis/insult-starter-nodejs.git

```

### Or download the project zip file

You can download the zip file from Github by opening https://github.com/jeremyrdavis/insult-starter-springboot
and choosing, "Download ZIP" from the green, "Clone or Download" button

![](./images/lab7/lab7-sb-01-download.png)  

### Rename the folder

Rename the folder from "insult-starter-nodejs" to "insult-adjectives"

### Import the app into VS Code

Open Visual Studio Code, choose "Open," and navigate to the root folder of the project

### Update the package.json file

Change the name to "insult-adjectives"  from "insult-starter-nodejs":

```json

{
  "name": "insult-adjectives",
  "version": "1.0.0",

```

### Build and run

Open a Terminal from Visual Studio Code by choosing "Terminal -> New Terminal" from the menu.  Run npm install and npm start:

```bash

    npm install && npm start

```

### Deploy to OpenShift

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

    npm run deploy

```
### Validating the deployment:  

1. Login to OpenShift Console - with your user name and password
2. Click on Project ‘red-hat-summit-2019’ if you are not already in that project
3. You should see 1 running pod and a url that you can access
4. Try the url

You should see:

![](./images/lab3/lab-03-njs-browser_verify_locally.png)  


![](./images/4-1/06-greeting_service.png)  


