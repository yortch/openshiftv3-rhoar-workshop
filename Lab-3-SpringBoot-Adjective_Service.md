## Lab 3:  Creating a SpringBoot Adjective Services

In this lab we will create 3 rest services based on SpringBoot
* Creating SpringBoot Rest Services  
	* Adjective Service  
	* Noun Service  
	* Insult Service  
* Service Discovery  
* Unit testing -  RestAssured  
* Integration testing -Arquillian Cube  
* Health Check and Service Healing  
* Circuit Breaker   


### Pre-requisites 

Visual Studio Code
JDK 8 or above
### Description

The idea of this lab is to generate to random noun and an adjective to generate an insult. It is based on the following idea:  
http://www.literarygenius.info/a1-shakespearean-insults-generator.htm  

### Steps

1. Create a new project in OpenShift
2. Clone or download the sample project from Github.
3. Add the necessary functionality to return adjectives

The project we use will be based on the REST level 0 example application from OpenShift Launcher.  You can find OpenShift Launcher at https://launch.openshift.io 


### Create a new project in OpenShift

Log into your OpenShift console if you haven't already.  Create a new project from the blue, "Create Project" button on the top right of the screen.  Our example uses the name, "red-hat-summit-2019-user1."  Be sure to append your username!

![](./images/4-1/01.png)  
![](./images/4-1/02.png)  
![](./images/4-1/03.png)  


#####  Clone the repository 

```bash

git clone https://github.com/jeremyrdavis/insult-starter-springboot

```

##### Download the project zip file

Download the zip file from Github by opening git clone https://github.com/jeremyrdavis/insult-starter-springboot
and choosing, "Download ZIP" from the green, "Clone or Download" button

![](./images/4-1/github-download_zip.png)  


##### Rename the Folder
Rename the folder from "insult-starter-springboot" to "adjective-service"

##### Import the app into VS Code

Open Visual Studio Code, choose "Open," and navigate to the root folder of the project

##### Update the project settings

We need to update our project's settings from the default starter app to the adjective service we are building.

Open the pom.xml file and change the artifactId, name, and description (lines 25-28) to 
"insult-adjectives," "Spring Boot Insult Adjective Service," and "Spring Boot Insult App for Shakespearean Insults Workshop."

```xml

25  <artifactId>insult-adjectives</artifactId>
26  <version>1.0.0</version>
27  <name>Spring Boot Insult Adjective Service</name>
28  <description>Spring Boot Insult App for Shakespearean Insults Workshop</description>

```

##### Build the app

We will use Maven to build our app.  Open a new Terminal either from the command line or within Visual Studio Code by choosing, "Terminal -> New Terminal"


```bash

mvn clean package

```

The tests should all complete successfully, and you should see a success message.

![](./images/4-1/vscode-01-clean_package.png)  

![](./images/4-1/vscode-02-build_success.png)  


#### Deploying to OpenShift  

##### Building a Docker container for OpenShift

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


##### Log in to OpenShift

Fabric8 will build a Docker container and deploy it to OpenShift for us, but we need to be logged in first.  From your OpenShift console copy the login command by clicking on your name in the top right and choosing, "Copy Login Command."

![](./images/4-1/04-copy_login_command.png)  

Paste and enter the command into your terminal

![](./images/4-1/vscode-03-login.png)  


##### Build and deploy to OpenShift

Now we can deploy our app.  From the terminal run the following maven command:

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

![](./images/4-1/vscode-04-fabric8_deploy.png)  

![](./images/4-1/vscode-05-fabric8_deploy_success.png)  


##### Validating the deployment:  

1. Login to OpenShift Console - with user userXX/r3dh4t1!
2. Click on Project ‘red-hat-summit-2019’ if you are not already in that project
3. You should see 1 running pod and a url that you can access
4. Try the url


![](./images/4-1/05-initial_deploy.png)  


You should see:


![](./images/4-1/06-greeting_service.png)  


###  Create Adjective Rest Service

Now that we got an understanding of how to build our application and deploy it to OpenShift it's time to implement some actual functionality.  We need a REST endpoint that returns an adjective.

We will be following Test Driven Development in this tutorial so our first step is to create a Unit Test.  We will use JUnit in this application.


#### Create and fail a JUnit Test for our endpoint

1. Create a new class, "AdjectiveServiceTest," in the "src/test/java/com/redhat/summit2019" directory

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
public class AdjectiveServiceTest {

    private static final String ENDPOINT_PATH = "api/adjective";

    @Value("${local.server.port}")
    private int port;

