# Lab 5 Vert.x Insult Service

## Create the Spring Boot Insult Service

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

In this lab we will create a microservice that returns a complete insult.  We will call the Adjective service twice, the Noun service, and concatenate the results with our basic insult text.

##  Clone or download the repository 

```bash

git clone https://github.com/jeremyrdavis/insult-starter-vertx.git

```

Download the zip file from Github by opening https://github.com/jeremyrdavis/insult-starter-vertx
and choosing, "Download ZIP" from the green, "Clone or Download" button

![](./images/lab3/lab-03-vertx-02-browser_clone_download.png)  

## Rename the Folder

Rename the folder from "insult-starter-springboot" to "insult-service"

## Import the app into VS Code

Open Visual Studio Code, choose "Open," and navigate to the root folder of the project

## Update the app

Second open the pom.xml and change "artifactId," "name," and "description" to "shakespearean-insults," "Shakespearean Insults," and "Red Hat Summit 2019 Shakespearean Insult Workshop" respectively:

```xml

  <modelVersion>4.0.0</modelVersion>
  <groupId>com.redhat.summit2019</groupId>
  <artifactId>insult-service</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>
  <name>Shakespearean Insults</name>
  <description>Red Hat Summit 2019 Shakespearean Insult Workshop</description>

``` 

## Building and deploying

### Building the app

We will use Maven to build our app.  Open a new Terminal either from the command line or within Visual Studio Code by choosing, "Terminal -> New Terminal"


```bash

mvn clean package

```

The tests should all complete successfully, and you should see a success message.

![](./images/lab3/lab-03-vertx-03-vscode_build_success.png)  

### Deploying to OpenShift  

#### Building a Docker container for OpenShift

We will use the Fabric8 Maven Plugin to deploy our application to OpenShift.  The fabric8 plugin is already part of your pom.xml.  Check out lines 115-140:

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

#### Build and deploy to OpenShift

Fabric8 will build a Docker container and deploy it to OpenShift for us, but we need to be logged in first.  You shoule be logged into OpenShift, and you can verify by typing the following into your terminal:

```bash

oc whoami

```

OpenShit will return your username.  If you do not have your username returned open the OpenShift console in a browser and copy the login command by clicking on your name in the top right and choosing, "Copy Login Command."

![](./images/4-1/04-copy_login_command.png)  

Paste and enter the command into your terminal

![](./images/4-1/vscode-03-login.png)  


Now we can deploy our app.  From the terminal run the following maven command:

```bash

mvn clean fabric8:deploy -Popenshift 

```

This build will take longer because we are building Docker containers in addition to our Vert.x application.  When the build and push to OpenShift is complete you will see a success message similar to the following:

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

##  Create an Insult Rest Service

Now that we got an understanding of how to build our application and deploy it to OpenShift it's time to implement some actual functionality.  We need a REST endpoint that returns a noun.

We will be following Test Driven Development in this tutorial so our first step is to create a Unit Test.  We will use JUnit in this application.

### Create and fail a JUnit Test for our endpoint

Testing reactive, asynchronouse code is different than testing traditional, imperative code.  Vert.x provides "Vertx Unit" to make writing asynchronous unit tests easier.  You can find the dependency on line 54 of the project's pom.xml:

```xml

    <dependency>
      <groupId>io.vertx</groupId>
      <artifactId>vertx-unit</artifactId>
      <scope>test</scope>
    </dependency>

```

The Vertx Unit Api borrows from existing test frameworks like JUnit or QUnit and follows typical Vert.x practices.

Let's get started wiht asynchronous testing.  Create a TestCase, "NounEndpointTest" in the "com.redhat.summit2019" package.  The test case will make an asynchronous call to our Noun endpoint and verify that we get a result.

Add the following content to the InsultEndpointTest class:

