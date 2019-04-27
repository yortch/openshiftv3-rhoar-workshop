# Lab 1:  Creating Thorntail Rest Services

In this lab we will create 3 rest services based on Thorntail
* Creating Thorntail Rest Services  
    * Adjective Service  
    * Noun Service  
    * Insult Service  
* Service Discovery  
* Unit testing -  RestAssured  
* Integration testing -Arquillian Cube  

## Pre-requisites 

Must have completed Labs 1 and 2. We will be using those components for following labs

## Get the starter project up and runnint

###  Clone the repository 

```bash

git clone https://github.com/jeremyrdavis/trusty-art.git

```

### Download the project zip file

You can download the zip file from Github instead of cloning it by opening 

TODO: INSERT_ZIP_DOWNLOAD

and choosing, "Download ZIP" from the green, "Clone or Download" button

![](./images/4-1/github-download_zip.png)  


### Import the app into VS Code

Open Visual Studio Code, choose "Open," and navigate to the root folder of the project

![](./images/lab3/lab-03-thorntail-01-vscode_import.png)  

### Build the app

We will use Maven to build our app.  Open a new Terminal either from the command line or within Visual Studio Code by choosing, "Terminal -> New Terminal"


```bash

mvn clean package

```

The tests should all complete successfully, and you should see a success message.

![](./images/lab3/lab-03-thorntail-02-vscode_build_success.png)  

### Deploying to OpenShift  

If you have followed along in the labs you should already be logged into OpenShift.  You can verify by typing the following into your terminal:

```bash

 oc whoami

```

The response will be your user name if you are logged in.  If you aren't you can copy the login command from the top right of your OpenShift terminal and log in again.

![](./images/lab2/lab2-04-console_copy_login_command.png)  

Type the command into your terminal:

```bash

oc login https://api.pro-us-east-1.openshift.com --token=EbJWOzrH7bWkp_ARZzOALheibhQoAtm3A4Ftq23cGSqx31UU

```

or into the terminal in Visual Studio Code:

![](./images/lab2/lab2-05-vscode_login.png)  

*IMPORTANT: Be sure to set the appropriate project after logging in*

```bash

 oc project red-hat-summit-insults

```


### Building a Docker container for OpenShift

We will use the Fabric8 Maven Plugin to deploy our application to OpenShift.  The fabric8 plugin is already part of your pom.xml.  Check out lines 181-209:

```xml

          <plugin>
            <groupId>io.fabric8</groupId>
            <artifactId>fabric8-maven-plugin</artifactId>
            <version>${version.fabric8-maven-plugin}</version>
            <executions>
              <execution>
                <goals>
                  <goal>resource</goal>
                  <goal>build</goal>
                </goals>
              </execution>
            </executions>
            <configuration>
              <generator>
                <includes>
                  <include>thorntail-v2</include>
                </includes>
                <excludes>
                  <exclude>webapp</exclude>
                </excludes>
              </generator>
              <enricher>
                <config>
                  <thorntail-v2-health-check>
                    <path>/</path>
                  </thorntail-v2-health-check>
                </config>
              </enricher>
            </configuration>
          </plugin>

```

You can read more about the Fabric8 project here, http://fabric8.io/

From the terminal run the following maven command:

```bash
mvn clean fabric8:deploy -Popenshift  
```

This build will take longer because we are building Docker containers in addition to our Spring Boot application.  When the build and push to OpenShift is complete you will see a success message similar to the following:

```bash
[INFO] F8: HINT: Use the command `oc get pods -w` to watch your pods start up
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  06:40 min
[INFO] Finished at: 2019-04-24T12:49:12-04:00
[INFO] ------------------------------------------------------------------------
```

### Verify OpenShift deployment

You should see your pod running in OpenShift, and clicking on the url should display the default "Greeting" application.

![](./images/lab3/lab-03-thorntail-03-ocp_initial_deployment.png)  

![](./images/lab3/lab-03-thorntail-01-ocp_greeting.png)  

##  Create Adjective Rest Service

