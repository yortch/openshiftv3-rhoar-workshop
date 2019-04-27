# Lab 5 SpringBoot Insult Service

## Create the Spring Boot Insult Service

In this lab we will create a microservice that returns a complete insult.  We will call the Adjective service twice, the Noun service, and concatenate the results with our basic insult text.

### Pre-requisites 

Must have completed labs 1-4. We will be using those components for this lab.

### Description

The idea of this lab is to call the Adjective and Noun services and generate a complete insult. It is based on the following idea:  
http://www.literarygenius.info/a1-shakespearean-insults-generator.htm  

###  Clone the repository 

```bash

git clone https://github.com/jeremyrdavis/shiny-elk.git`

```

### Or download the project zip file

Download the zip file from Github by opening https://github.com/jeremyrdavis/wooly-cucumber-zip and choosing, "Download ZIP" from the green, "Clone or Download" button

![](./images/4-1/github-download_zip.png)  

### Import the app into VS Code

Open Visual Studio Code, choose "Open," and navigate to the root folder of the project


### Build the app

We will use Maven to build our app.  Open a new Terminal either from the command line or within Visual Studio Code by choosing, "Terminal -> New Terminal"


```bash

mvn clean package

```

The tests should all complete successfully, and you should see a success message.

![](./images/4-1/vscode-01-clean_package.png)  

![](./images/4-1/vscode-02-build_success.png)  


### Deploying to OpenShift  

#### Building a Docker container for OpenShift

We will use the Fabric8 Maven Plugin to deploy our application to OpenShift.  The fabric8 plugin is already part of your pom.xml.  Check out lines 214-226:

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


#### Log in to OpenShift

You may still be logged into OpenShift.  You can check by running the following command:

```bash

oc whoami

```

If the response is your username then you are still logged in.  If you are still logged in you can skip the next step.

Fabric8 will build a Docker container and deploy it to OpenShift for us, but we need to be logged in first.  From your OpenShift console copy the login command by clicking on your name in the top right and choosing, "Copy Login Command."

![](./images/4-1/04-copy_login_command.png)  

Paste and enter the command into your terminal

![](./images/4-1/vscode-03-login.png)  


#### Build and deploy to OpenShift

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


#### Validating the deployment:  

1. Login to OpenShift Console - with your user name and password
2. Click on your project if you are not already in that project
3. You should see running pods for the Adjective, Noun, and Insult services
4. Try the url for the Insult service

##  Create Insult REST Service

We will take the same, test-driven approach to building the Insult Service that we did for the Adjective and Noun Services.

### Create and fail a JUnit Test for our endpoint

1. Create a new test class, InsultServiceTest.java

Enter the following content:

```java

package io.openshift.booster;

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
public class InsultServiceTest{

    private static final String ENDPOINT_PATH = "api/insult";

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

        assertEquals("{\"insult\":\"Verily, ye be a puking, cockle-brained pantaloon\"}", response.body().asString());
    }

    protected String baseURI() {
        return String.format("http://localhost:%d", port);
    }
}

````

Run the test either by Clicking the "Run Test" link in the IDE (just under the @Test annotation) or in the terminal with:

```bash

mvn clean test -Dtest=InsultServiceTest

```

Obviously our test should fail.  If for some reason it passes feel free to raise your hand and ask for help.

*This test will become significantly more complicated as we build out our service*

### Pass our JUnit test

#### Steps

1.  Create a domain model with an Adjective, Noun, and Insult
2.  Create Spring RestTemplates to call the Adjective and Noun Services
3.  Create an InsultService that retrieves 2 adjectives and a noun and returns a complete Insult

####  Create our Domain Models  

We are only returning a String and don't really need a domain model, but to be consistent with the rest of the tutorial and of course real applications we will create a domain models for our application.  

Create classes for our three models, "Adjective," "Noun," and "Insult."

You may be wondering why we are re-creating the Adjective and Noun classes instead of using the ones we created earlier.  The answer is that we don't want any dependencies across Services.

"Insult" in the package, "io.openshift.booster.insults.model"

Our Insult model will contain 2 Adjectives and 1 Noun and will return an insult in the format of "Verily, ye be a cockle-brained, puking measle!"  