```java

package com.redhat.summit2019;

import io.vertx.core.DeploymentOptions;
import io.vertx.core.Vertx;
import io.vertx.core.file.AsyncFile;
import io.vertx.core.file.OpenOptions;
import io.vertx.core.json.JsonObject;
import io.vertx.core.parsetools.RecordParser;
import io.vertx.ext.unit.Async;
import io.vertx.ext.unit.TestContext;
import io.vertx.ext.unit.junit.VertxUnitRunner;
import io.vertx.ext.web.client.WebClient;
import org.junit.After;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;

import java.util.ArrayList;
import java.util.List;

import static junit.framework.TestCase.assertTrue;

@RunWith(VertxUnitRunner.class)
public class InsultEndpointTest {

    private static final int PORT = 8081;

    private Vertx vertx;
    private WebClient client;

    @Before
    public void before(TestContext context) {
        vertx = Vertx.vertx();
        vertx.exceptionHandler(context.exceptionHandler());
        vertx.deployVerticle(HttpApplication.class.getName(),
            new DeploymentOptions().setConfig(new JsonObject().put("http.port", PORT)),
            context.asyncAssertSuccess());
        client = WebClient.create(vertx);
    }

    @After
    public void after(TestContext context) {
        vertx.close(context.asyncAssertSuccess());
    }

    @Test
    public void callInsultEndpoint(TestContext context) {
        // Send a request and get a response
        Async async = context.async();
        client.get(PORT, "localhost", "/api/insult")
            .send(resp -> {
                context.assertTrue(resp.succeeded());
                context.assertEquals(resp.result().statusCode(), 200);
                String insult = resp.result().bodyAsJsonObject().getString("insult");
                context.assertNotNull(insult);
                async.complete();
            });
    }
}

```

There are 2 things happening in the Before method that we will look at.  First we use our class' Vertx instance to configure our TestContext and deploy our Verticle:

```java

        vertx = Vertx.vertx();
        vertx.exceptionHandler(context.exceptionHandler());
        vertx.deployVerticle(HttpApplication.class.getName(),
            new DeploymentOptions().setConfig(new JsonObject().put("http.port", PORT)),
            context.asyncAssertSuccess());

```

Second we get a Vertx WebClient from our TestContext.  We will use the WebClient to excercise our endpoint in the test method.

```java

        client = WebClient.create(vertx);

```

There are also 2 import things to take note of in the test mthod:

1.  The method takes a single argument, "TestContext."  Vertx Unit provides a TestContext that is used for performing assertions and completing the test

```java

    @Test
    public void callInsultEndpoint(TestContext context) {
        // Send a request and get a response
        ...
    }

```

2.  The TestContext provides us with get an "Async" object, and we complete the test by calling, "async.complete()"

```java

    @Test
    public void callInsultEndpoint(TestContext context) {
        // Send a request and get a response
        Async async = context.async();
        client.get(PORT, "localhost", "/api/insult")
            .send(resp -> {
                
                ...
                
                async.complete();
            });
    }

```

3.  We use the Vertx WebClient to call our endpoint

```java

        client.get(PORT, "localhost", "/api/insult")
            .send(resp -> {

                ...

            });
    }

```

4.  We excercise asserting with the TestContext

```java

    context.assertTrue(resp.succeeded());
    context.assertEquals(resp.result().statusCode(), 200);

```

Run the test either by Clicking the "Run Test" link in the IDE (just under the @Test annotation) or in the terminal with:

```bash

mvn clean test -Dtest=InsultEndpointTest

```

The test should of course fail.

### Pass our JUnit test

The Vertx Web module makes it easy to build webapps, but we are only implementing a single endpoint so we will stick with basic HTTP functionality.  Open the HttpApplication class and add the following method to handle returning an insult (we will hard code an insult for our initial pass):

#### Create a method to handle GET requests

```java

  private void insultHandler(RoutingContext rc) {

    JsonObject response = new JsonObject()
            .put("insult", "{\"insult\":\"Verily, ye be a puking, beslubbering pantaloon!\"}");

    rc.response()
            .putHeader(CONTENT_TYPE, "application/json; charset=utf-8")
            .end(response.encodePrettily());
  }

```

This method will temporarily enable us to pass our test because we are hardcoding the expected result.  We will fix that shortly, but first we will update our test case.


##### RoutingContext 

Our handler method takes a Vert.x RoutingContext as an argument.  