    @Test
    public void testAdjectiveEndpoint() {
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

mvn clean test -Dtest=AdjectiveServiceTest

```

The test should of course fail.  If for some reason your test passes feel free to raise your hand and ask for help.


### Pass our JUnit test

#### Steps

We are only returning a String and don't really need a domain model, but we are going to pretend that we are creating a full application and create a domain model with an Adjective class.

###### Adjective domain model

First create a new folder under "src/main/java/com/redhat/summit2019" named, "model."  Second create a class, "Adjective.java" in your new model package:


```java

package com.redhat.summit2019.model;

import java.util.Objects;

public class Adjective {


    private String adjective;

    public Adjective() {
    }

    public Adjective(String adjective) {
        this.adjective = adjective;
    }

    public String getAdjective() {
        return adjective;
    }

    public Adjective adjective(String adjective) {
        this.adjective = adjective;
        return this;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Adjective adjective1 = (Adjective) o;
        return Objects.equals(adjective, adjective1.adjective);
    }

    @Override
    public int hashCode() {
        return Objects.hash(adjective);
    }

    @Override
    public String toString() {
        final StringBuffer sb = new StringBuffer("Adjective{");
        sb.append("adjective='").append(adjective).append('\'');
        sb.append('}');
        return sb.toString();
    }

}

```

##### Create an AdjectiveRepository

Spring Data uses a repository abstraction to reduce boilerplate database code.  If you are familiar with Spring Boot you are familiar with classes like CrudRepository and JpaRepository.

In this tutorial we will use a text file instead of a database to keep things simple.  However, we will follow the Spring Data convention and create an AdjectiveRepository.

Create a new package, "com.redhat.summit2019.repository," and add a new class, "AdjectiveRepository," with the following code:

```java

package com.redhat.summit2019.repository;

import java.util.ArrayList;
import java.util.List;
import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.Random;

import com.redhat.summit2019.model.Adjective;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Component;

@Component("adjectiveStore")
public class AdjectiveRepository {

    private List<Adjective> adjectives = new ArrayList<>();

    public AdjectiveRepository(){
        try {
            Resource resource = new ClassPathResource("adjectives.txt");
            InputStream is = resource.getInputStream();
            if (is != null) {
                BufferedReader reader = new BufferedReader(new InputStreamReader(is));
                reader.lines()
                        .forEach(adj -> adjectives.add(new Adjective(adj.trim())));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public Adjective getRandomAdjective(){
        return adjectives.get(new Random().nextInt(adjectives.size()));
    }


}
```

##### Create an AdjectiveService

We now have everything we need to return an Adjective.  It is time to create an AdjectiveService to expose the REST endpoint.  Create a new class, "AdjectiveService," in the package, "com.redhat.summit2019.service" with the following code:


```java

package com.redhat.summit2019.service;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;

import com.redhat.summit2019.model.Adjective;
import com.redhat.summit2019.repository.AdjectiveRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Path("/adjective")
@Component
public class AdjectiveService {

    @Autowired
    private AdjectiveRepository adjectiveStore;
    
    @GET
    @Produces("application/json")
    public Adjective adjective() {
        return adjectiveStore.getRandomAdjective();
    }
}

```

Re-run the test case and verify that it passes.

#### Re-deploy to OpenShift

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


## Appendix

Insults File is located here for review

https://raw.githubusercontent.com/rmaddali/summit-cloudnative-workshop/master/springboot-adjective-service/src/main/resources/adjectives.txt



Edit index.html file ( under src/main/resources/static folder)

```
<html>
<head>
    <meta charset="utf-8">
    <title>API Level 0 Mission - Spring Boot</title>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css"
          integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
</head>

<body>
<div class="container">
    <div class="row">
        <div class="sect1">
<h2 id="_http_booster">HTTP Booster</h2>
<div class="sectionbody">
<div class="paragraph">
<p>Adjective Service Rest Endpoint</p>
</div>
<div class="sect2">
<h3 id="_using_the_greeting_service">Using the Adjective service</h3>

</div>
</div>
</div>

        <form class="form-inline">
            <div class="form-group">
                <label for="name">Name</label>
                <input type="text" class="form-control" id="name" placeholder="World">
            </div>
            <button id="invoke" type="submit" class="btn btn-success">Invoke</button>
        </form>

        <p class="lead">Result:</p>
        <pre><code id="greeting-result">Invoke the service to see the result.</code></pre>
    </div>
</div>

<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"
        integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa"
        crossorigin="anonymous"></script>

<script>
  $(document).ready(function () {
    $("#invoke").click(function (e) {
      var n = $("#name").val() || "World";
      $.getJSON("/api/adjective?name=" + n, function (res) {
        $("#greeting-result").text(JSON.stringify(res));
      });
      e.preventDefault();
    });
  });
</script>

</body>

</html>

```

Thats it - we are done. Let’s quickly redeploy the app to openshift  



### Recap

Our first service is complete!

