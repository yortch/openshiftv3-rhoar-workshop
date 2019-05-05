# Lab 3:  Creating a Vert.x Adjective Services

## Pre-requisites 

Must have completed labs 1-3. We will be using those components for following labs

## Description

The idea of this lab is to generate to random noun and an adjective to generate an insult. It is based on the following idea:  
http://www.literarygenius.info/a1-shakespearean-insults-generator.htm  

## Steps

1. Create a new project in OpenShift (if you haven't already)
2. Clone or download the sample project from Github
3. Add the necessary functionality to return adjectives

The project we use will be based on the REST level 0 example application from OpenShift Launcher.  You can find OpenShift Launcher at https://launch.openshift.io 


## Create a new project in OpenShift

Log into your OpenShift console if you haven't already.  Create a new project from the blue, "Create Project" button on the top right of the screen.  Our example uses the name, "red-hat-summit-2019."

![](./images/4-1/01.png)  
![](./images/4-1/02.png)  
![](./images/4-1/03.png)  


##  Clone or download the repository 

```bash

git clone https://github.com/jeremyrdavis/insult-starter-vertx.git

```

Download the zip file from Github by opening https://github.com/jeremyrdavis/insult-starter-vertx
and choosing, "Download ZIP" from the green, "Clone or Download" button

![](./images/lab3/lab-03-vertx-02-browser_clone_download.png)  

## Import the app into VS Code

Open Visual Studio Code, choose "Open," and navigate to the root folder of the project

### Update the app

Our first step will be to customize the starter application.  Open the pom.xml and change lines 5, 8, and 9 to be "insult-adjectives," "Insult Adjectives," and "Red Hat Summit 2019 Insult Workshop Adjectives Service" respectively:

```xml

  <modelVersion>4.0.0</modelVersion>
  <groupId>com.redhat.summit2019</groupId>
  <artifactId>insult-adjectives</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>
  <name>Vert.x Insult Adjectives Service</name>
  <description>Red Hat Summit 2019 Insult Workshop Adjectives Service</description>

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

#### Log in to OpenShift

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
### Verify OpenShift deployment

You should see your pod running in OpenShift, and clicking on the url should display the default "Greeting" application.

![](./images/lab3/lab-03-vertx-04-ocp_deploy_success.png)  

![](./images/lab3/lab-03-vertx-05-ocp_greeting.png)  

##  Create Adjective Rest Service

Now that we got an understanding of how to build our application and deploy it to OpenShift it's time to implement some actual functionality.  We need a REST endpoint that returns an adjective.

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

Let's get started wiht asynchronous testing.  Create a TestCase, "AdjectiveEndpointTest" in the "com.redhat.summit2019" package ("src/test/com/redhat/summit2019/" folder.)  

Our test case will make an asynchronous call to our Adjective endpoint and verify that we get a result.

Add the following content to the AdjectiveEndpointTest class:

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
public class AdjectiveEndpointTest {

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
    public void callAdjectiveEndpoint(TestContext context) {
        // Send a request and get a response
        Async async = context.async();
        client.get(PORT, "localhost", "/api/adjective")
            .send(resp -> {
                context.assertTrue(resp.succeeded());
                context.assertEquals(resp.result().statusCode(), 200);
                String adjective = resp.result().bodyAsJsonObject().getString("adjective");
                Assert.assertNotNull(adjective);
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
    public void callAdjectiveEndpoint(TestContext context) {
        // Send a request and get a response
        ...
    }

```

2.  The TestContext provides us with get an "Async" object, and we complete the test by calling, "async.complete()"

```java

    @Test
    public void callAdjectiveEndpoint(TestContext context) {
        // Send a request and get a response
        Async async = context.async();
        client.get(PORT, "localhost", "/api/adjective")
            .send(resp -> {
                
                ...
                
                async.complete();
            });
    }

```

3.  We use the Vertx WebClient to call our endpoint.  We can call any endpoint with the Vert.x WebClient.  In Lab 5 we will use it to retrieve 2 adjectives and a Noun.

```java

        client.get(PORT, "localhost", "/api/adjective")
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

mvn clean test -Dtest=AdjectiveEndpointTest

```

The test should of course fail.

### Pass our JUnit test

The Vertx Web module makes it easy to build webapps, but we are only implementing a single endpoint so we will stick with basic HTTP functionality.  Open the HttpApplication class and add the following method to handle returning an adjective (we will hard code an adjective for our initial pass):

#### Create a method to handle GET requests

```java

  private void adjectiveHandler(RoutingContext rc) {

    JsonObject response = new JsonObject()
            .put("adjective", "clapper-clawed");

    rc.response()
            .putHeader(CONTENT_TYPE, "application/json; charset=utf-8")
            .end(response.encodePrettily());
  }

```

##### RoutingContext 

Our handler method takes a Vert.x RoutingContext as an argument.  

The RoutingContext encapsulates the context for handling an Http request.  Vert.x creates a new RoutingContext for each HTTP request that is handled by Router#handle(HttpServerRequest) methods and automatically discards it after processing is finished.

The RoutingContext provides access to the HttpServerRequest and HttpServerResponse and allows you to maintain arbitrary data that lives for the lifetime of the context.

##### JsonObject

Vert.x' JsonObject encapsulates the notion of a JSON Object and makes it extremely easy to work with JSON.

#### Add the route

We will add the following route on line 22 just after the existing route for "/api/greeting":

```java

    router.get("/api/adjective").handler(this::adjectiveHandler);

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
    router.get("/api/adjective").handler(this::adjectiveHandler);
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

  private void adjectiveHandler(RoutingContext rc) {

    JsonObject response = new JsonObject()
            .put("adjective", "clapper-clawed");

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

mvn clean test -Dtest=AdjectiveEndpointTest

```

The test should pass, but we aren't actually doing anything.  Let's load the adjectives into memory and return a random one from our handler method.

#### Loading the adjectives

Let's create a method to load the content of the adjectives.txt file:

```java

  private Future<Void> loadAdjectives() {

    if (adjectives == null) {
      adjectives = new ArrayList<>();
    }

    Future<Void> future = Future.future();

    try {
      InputStream is = this.getClass().getClassLoader().getResourceAsStream("adjectives.txt");
      if (is != null) {
        BufferedReader reader = new BufferedReader(new InputStreamReader(is));
        reader.lines()
                .forEach(adj -> adjectives.add(new Adjective(adj.trim())));
      }
      future.complete();
    } catch (Exception e) {
      e.printStackTrace();
      future.fail(e.getCause());
    }

    return future;
  }

```

##### Vertx Futures

Most of this method is pretty standard stuff; however, the use of Vertx Futures is probably new:

```java

  private Future<Void> loadAdjectives() {

    ...

    Future<Void> future = Future.future();

    ...

      future.complete();
    ...

      future.fail(e.getCause());

```

Vertx Futures represent the result of an action that may, or may not, have occurred yet.  They allow us to call multiple methods and do other work while the methods execute.

You probably noticed that we are blocking the event loop while reading the adjectives file.  "Don't block the event loop!" is Vert.x' cardinal rule, but in the case of initializing the app it is okay.  After all if we can't load the adjectives there is no reason for the service to start.

Let's extract the code that creates the HttpServer into its' own method that returns a Future:

```java

  private Future<Void> startHttpServer(){
    Future<Void> future = Future.future();

    // Create a router object.
    Router router = Router.router(vertx);

    router.get("/api/greeting").handler(this::greeting);
    router.get("/api/adjective").handler(this::adjectiveHandler);
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
                  future.complete();
                }else{
                  future.fail(ar.cause());
                }
              });

    return future;
  }

```

This code is almost identical to the previous method except for the addition of a Vert.x Future.

Now we can put our Futures to use.  Change the start method to the following:

```java

  @Override
  public void start(Future<Void> startFuture) {
    Future init = loadAdjectives().compose(v -> startHttpServer()).setHandler(startFuture.completer());
  }

```

In this method we are chaining together multiple Futures with the final result either succeeding and starting the class or failing and preventing it from starting.

1.  We have a Future<Void> method in our start signature.  This will tell our class that it has successfully started or not
2.  We have a Future "init" which combines or "composes" loading the adjectives and starting the HttpServer
3.  As each Future completes it kicks off the next one with its' success or failure.  A single failure will prevent our application from starting


Re-run the test case and verify that it passes.

```bash

mvn clean test -Dtest=AdjectiveEndpointTest

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