The RoutingContext encapsulates the context for handling an Http request.  Vert.x creates a new RoutingContext for each HTTP request that is handled by Router#handle(HttpServerRequest) methods and automatically discards it after processing is finished.

The RoutingContext provides access to the HttpServerRequest and HttpServerResponse and allows you to maintain arbitrary data that lives for the lifetime of the context.

##### JsonObject

Vert.x' JsonObject encapsulates the notion of a JSON Object and makes it extremely easy to work with JSON.

#### Add the route

We will add the following route just after the existing route for "/api/greeting":

```java

    router.get("/api/insult").handler(this::insultHandler);

```

The entire class will now be:

```java

package com.redhat.summit2019;

import io.vertx.core.AbstractVerticle;
import io.vertx.core.Future;
import io.vertx.core.json.JsonObject;
import io.vertx.ext.web.Router;
import io.vertx.ext.web.RoutingContext;
import io.vertx.ext.web.handler.StaticHandler;

import static io.vertx.core.http.HttpHeaders.CONTENT_TYPE;

public class HttpApplication extends AbstractVerticle {

  static final String template = "Hello, %s!";

  @Override
  public void start(Future<Void> future) {
    // Create a router object.
    Router router = Router.router(vertx);

    router.get("/api/greeting").handler(this::greeting);
    router.get("/api/insult").handler(this::insultHandler);
    router.get("/*").handler(StaticHandler.create());

    // Create the HTTP server and pass the "accept" method to the request handler.
    vertx
        .createHttpServer()
        .requestHandler(router)
        .listen(
            // Retrieve the port from the configuration, default to 8080.
            config().getInteger("http.port", 8080), ar -> {
              if (ar.succeeded()) {
                System.out.println("Server started on port " + ar.result().actualPort());
              }
              future.handle(ar.mapEmpty());
            });

  }

  private void insultHandler(RoutingContext rc) {

    JsonObject response = new JsonObject()
            .put("insult", "{\"insult\":\"Verily, ye be a puking, beslubbering pantaloon!\"}");

    rc.response()
            .putHeader(CONTENT_TYPE, "application/json; charset=utf-8")
            .end(response.encodePrettily());
  }

  private void greeting(RoutingContext rc) {
    String name = rc.request().getParam("name");
    if (name == null) {
      name = "World";
    }

    JsonObject response = new JsonObject()
        .put("content", String.format(template, name));

    rc.response()
        .putHeader(CONTENT_TYPE, "application/json; charset=utf-8")
        .end(response.encodePrettily());
  }
}


```

Re-run the test case and verify that it passes.

```bash

mvn clean test -Dtest=InsultEndpointTest

```

The test should pass, but we aren't actually doing anything.  Let's call the existing endpoints to get a result.

#### Update our Test Case

We will refactor our HttpApplication to get the port numbers and urls from the application's configuration.  Update the "before method" and add your existing OpenShift url's:

```java

        vertx.deployVerticle(HttpApplication.class.getName(),
                new DeploymentOptions().setConfig(
                        new JsonObject()
                                .put("http.port", PORT)
                                .put("adjective.url", "insult-adjectives-redhat-summit-insult-workshop-vertx.b9ad.pro-us-east-1.openshiftapps.com")
                                .put("noun.url", "insult-starter-vertx-redhat-summit-insult-workshop-vertx.b9ad.pro-us-east-1.openshiftapps.com")
                ),
                context.asyncAssertSuccess());

```

The complete class should be:

```java

package com.redhat.summit2019;

import io.vertx.core.DeploymentOptions;
import io.vertx.core.Vertx;
import io.vertx.core.file.AsyncFile;
import io.vertx.core.file.OpenOptions;
import io.vertx.core.json.JsonObject;
import io.vertx.core.parsetools.RecordParser;
import io.vertx.ext.unit.Async;
import io.vertx.ext.unit.TestContext;
import io.vertx.ext.unit.junit.VertxUnitRunner;
import io.vertx.ext.web.client.WebClient;
import org.junit.After;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;

import java.util.ArrayList;
import java.util.List;

import static junit.framework.TestCase.assertTrue;

@RunWith(VertxUnitRunner.class)
public class InsultEndpointTest {

    private static final int PORT = 8081;

    private Vertx vertx;
    private WebClient client;

    @Before
    public void before(TestContext context) {
        vertx = Vertx.vertx();
        vertx.exceptionHandler(context.exceptionHandler());
        vertx.deployVerticle(HttpApplication.class.getName(),
                new DeploymentOptions().setConfig(
                        new JsonObject()
                                .put("http.port", PORT)
                                .put("adjective.port", 80)
                                .put("noun.port", 80)
                                .put("adjective.url", "insult-adjectives-redhat-summit-insult-services.b9ad.pro-us-east-1.openshiftapps.com")
                                .put("noun.url", "insult-nouns-redhat-summit-insult-services.b9ad.pro-us-east-1.openshiftapps.com")
                ),
                context.asyncAssertSuccess());
        client = WebClient.create(vertx);
    }

    @After
    public void after(TestContext context) {
        vertx.close(context.asyncAssertSuccess());
    }

    @Test
    public void callInsultEndpoint(TestContext context) {
        // Send a request and get a response
        Async async = context.async();
        client.get(PORT, "localhost", "/api/insult")
            .send(resp -> {
                context.assertTrue(resp.succeeded());
                context.assertEquals(resp.result().statusCode(), 200);
                String insult = resp.result().bodyAsJsonObject().getString("insult");
                context.assertNotNull(insult);
                async.complete();
            });
    }
}

````

#### Calling the Adjective and Noun services

Time for some major changes (and reactive code.) 

####  Create our Domain Model

We are only returning a String and don't really need a domain model, but to be consistent with the rest of the tutorial and of course real applications we will create a domain model for our application.  

Create a new package "com.redhat.summit2019.model" (or folder structure "src/main/java/com/redhat/summit2019/model") and a new file, "Insult.java" in the model package.

Our Insult model will contain 2 Adjectives and 1 Noun and will return an insult in the format of "Verily, ye be a cockle-brained, puking measle!"  

```java

package com.redhat.summit2019.model;

import io.vertx.core.json.JsonObject;

public class Insult {

    String adjective1;

    String adjective2;

    String noun;

    public Insult(String adjective1, String adjective2, String noun) {
        this.adjective1 = adjective1;
        this.adjective2 = adjective2;
        this.noun = noun;
    }

    public Insult(JsonObject adjective1, JsonObject adjective2, JsonObject noun) {
        this.adjective1 = adjective1.getString("adjective");
        this.adjective2 = adjective2.getString("adjective");
        ;
        this.noun = noun.getString("noun");
    }

    public String getInsult() {
        StringBuilder builder = new StringBuilder();
        builder.append("Verily, ye be a ");
        builder.append(adjective1);
        builder.append(", ");
        builder.append(adjective2);
        builder.append(" ");
        builder.append(noun);
        builder.append("!");
        return builder.toString();
    }

    public Insult() {
    }

    @Override
    public String toString() {
        StringBuilder builder = new StringBuilder();
        builder.append("{");
        builder.append("\"insult\":\"");
        builder.append(getInsult());
        builder.append("\"");
        builder.append("}");
        return builder.toString();

    }

    public String getAdjective1() {
        return adjective1;
    }

    public void setAdjective1(String adjective1) {
        this.adjective1 = adjective1;
    }

    public String getAdjective2() {
        return adjective2;
    }

    public void setAdjective2(String adjective2) {
        this.adjective2 = adjective2;
    }
}
```

#### Update HttpApplication

##### Change our imported classes

The first thing we need to do is to swap out a lot of our imports. Replace the "io.vertx.core" and "io.vertx.ext" packages with "io.vertx.reactivex.core" and "io.vertx.reactivex.ext" for the following import statements:

```java

import io.vertx.reactivex.core.AbstractVerticle;
import io.vertx.reactivex.ext.web.Router;
import io.vertx.reactivex.ext.web.RoutingContext;
import io.vertx.reactivex.ext.web.client.HttpResponse;
import io.vertx.reactivex.ext.web.client.WebClient;
import io.vertx.reactivex.ext.web.handler.StaticHandler;