```java

package io.openshift.booster.model;

public class Insult {

    Adjective adjective1;

    Adjective adjective2;

    Noun noun;

    public Insult(Adjective adjective1, Adjective adjective2, Noun noun) {
        this.adjective1 = adjective1;
        this.adjective2 = adjective2;
        this.noun = noun;
    }

    public String getInsult() {
        StringBuilder builder = new StringBuilder();
        builder.append("Verily, ye be a ");
        builder.append(adjective1.getAdjective());
        builder.append(", ");
        builder.append(adjective2.getAdjective());
        builder.append(" ");
        builder.append(noun.getNoun());
        builder.append("!");
        return builder.toString();
    }

    public Insult() {
    }

    @Override
    public String toString(){
        StringBuilder builder = new StringBuilder();
        builder.append("{");
        builder.append("\"insult\":\"");
        builder.append(getInsult());
        builder.append("\"");
        builder.append("}");
        return builder.toString();

    }

    public Adjective getAdjective1() {
        return adjective1;
    }

    public void setAdjective1(Adjective adjective1) {
        this.adjective1 = adjective1;
    }

    public Adjective getAdjective2() {
        return adjective2;
    }

    public void setAdjective2(Adjective adjective2) {
        this.adjective2 = adjective2;
    }

    public Noun getNoun() {
        return noun;
    }

    public void setNoun(Noun noun) {
        this.noun = noun;
    }

}


```

We also need Adjective and Noun domain models for our Insult to compile.  Create the Adjective and Noun models in the same package with the following code for Adjective:

```java

package io.openshift.booster.model;

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

    public void setAdjective(String adjective) {
        this.adjective = adjective;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Adjective adjective1 = (Adjective) o;
        return Objects.equals(getAdjective(), adjective1.getAdjective());
    }

    @Override
    public int hashCode() {

        return Objects.hash(getAdjective());
    }

    @Override
    public String toString() {
        StringBuffer sb = new StringBuffer("Adjective{");
        sb.append("adjective='").append(adjective).append('\'');
        sb.append('}');
        return sb.toString();
    }

}


```

and Noun:

```java

package io.openshift.booster.model;

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
        return Objects.hash(new Object[]{noun});
    }

    public String toString() {
        StringBuffer sb = new StringBuffer("Noun{");
        sb.append("noun='").append(noun).append('\'');
        sb.append('}');
        return sb.toString();
    }

}


```

#### Create the Insult Service

Now that we have modeled our domain we can build our service.  Create a class, "InsultService," in the "io.openshift.booster.service" package with the following code:

```java

package io.openshift.booster.service;

import io.openshift.booster.model.Adjective;
import io.openshift.booster.model.Insult;
import io.openshift.booster.model.Noun;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;

@Path("/insult")
@Component
public class InsultService {

    @Autowired
    AdjectiveService adjectiveService;

    @Autowired
    NounService nounService;

    @GET
    @Produces("application/json")
    public String insult() {

        Adjective adjective1 = adjectiveService.getAdjective();
        Adjective adjective2 = adjectiveService.getAdjective();
        Noun noun = nounService.getNoun();

        return new Insult(adjective1, adjective2, noun).toString();
    }
}


```

The class won't compile at the moment because of the 2 classes we are injecting.  We are injecting 2 services, "AdjectiveService" and "NounService"

```java

    @Autowired
    AdjectiveService adjectiveService;

    @Autowired
    NounService nounService;


```

These services will be simple but like our domain model we are creating them to reflect how a real world application would be structured.

The services are essentially the same with each one handling a single REST call to the appropriate microservice.  Let's create the 2 services:

```java

package io.openshift.booster.service;

import io.openshift.booster.model.Adjective;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class AdjectiveService {

    private final RestTemplate restTemplate = new RestTemplate();
    private final String adjectiveHost = System.getProperty("adjective.host", "http://smart-appliance:8080");

    public Adjective getAdjective() {
        return restTemplate.getForObject(adjectiveHost + "/api/adjectives", Adjective.class);
    }
}


```

and

```java

package io.openshift.booster.service;

import io.openshift.booster.model.Noun;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class NounService {

    private final RestTemplate restTemplate = new RestTemplate();
    private final String nounHost = System.getProperty("noun.host", "http://wooly-cucumber:8080");

    public Noun getNoun() {
        return restTemplate.getForObject(nounHost + "/api/noun", Noun.class);
    }
}

```

There are several things to note here.  First each service instantiates a Spring RestTemplate to handle the call to our existing microservices:

```java

    private final RestTemplate restTemplate = new RestTemplate();
    ...
    restTemplate.getForObject(nounHost + "/api/noun", Noun.class);

```

Spring's RestTemplate simplifies marshalling Json into POJO's as well as provides functionality for authentication, form submission and other functionality.  You can read more about the class here: https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/html/boot-features-resttemplate.html


