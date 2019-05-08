# Lab-9-Thorntail-Twitter_Service.md

## Create a Project for the Thorntail Twitter Service  
git clone https://github.com/jeremyrdavis/insult-starter-thorntail

## Create a Project for the Spring Boot Twitter Service  

###  Clone the repository 

If you've come this far you've probably figured out the first step by now

```bash

git clone https://github.com/jeremyrdavis/insult-starter-thorntail

```

### Or download the project zip file

You can download the zip file from Github by opening https://github.com/jeremyrdavis/insult-starter-thorntail
and choosing, "Download ZIP" from the green, "Clone or Download" button

![](./images/lab7/lab7-sb-01-download.png)  

### Import the app into VS Code

Open Visual Studio Code, choose "Open," and navigate to the root folder of the project

### Update the project settings

We need to update our project's settings from the default starter app to the adjective service we are building.

Open the pom.xml file and change the artifactId and name to 
"insult-tweets," and "Thorntail Insult Twitter Service":

```xml

  <artifactId>insults-twitter</artifactId>
  <version>1.0.0</version>
  
  ...
  
  <name>Thorntail Insult Twitter Service</name>

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

#### Log in to OpenShift

You may still be logged into OpenShift.  You can check by running the following command:

```bash

oc whoami

```

If the response is your username then you are still logged in.  If you are still logged in you can skip the next step.

##### Log back in to OpenShift

Fabric8 will build a Docker container and deploy it to OpenShift for us, but we need to be logged in first.  From your OpenShift console copy the login command by clicking on your name in the top right and choosing, "Copy Login Command."

![](./images/4-1/04-copy_login_command.png)  

Paste and enter the command into your terminal

![](./images/4-1/vscode-03-login.png)  


#### Build and deploy to OpenShift

Now we can deploy our app.  From the terminal run the following maven command:

```bash

mvn clean fabric8:deploy -Popenshift -DskipTests 

```

We have already run the tests so we will skip them to speed up the build

### Validating the deployment:  

1. Login to OpenShift Console - with your user name and password
2. Click on Project ‘red-hat-summit-2019’ if you are not already in that project
3. You should see 1 running pod and a url that you can access
4. Try the url


![](./images/4-1/05-initial_deploy.png)  


You should see:


![](./images/4-1/06-greeting_service.png)  


## Get coding!

### Create and fail a JUnit Test for our endpoint

We are of course practicing TDD in this tutorial so our first step will be to write a Unit Test.  Create a new class, "TwitterResourceTest.java," in the "src/test/java/com/redhat/summit2019" directory with the following content:

```java

package com.redhat.summit2019;

import org.jboss.arquillian.container.test.api.RunAsClient;
import org.jboss.arquillian.junit.Arquillian;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.wildfly.swarm.arquillian.DefaultDeployment;

import static org.junit.Assume.assumeNoException;

import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.client.Entity;
import javax.ws.rs.client.WebTarget;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;

import com.redhat.summit2019.model.Insult;

@RunWith(Arquillian.class)
@DefaultDeployment
public class TwitterResourceTest {

    @Test
    @RunAsClient
    public void serviceInvocation() {
        Client client = ClientBuilder.newClient();
        WebTarget target = client.target("http://localhost:8080")
                .path("api").path("tweet");

        Insult insult = new Insult("Verily, ye be a goatish, mangled miscreant!");

        Response response = target.request(MediaType.APPLICATION_JSON).post(Entity.entity(insult, MediaType.APPLICATION_JSON));

        Assert.assertEquals(200, response.getStatus());
        Assert.assertNotNull(response.readEntity(String.class));
    }

}

```

Run the test to verify that it fails:

```bash

mvn clean test

```

Run the test to verify that it fails:

```bash

mvn clean test

```

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

### A domain model

Our service will be receiving a single insult so we only need to model that object.  Create a new package "com.redhat.summit2019.model," ("src/main/java/com/redhat/summit2019/model.")  Create new class "Insult.java," in the package with the following content:

```java

package com.redhat.summit2019.model;

import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;

public class Insult {

    String insult;

    @JsonCreator
    public Insult(@JsonProperty("id") String insult) {
        this.insult = insult;
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

    public String getInsult() {
        return insult;
    }

    public void setInsult(String insult) {
        this.insult = insult;
    }
}


```

You may not recognize the, "@JsonCreator" annotation on the constructor.  We need this to properly unmarshall the Json payload from our endpoint.

### A REST Endpoint

Create a new class, "TwitterResource" in the "com.redhat.summit2019.services" package ("src/main/java/com/redhat/summit2019/services/TwitterResource.java") with the following code:

```java

package com.redhat.summit2019;

import twitter4j.Status;
import twitter4j.StatusUpdate;
import twitter4j.Twitter;
import twitter4j.TwitterFactory;

import javax.ws.rs.Consumes;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.Response;
import javax.ws.rs.core.MediaType;

import com.redhat.summit2019.model.Insult;

import java.time.Instant;
import java.util.Date;

@Path("/")
public class TwitterResource {

    @POST
    @Path("/tweet")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    public Response tweet(Insult insult) {

        Response.ResponseBuilder builder = null;

        try{
            Twitter twitter = TwitterFactory.getSingleton();
            StringBuilder stringBuilder = new StringBuilder()
                    .append(insult.getInsult())
                    .append(" - Generated by Thorntail with the Red Hat Summit Shakespearean Insult Workshop at ")
                    .append(Date.from(Instant.now()).getTime());
            StatusUpdate statusUpdate = new StatusUpdate(stringBuilder.toString());
            Status status = twitter.updateStatus(statusUpdate);
            System.out.println(status.toString());
            builder = Response.ok().entity(statusUpdate);
        }catch(Exception e){
            System.out.println(e.getMessage());
            builder = Response.status(Response.Status.BAD_REQUEST).entity(e);
    }
    return builder.build();
    }
}

```
The Twitter4j code is quite simple:

```java

      Twitter twitter = TwitterFactory.getSingleton();

      ...

      StatusUpdate statusUpdate = new StatusUpdate(stringBuilder.toString());
      Status status = twitter.updateStatus(statusUpdate);

```

Twitter4j provides a "Twitter" object that we retrieve as a Singleton from a "TwitterFactory."  The Twitter ojbect will handle all of the communication with Twitter's API's.

Sending a Tweet is "StatusUpdate" in Twitter terminology.  We are building our Tweet or StatusUpdate with a StringBuilder and then passing it to the StatusUpdate constructor:

```java


      StringBuilder stringBuilder = new StringBuilder()
              .append(insult.getInsult())
              .append(" - Generated by SpringBoot in the Red Hat Summit Shakespearean Insult Workshop at ")
              .append(Date.from(Instant.now()).getTime());
      StatusUpdate statusUpdate = new StatusUpdate(stringBuilder.toString());

```

We then send our StatusUpdate object to the Twitter object's "updateStatus" method:

```java

    Status status = twitter.updateStatus(statusUpdate);

```

The returned Status object is represented in JSON as:

```json

{
    "status": "Verily, ye be a droning, roguish clack-dish! - Generated by Spring Boot with the Red Hat Summit Shakespearean Insult Workshop at 1557223114958",
    "inReplyToStatusId": -1,
    "location": null,
    "placeId": null,
    "displayCoordinates": true,
    "possiblySensitive": false,
    "autoPopulateReplyMetadata": false,
    "attachmentUrl": null
}

```

Let's run our test again, which should now pass:

```bash

mvn clean test -Dtest=TwitterResourceTest

```