```

##### Add a WebClient

Create a Vert.x WebClient member variable.  This is the same utility that we have used in our tests (we mentioned earlier that it is used for more than tests.)  And instantiate it in the "start" method:

```java

  WebClient webClient;

    @Override
    public void start(Future<Void> future) {

        webClient = WebClient.create(vertx);

        ...

```

##### Update the start method

Be sure you initialize the webClient in the start method (above.)  The complete method is:

```java

  @Override
  public void start(Future<Void> future) {

    webClient = WebClient.create(vertx);
    
    // Create a router object.
    Router router = Router.router(vertx);

    router.get("/api/greeting").handler(this::greeting);
    router.get("/api/insult").handler(this::insultHandler);
    router.get("/*").handler(StaticHandler.create());

    // Create the HTTP server and pass the "accept" method to the request handler.
    vertx
        .createHttpServer()
        .requestHandler(router)
        .listen(
            // Retrieve the port from the configuration, default to 8080.
            config().getInteger("http.port", 8080), ar -> {
              if (ar.succeeded()) {
                System.out.println("Server started on port " + ar.result().actualPort());
              }
              future.handle(ar.mapEmpty());
            });

  }

```

##### Call our Microservices

Swap out our stubbed out "insultHandler" method in "HttpApplication" with the following code.  Don't worry if it looks odd; we will unpack it later.

```java

  private void insultHandler(RoutingContext rc) {

    Single<JsonObject> noun = webClient
            .get(config().getInteger("noun.port", 8080), config().getString("noun.url", "insult-nouns"),"/api/noun")
            .rxSend()
            .doOnSuccess(r -> System.out.println("noun" + r.bodyAsString()))
            .map(HttpResponse::bodyAsJsonObject);

    Single<JsonObject> adj1 = webClient
            .get(config().getInteger("adjective.port", 8080), config().getString("adjective.url", "insult-adjectives"), "/api/adjective")
            .rxSend()
            .doOnSuccess(r -> System.out.println("adj1" + r.bodyAsString()))
            .map(HttpResponse::bodyAsJsonObject);

    Single<JsonObject> adj2 = webClient
            .get(config().getInteger("adjective.port", 8080), config().getString("adjective.url", "insult-adjectives"), "/api/adjective")
            .rxSend()
            .doOnSuccess(r -> System.out.println("adj2" + r.bodyAsString()))
            .map(HttpResponse::bodyAsJsonObject);

    Single.zip(
            adj1.doOnError(error -> System.out.println(error.getMessage())),
            adj2.doOnError(error -> System.out.println(error.getMessage())),
            noun.doOnError(error -> System.out.println(error.getMessage())),
            Insult::new)
            .subscribe(r ->
                    rc.response()
                            .putHeader(CONTENT_TYPE, "application/json; charset=utf-8")
                            .end(r.toString()));
  }

```

And add a method to handle errors:

```java

    private void error(RoutingContext rc, Throwable error){
        rc.response()
                .setStatusCode(500)
                .putHeader(CONTENT_TYPE, "application/json; charset=utf-8")
                .end(new JsonObject().put("error", error.getMessage()).encodePrettily());
    }


```

##### rxJava Single

Unles you have used rxJava before this method probably looks strange.  A "Single" ...

```java

    Single<JsonObject> noun = webClient
            .get(config().getInteger("noun.port", 8080), config().getString("noun.url", "insult-nouns"),"/api/noun")
            .rxSend()
            .doOnSuccess(r -> System.out.println("noun" + r.bodyAsString()))
            .map(HttpResponse::bodyAsJsonObject);

```

##### Add a method to handle errors

```java

    private void error(RoutingContext rc, Throwable error){
        rc.response()
                .setStatusCode(500)
                .putHeader(CONTENT_TYPE, "application/json; charset=utf-8")
                .end(new JsonObject().put("error", error.getMessage()).encodePrettily());
    }


```

The entire class should look like the following:

```java

package com.redhat.summit2019;

import com.redhat.summit2019.model.Insult;
import io.reactivex.Single;
import io.vertx.core.Future;
import io.vertx.core.json.JsonObject;
import io.vertx.reactivex.core.AbstractVerticle;
import io.vertx.reactivex.ext.web.Router;
import io.vertx.reactivex.ext.web.RoutingContext;
import io.vertx.reactivex.ext.web.client.HttpResponse;
import io.vertx.reactivex.ext.web.client.WebClient;
import io.vertx.reactivex.ext.web.handler.BodyHandler;
import io.vertx.reactivex.ext.web.handler.StaticHandler;

import java.util.concurrent.TimeUnit;

import static io.vertx.core.http.HttpHeaders.CONTENT_TYPE;

public class HttpApplication extends AbstractVerticle {

  static final String template = "Hello, %s!";

  WebClient webClient;

  @Override
  public void start(Future<Void> future) {

    webClient = WebClient.create(vertx);
    
    // Create a router object.
    Router router = Router.router(vertx);

    router.get("/api/greeting").handler(this::greeting);
    router.get("/api/insult").handler(BodyHandler.create());
    router.get("/api/insult").handler(this::insultHandler);
    router.get("/*").handler(StaticHandler.create());

    // Create the HTTP server and pass the "accept" method to the request handler.
    vertx
        .createHttpServer()
        .requestHandler(router)
        .listen(
            // Retrieve the port from the configuration, default to 8080.
            config().getInteger("http.port", 8080), ar -> {
              if (ar.succeeded()) {
                System.out.println("Server started on port " + ar.result().actualPort());
                System.out.println("Config startup message " + config().getString("startup.message", "none"));
              }else{
                System.out.println("Server failed " + ar.cause());
                future.fail(ar.cause());
              }
                future.handle(ar.mapEmpty());
            });

  }

  private void insultHandler(RoutingContext rc) {

    Single<JsonObject> noun = webClient
            .get(config().getInteger("noun.port", 8080), config().getString("noun.url", "insult-nouns"),"/api/noun")
            .rxSend()
            .doOnSuccess(r -> System.out.println("noun" + r.bodyAsString()))
            .map(HttpResponse::bodyAsJsonObject);

    Single<JsonObject> adj1 = webClient
            .get(config().getInteger("adjective.port", 8080), config().getString("adjective.url", "insult-adjectives"), "/api/adjective")
            .rxSend()
            .doOnSuccess(r -> System.out.println("adj1" + r.bodyAsString()))
            .map(HttpResponse::bodyAsJsonObject);

    Single<JsonObject> adj2 = webClient
            .get(config().getInteger("adjective.port", 8080), config().getString("adjective.url", "insult-adjectives"), "/api/adjective")
            .rxSend()
            .doOnSuccess(r -> System.out.println("adj2" + r.bodyAsString()))
            .map(HttpResponse::bodyAsJsonObject);

    Single.zip(
            adj1.doOnError(error -> error(rc, error)),
            adj2.doOnError(error -> error(rc, error)),
            noun.doOnError(error -> error(rc, error)),
            Insult::new)
            .subscribe(r ->
                    rc.response()
                            .putHeader(CONTENT_TYPE, "application/json; charset=utf-8")
                            .end(r.toString()));
  }

  private void error(RoutingContext rc, Throwable error){
    rc.response()
            .setStatusCode(500)
            .putHeader(CONTENT_TYPE, "application/json; charset=utf-8")
            .end(new JsonObject().put("error", error.getMessage()).encodePrettily());
  }



  private void greeting(RoutingContext rc) {
    String name = rc.request().getParam("name");
    if (name == null) {
      name = "World";
    }

    JsonObject response = new JsonObject()
        .put("content", String.format(template, name));

    rc.response()
        .putHeader(CONTENT_TYPE, "application/json; charset=utf-8")
        .end(response.encodePrettily());
  }
}

```


Re-run the test case and verify that it passes.

```bash

mvn clean test -Dtest=InsultEndpointTest

```

The test should pass, and this time we are actually doing something.

### Build and deploy to OpenShift

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
#### Verify OpenShift deployment

You should see your pod running in OpenShift, and clicking on the url should display the default "Greeting" application.

![](./images/lab3/lab-03-vertx-04-ocp_deploy_success.png)  

![](./images/lab3/lab-03-vertx-05-ocp_greeting.png)  
