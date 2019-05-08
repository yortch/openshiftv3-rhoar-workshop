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

IMPORTANT: We need to delete the configuration from the fabric8 settings.  Replace the "configuration" section of the fabric8 plugin with an empty tag so that it mirrors the following:


```xml

          <plugin>
            <groupId>io.fabric8</groupId>
            <artifactId>fabric8-maven-plugin</artifactId>
            <version>${fabric8-maven-plugin.version}</version>
            <executions>
              <execution>
                <id>fmp</id>
                <goals>
                  <goal>resource</goal>
                  <goal>build</goal>
                </goals>
              </execution>
            </executions>
            <configuration/>
          </plugin>


```

This is to address a bug discovered this morning and will be addressed in the near future

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

### Create and fail a JUnit Test for our endpoint

We are of course practicing TDD in this tutorial so our first step will be to write a Unit Test.  Create a new class, "TwitterVerticleTest.java," in the "src/test/java/com/redhat/summit2019" directory with the following content:


```java

package com.redhat.summit2019;

import static org.junit.Assert.assertNotNull;
import static org.junit.Assert.assertThat;
import static org.junit.Assert.assertTrue;

import java.time.Instant;
import java.util.Date;

import org.junit.Test;
import org.junit.Before;
import org.junit.After;
import org.junit.runner.RunWith;

import io.vertx.core.Vertx;
import io.vertx.core.eventbus.DeliveryOptions;
import io.vertx.core.json.Json;
import io.vertx.core.json.JsonObject;
import io.vertx.ext.unit.Async;
import io.vertx.ext.unit.TestContext;
import io.vertx.ext.unit.junit.VertxUnitRunner;

@RunWith(VertxUnitRunner.class)
public class TwitterVerticleTest{

    private Vertx vertx;

    @Before
    public void before(TestContext context) {
        vertx = Vertx.vertx();
        vertx.exceptionHandler(context.exceptionHandler());
        vertx.deployVerticle(TwitterVerticle.class.getName(),
        context.asyncAssertSuccess());
    }

    @After
    public void after(TestContext context) {
        vertx.close(context.asyncAssertSuccess());
    }
    @Test(timeout = 1000L)
    public void testSendingATweet(TestContext tc) {

        Async async = tc.async();

        String replyText = new StringBuilder()
        .append(" Verily, ye be a crook-pated, paunchy hedge-pig! ")
        .append(Date.from(Instant.now())).toString();
  
        JsonObject message = new JsonObject()
            .put(TwitterVerticle.MESSAGE_KEY, new JsonObject()
            .put(TwitterVerticle.ACTION, TwitterVerticle.ACTIONS_STATUS_UPDATE)
            .put(TwitterVerticle.PARAMETERS_STATUS, replyText));
            System.out.println("testSendingATweet: " + Json.encodePrettily(message));

              DeliveryOptions deliveryOptions = new DeliveryOptions()
                .setSendTimeout(600000);
              vertx.<JsonObject>eventBus().send( TwitterVerticle.ADDRESS, message, deliveryOptions, ar -> {
                if (ar.succeeded()) {
                  System.out.println(ar.result().toString());
                  tc.assertNotNull(ar.result().body());
                  tc.assertTrue(ar.result().body().toString().contains(TwitterVerticle.RESULT_SUCCESS));
                  async.complete();
                }else{
                    System.out.println(ar.result().toString());
                    tc.assertTrue(ar.succeeded());
                    async.complete();
                }
              });
        
        
          }

}
```

There is no need to run the test to verify that it fails because it won't compile yet.

### Add Twitter4j dependenices

Twitter has well documented API's for interacting with its' features, https://developer.twitter.com/content/developer-twitter/en.html.  If you are developing marketing applications for Twitter you will want to check them out.  

For our purposes the excellent Twitter4j project has all of the functionality we need, http://twitter4j.org/en/index.html.  We will use Twitter4j to tweet our insults.

Add the Twitter4j dependencies.  First, add the version in the pom's properties section:

```xml

    <twitter4j.version>[4.0,)</twitter4j.version>

```

Then add the dependency to the dependencies section:

```xml

    <dependency>
      <groupId>org.twitter4j</groupId>
      <artifactId>twitter4j-core</artifactId>
      <version>${twitter4j.version}</version>
    </dependency>

```

#### Add our Twitter properties

Create a file "twitter4j.properties" in src/main/resources:

```yaml

debug=true
oauth.consumerKey=YOUR_KEY
oauth.consumerSecret=YOUR_CONSUMER_SECRET
oauth.accessToken=YOUR_ACCESS_TOKEN
oauth.accessTokenSecret=YOUR_ACCESS_TOKEN_SECRET

```

You will need to add your accounts' settings here.  For the sake of this lab you can use the following:

```yaml


```

Twitter4j will automatically pick up these properties!

### An EventBus Verticle

We will create a new Verticle to handle sending Tweets:

```java

package com.redhat.summit2019;

import io.vertx.core.AbstractVerticle;
import io.vertx.core.Future;
import io.vertx.core.eventbus.EventBus;
import io.vertx.core.eventbus.Message;
import io.vertx.core.eventbus.MessageConsumer;
import io.vertx.core.json.JsonObject;
import twitter4j.Status;
import twitter4j.StatusUpdate;
import twitter4j.Twitter;
import twitter4j.TwitterException;
import twitter4j.TwitterFactory;

public class TwitterVerticle extends AbstractVerticle {

  Twitter twitter;

  public static final String ADDRESS = "tweetbus";
  public static final String ACTION = "action";
  public static final String ACTIONS_STATUS_UPDATE = "status_update";
  public static final String ACTIONS_REPLY = "reply";
  public static final String PARAMETERS_REPLY_TO_STATUS_ID = "reply_to_status_id";
  public static final String PARAMETERS_REPLY_TO_SCREEN_NAME = "screen_name";
  public static final String PARAMETERS_STATUS = "status";
  public static final String RESULT_FAILURE = "failure";
  public static final String RESULT_FAILURE_MESSAGE = "error_message";
  public static final String RESULT_SUCCESS = "success";
  public static final String PARAMETERS_REPLY_STATUS = "reply_message";
  public static final String MESSAGE_KEY = "message";

  public enum EventBusErrors {

    UNKNOWN_ADDRESS(1, "unrecognized address");

    public final int errorCode;

    public final String errorMessage;

    private EventBusErrors(int errorCodeToSet, String errorMessageToSet){
      this.errorCode = errorCodeToSet;
      this.errorMessage = errorMessageToSet;
    }
  }

  @Override
  public void start(Future<Void> startFuture) {

    twitter = TwitterFactory.getSingleton();
    EventBus eventBus = vertx.eventBus();
    MessageConsumer<JsonObject> consumer = eventBus.consumer(ADDRESS);

    consumer.handler(message -> {

      String action = message.body().getJsonObject(MESSAGE_KEY).getString(ACTION);

      switch (action) {
        case ACTIONS_STATUS_UPDATE:
          tweet(message);
          break;
        default:
          message.fail(EventBusErrors.UNKNOWN_ADDRESS.errorCode, EventBusErrors.UNKNOWN_ADDRESS.errorMessage);
      }
    });

    startFuture.complete();

  }

  private void tweet(Message<JsonObject> message) {
    System.out.println("TwitterVerticle.tweet: " + message.body());
    JsonObject replyJson = message.body().getJsonObject(MESSAGE_KEY);

    StatusUpdate statusUpdate = new StatusUpdate(replyJson.getString(PARAMETERS_STATUS));

    System.out.println(statusUpdate.toString());

    vertx.executeBlocking((Future<Object> future) -> {
      try {
        Status status = twitter.updateStatus(statusUpdate);
        System.out.println(status.toString());
        future.complete();
      } catch (TwitterException e) {
        e.printStackTrace();
        future.complete(e.getMessage());
      }
    }, res -> {
      if (res.succeeded()) {
        message.reply(new JsonObject().put("outcome", RESULT_SUCCESS));
      } else {
        message.reply(new JsonObject().put("outcome", RESULT_FAILURE).put(RESULT_FAILURE_MESSAGE, res.cause()));
      }
    });
  }

}

```

Run the test:

```bash

mvn clean test -Dtest=TwitterVerticleTest

```


Update HttpApplication with a new method to receive JSON and call the EventBus:

```java

  private void tweetHandler(RoutingContext routingContext) {

    System.out.println("tweetHandler");

    JsonObject requestJson = routingContext.getBodyAsJson();

    System.out.println("request payload:\n" + requestJson);

    String tweetText = new StringBuilder()
      .append(requestJson.getString("text"))
      .append(" - Generated by the Red Hat Summit Shakespearean Insult Workshop at ")
      .append(Date.from(Instant.now()).getTime()).toString();

    System.out.println("status:\n" + tweetText);

    JsonObject message = new JsonObject()
      .put(TwitterVerticle.MESSAGE_KEY, new JsonObject()
        .put(TwitterVerticle.ACTION, TwitterVerticle.ACTIONS_STATUS_UPDATE)
        .put(TwitterVerticle.PARAMETERS_STATUS, tweetText));

    System.out.println("message:\n" + message);

    vertx.<JsonObject>eventBus().send(TwitterVerticle.ADDRESS, message, ar -> {
      if (ar.succeeded()) {
        HttpServerResponse response = routingContext.response();
        response
          .putHeader("Content-Type", "application/json")
          .end(Json.encodePrettily(new JsonObject().put("outcome", "success").put("status", tweetText)));
      } else {
        HttpServerResponse response = routingContext.response();
        response
          .putHeader("Content-Type", "application/json")
          .end(new JsonObject().put("error", ar.cause().getMessage()).toBuffer());
      }
    });
  }

```

and add the URL:

```java

    Router apiRouter = Router.router(vertx);
    apiRouter.get("/api/greeting").handler(this::greeting);
    apiRouter.post("/api/tweet").handler(this::tweetHandler);
    apiRouter.get("/*").handler(StaticHandler.create());

```