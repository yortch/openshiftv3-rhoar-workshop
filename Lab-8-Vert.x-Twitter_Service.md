# Lab-9-Vert.x-Twitter_Service.md

## Create a Project for the Vert.x Twitter Service  
git clone https://github.com/jeremyrdavis/insult-starter-vertx

## Set the active project

Use the oc application to see which project you are using:

```bash

oc projects

...

* istio-system
userXX-insult-app

```

If you do not have an asterisk next to userXX-insult-app run the following command to set userXX-insult-app as the active project:

```bash

oc project userXX-insult-app

```

In this lab we will create a microservice that takes a JSON payload and sends it to Twitter.

##  Clone the repository 

If you've come this far you've probably figured out the first step by now

```bash

git clone https://github.com/jeremyrdavis/insult-starter-vertx

```

### Or download the project zip file

You can download the zip file from Github by opening https://github.com/jeremyrdavis/insult-starter-vertx

and choosing, "Download ZIP" from the green, "Clone or Download" button

![](./images/lab7/lab7-sb-01-download.png)  

### Import the app into VS Code

Open Visual Studio Code, choose "Open," and navigate to the root folder of the project

### Update the project settings

We need to update our project's settings from the default starter app to the twitter service we are building.  Open the pom.xml file and change the artifactId, name, and description to 
"twitter-service," and "Vert.x Insult Twitter Service," and "Red Hat Summit 2019 Insult Workshop Twitter Service":


```xml

  <groupId>com.redhat.summit2019</groupId>
  <artifactId>twitter-service</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>
  <name>Vert.x Insult Twitter Service</name>
  <description>Red Hat Summit 2019 Insult Workshop Twitter Service</description>

```

### Verify that the project builds

```bash

mvn clean package

```

The tests should all complete successfully, and you should see a success message.

![](./images/4-1/vscode-01-clean_package.png)  

![](./images/4-1/vscode-02-build_success.png)  

### Verify that the project deploys to OpenShift  

We will use the Fabric8 Maven Plugin to deploy our application to OpenShift.  The fabric8 plugin is already part of your pom.xml.

```xml

          <plugin>
            <groupId>io.fabric8</groupId>
            <artifactId>fabric8-maven-plugin</artifactId>
            <executions>
              <execution>
                <id>fmp</id>
                <goals>
                  <goal>resource</goal>
                  <goal>build</goal>
                </goals>
              </execution>
            </executions>
          </plugin>

```

You can read more about the Fabric8 project here, http://fabric8.io/

