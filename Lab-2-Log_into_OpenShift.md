# Lab 2: Log into OpenShift

## Steps

1.  Log into the OpenShift web console
2.  Create a new project
3.  Log in from your laptop

### Log into the OpenShift web console

Open the url provided by your instructor and log in with your username and password.  Your username/password combination will be something like user15/r3dh4t1!

![](./images/2/lab2-01-console-create_project.png)  

### Create a new project

Click the blue, "Create Project" button in the top right of your screen to create a new project.  You can name your project anything you want as long as you don't want spaces or special characters.  

We suggest:
* Name: redhat-summit-insults
* Display Name: Red Hat Summit Insults
* Description: Awesome Red Hat Summit Insult Workshop

![](./images/2/lab2-02-console-create_project.png)  

![](./images/2/lab2-03-console-create_project_success.png)  

### Log in from your laptop

You can log in from a command prompt or from the terminal inside of Visual Studio Code. The first step is to get the login command from the OpenShift console.  Click on your name in the top right of the screen and choose, "Copy Login Command."  This puts the login command in your clipboard.

![](./images/2/lab2-04-console-copy_login_command.png)  

If you are working from a command prompt simply paste the command into the prompt and enter it:

```bash

oc login https://api.pro-us-east-1.openshift.com --token=EbJWOzrH7bWkp_ARZzOALheibhQoAtm3A4Ftq23cGSqx31UU

```

If you prefer using the terminal in Visual Studio Code open a new terminal by choosing "Terminal -> New Terminal" from the top menu and pasting the command into the new terminal window.

![](./images/2/lab2-05-vscode_login.png)  

You are now ready to start coding!

