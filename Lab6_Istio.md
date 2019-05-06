# Lab 6 Istio

## Pre-requisites
We will be using those components for following labs


Prerequisites +
* Istio is installed and running on OpenShift +
* Your administrator has assigned you a userid +
* Deployed Noun, adjective and Insult Service

### Enable Istio for the InsultGateway  service



* Step1 - Login to openshift +
```java

oc login https://master.35b7.summit.opentlc.com/login -u $USER_ID -p $PWD

```

* Step 2 - Enable Istio Automatic injection @ Noun Service

You can use any of the noun service i.e SpringBoot, Thorntile, Vert.x or Node.js. In this example , we will be using SpringBoot

```bash

cd insult-solution-noun-service-springboot-master


```
* Step 2.1 Edit insult-solution-noun-service-springboot-master/pom.xml

```xml
<plugin>
  <groupId>io.fabric8</groupId>
  <artifactId>fabric8-maven-plugin</artifactId>
  <version>4.1.0</version>
  <executions>
    <execution>
      <goals>
        <goal>resource</goal>
        <goal>build</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <resources>
      <annotations>
        <deployment>
          <property>
            <name>sidecar.istio.io/inject</name>
            <value>true</value>
          </property>
        </deployment>
      </annotations>
    </resources>
  </configuration>
</plugin>

``` 

Please review the above fabric8-maven-plugin configuration, we are enabling istio side car injection while deploying the application with property sidecar.istio.io/inject

* Step 3 - Deploy Noun service 

``` bash
mvn clean deploy:fabric8 -Popenshift


``` 
Please make sure build is successful
``` bash
 

[INFO] Created Service: target/fabric8/applyJson/user1-insult-app/service-insult-nouns.json
[INFO] Using project: user1-insult-app
[INFO] Creating a DeploymentConfig from openshift.yml namespace user1-insult-app name insult-nouns
[INFO] Created DeploymentConfig: target/fabric8/applyJson/user1-insult-app/deploymentconfig-insult-nouns.json
[INFO] Creating Route user1-insult-app:insult-nouns host: null
[INFO] F8: HINT: Use the command `oc get pods -w` to watch your pods start up
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 02:07 min
[INFO] Finished at: 2019-05-06T07:52:11-04:00
```

Step 4 - Enable Istio Automatic injection @ Adjective Service
You can use any of the adjective service i.e SpringBoot, Thorntile, Vert.x or Node.js. In this example , we will be using SpringBoot

```bash

cd insult-solution-adjective-service-springboot-master


```
* Step 4.1 Edit insult-solution-adjective-service-springboot-master/pom.xml

```xml
<plugin>
  <groupId>io.fabric8</groupId>
  <artifactId>fabric8-maven-plugin</artifactId>
  <version>4.1.0</version>
  <executions>
    <execution>
      <goals>
        <goal>resource</goal>
        <goal>build</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <resources>
      <annotations>
        <deployment>
          <property>
            <name>sidecar.istio.io/inject</name>
            <value>true</value>
          </property>
        </deployment>
      </annotations>
    </resources>
  </configuration>
</plugin>

``` 

Please review the above fabric8-maven-plugin configuration, we are enabling istio side car injection while deploying the application with property sidecar.istio.io/inject

* Step 5 - Deploy Adjective service 

``` bash
mvn clean deploy:fabric8 -Popenshift


``` 
Please make sure build is successful
``` bash
[INFO] F8: Using project: user1-insult-app
[INFO] F8: Creating a Service from openshift.yml namespace user1-insult-app name insult-adjectives
[INFO] F8: Created Service: target/fabric8/applyJson/user1-insult-app/service-insult-adjectives.json
[INFO] F8: Creating a DeploymentConfig from openshift.yml namespace user1-insult-app name insult-adjectives
[INFO] F8: Created DeploymentConfig: target/fabric8/applyJson/user1-insult-app/deploymentconfig-insult-adjectives.json
[INFO] F8: Creating Route user1-insult-app:insult-adjectives host: null
[INFO] F8: HINT: Use the command `oc get pods -w` to watch your pods start up
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:25 min
[INFO] Finished at: 2019-05-06T07:59:58-04:00
[INFO] ------------------------------------------------------------------------


```
Step 6 - Enable Istio Automatic injection @ Insult Service
You can use any of the Insult service i.e SpringBoot, Thorntile, Vert.x or Node.js. In this example , we will be using SpringBoot

```bash

cd insult-solution-insult-service-springboot-master


```
* Step 6.1 Edit insult-solution-insult-service-springboot-master/pom.xml

```xml
<plugin>
  <groupId>io.fabric8</groupId>
  <artifactId>fabric8-maven-plugin</artifactId>
  <version>4.1.0</version>
  <executions>
    <execution>
      <goals>
        <goal>resource</goal>
        <goal>build</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <resources>
      <annotations>
        <deployment>
          <property>
            <name>sidecar.istio.io/inject</name>
            <value>true</value>
          </property>
        </deployment>
      </annotations>
    </resources>
  </configuration>
</plugin>

``` 

Please review the above fabric8-maven-plugin configuration, we are enabling istio side car injection while deploying the application with property sidecar.istio.io/inject

* Step 7 - Deploy Insult service 

``` bash
mvn clean deploy:fabric8 -Popenshift


``` 
Please make sure build is successful

* Step 2 - Deploy gateway.yml to OpenShift +

```bash


```
