# Lab 4: Creating a SpringBoot Noun Service

## Description

In this lab we will create a rest service that returns an noun.

The idea of this lab is to generate to random noun to be used when creating insults. It is based on the following idea:  
http://www.literarygenius.info/a1-shakespearean-insults-generator.htm  

### Steps

1. Clone or download a starter app from Github
2. Build and deploy to verify our starter app
3. Create a test to excercise our functionality
4. Implement the functionality
5. Re-deploy

The project is based on the REST level 0 example application from OpenShift Launcher.  You can find OpenShift Launcher at https://launch.openshift.io 

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

##  Clone the repository 

```bash

git clone https://github.com/jeremyrdavis/insult-starter-springboot

```

### Download the project zip file

If you don't have Git, or if you simply prefer to work from a zip file you can download a zip file of the project from Github at https://github.com/jeremyrdavis/insult-starter-springboot
by choosing, "Download ZIP" from the green, "Clone or Download" button

![](./images/4-1/github-download_zip.png)  

## Rename the Folder

Rename the folder from "insult-starter-springboot" to "noun-service"

## Import the app into VS Code

Open Visual Studio Code, choose "Open," and navigate to the root folder of the project

## Update the project settings

We need to update our project's settings from the default starter app to the noun service we are building.

Open the pom.xml file and change the artifactId, name, and description to 
"insult-nouns," "Spring Boot Insult Noun Service," and "Spring Boot Insult App for Shakespearean Insults Workshop."

```xml

  <artifactId>insult-nouns</artifactId>
  <version>1.0.0</version>
  <name>Spring Boot Insult Noun Service</name>
  <description>Spring Boot Insult App for Shakespearean Insults Workshop</description>

```

## Build the app

We will use Maven to build our app.  Open a new Terminal either from the command line or within Visual Studio Code by choosing, "Terminal -> New Terminal"


```bash

mvn clean package

```

The tests should all complete successfully, and you should see a success message.

![](./images/4-1/vscode-01-clean_package.png)  

![](./images/4-1/vscode-02-build_success.png)  

## Deploying to OpenShift  

### Building a Docker container for OpenShift

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

#### Validating the deployment:  

1. Login to OpenShift Console - with user userXX/r3dh4t1!
2. Click on Project ‘userXX-insult-app’ if you are not already in that project
3. You should see 1 running pod and a url that you can access
4. Try the url


![](./images/lab3/lab-03-sb-ocp_initial_deployment.png)  


You should see:


![](./images/4-1/06-greeting_service.png)  

##  Write code!

Now that we got an understanding of how to build our application and deploy it to OpenShift it's time to implement some actual functionality.  We need a REST endpoint that returns an noun.

We will be following Test Driven Development in this tutorial so our first step is to create a Unit Test.  We will use JUnit in this application.

### Create and fail a JUnit Test for our endpoint

1. Create a new class, "NounServiceTest," in the "com.redhat.summit2019" package ("src/main/java/com/redhat/summit2019/NounServiceTest.java.")

Enter the following content:

```java

package com.redhat.summit2019;

import static io.restassured.RestAssured.given;
import static org.junit.Assert.assertNotNull;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import io.restassured.response.Response;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class NounServiceTest {

    private static final String ENDPOINT_PATH = "api/noun";

    @Value("${local.server.port}")
    private int port;

    @Test
    public void testNounService() {
        Response response = given()
           .baseUri(baseURI())
           .get(ENDPOINT_PATH)
           .then()
           .statusCode(200)
           .extract().response();
        assertNotNull(response);
        System.out.println(response.toString());
    }

    protected String baseURI() {
        return String.format("http://localhost:%d", port);
    }
}

```

Run the test either by Clicking the "Run Test" link in the IDE (just under the @Test annotation) or in the terminal with:

```bash

mvn clean test -Dtest=NounServiceTest

```

Obviously our test should fail.  If for some reason it passes feel free to raise your hand and ask for help.

### Pass our JUnit test

#### Noun domain model

We are only returning a String and don't really need a domain model, but we are going to pretend that we are creating a full application and create a domain model with an Noun class.

First create a new package "com.redhat.summit2019.model" ("src/main/java/com/redhat/summit2019/model.")  Second create a class, "Noun" in the new model package ("src/main/java/com/redhat/summit2019/model/Noun.java"):


```java

package com.redhat.summit2019.model;

import java.util.Objects;

public class Noun {
    private String noun;

    public Noun() {
    }

    public Noun(String noun) {
        this.noun = noun;
    }

    public String getNoun() {
        return noun;
    }

    public Noun noun(String noun) {
        this.noun = noun;
        return this;
    }

    public boolean equals(Object o) {
        if (this == o)
            return true;
        if ((o == null) || (getClass() != o.getClass()))
            return false;
        Noun noun1 = (Noun) o;
        return Objects.equals(noun, noun);
    }

    public int hashCode() {
        return Objects.hash(new Object[] { noun });
    }

    public String toString() {
        StringBuffer sb = new StringBuffer("Noun{");
        sb.append("noun='").append(noun).append('\'');
        sb.append('}');
        return sb.toString();
    }
}

````

##### Create a NounRepository

Spring Data uses a repository abstraction to reduce boilerplate database code.  If you are familiar with Spring Boot you are familiar with classes like CrudRepository and JpaRepository.

In this tutorial we will use a text file instead of a database to keep things simple.  However, we will follow the Spring Data convention and create an NounRepository.

Create a new package, "com.redhat.summit2019.repository," and add a new class, "NounRepositry," with the following code:

```java

package com.redhat.summit2019.repository;

import java.util.ArrayList;
import java.util.List;
import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.Random;

import com.redhat.summit2019.model.Noun;

import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Component;

@Component("nounStore")
public class NounRepository {

    private List<Noun> nouns = new ArrayList<>();

    public NounRepository(){
        try {
            Resource resource = new ClassPathResource("nouns.txt");
            InputStream is = resource.getInputStream();
            if (is != null) {
                BufferedReader reader = new BufferedReader(new InputStreamReader(is));
                reader.lines()
                        .forEach(adj -> nouns.add(new Noun(adj.trim())));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public Noun getRandomNoun(){
        return nouns.get(new Random().nextInt(nouns.size()));
    }
}

```

### Create an NounService

We now have everything we need to return an Noun.  It is time to create an NounService to expose the REST endpoint.  Create a new class, "NounService," in the package, "com.redhat.summit2019.service" with the following code:


```java

package com.redhat.summit2019.service;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.atomic.AtomicLong;

import javax.annotation.PostConstruct;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;

import com.redhat.summit2019.model.Noun;

import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Component;

@Path("/")
@Component
public class NounService {
   
    private final AtomicLong counter = new AtomicLong();
    private List<Noun> nouns = new ArrayList<Noun>();

    @GET
    @Path("/noun")
    @Produces("application/json")
    public Noun getNoun() {
        return this.nouns.get(new Random().nextInt(this.nouns.size()));
    }

    @PostConstruct
    public void loadData() {
        try {
            Resource resource = new ClassPathResource("nouns.txt");
            InputStream is = resource.getInputStream();
            
            if (is != null) {
                BufferedReader reader = new BufferedReader(new InputStreamReader(is));
                reader.lines().forEach(noun -> {
                    this.nouns.add(new Noun(noun.trim()));
                }
                );
            }
        }
        catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

Re-run the test case and verify that it passes.

Re-run the test case and verify that it passes.

### Re-deploy to OpenShift

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

### Recap

Our second service is complete!

