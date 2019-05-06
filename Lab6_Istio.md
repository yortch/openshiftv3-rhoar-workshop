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

``` bash
INFO] F8: Using project: user1-insult-app
[INFO] F8: Creating a Service from openshift.yml namespace user1-insult-app name insult-service
[INFO] F8: Created Service: target/fabric8/applyJson/user1-insult-app/service-insult-service.json
[INFO] F8: Creating a DeploymentConfig from openshift.yml namespace user1-insult-app name insult-service
[INFO] F8: Created DeploymentConfig: target/fabric8/applyJson/user1-insult-app/deploymentconfig-insult-service.json
[INFO] F8: Creating Route user1-insult-app:insult-service host: null
[INFO] F8: HINT: Use the command `oc get pods -w` to watch your pods start up
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:26 min
[INFO] Finished at: 2019-05-06T08:06:21-04:00
[INFO] ------------------------------------------------------------------------


```


* Step 8 - Create a Gateway to access your application
In order to make your application accessible from outside the cluster, an Istio Gateway is required. Let us understand gateway and virtual service configurations

```xml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: insult-app-gateway
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "insult-service-user1-insult-app.apps.35b7.summit.opentlc.com"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: insult-app-virtual-service
spec:
  hosts:
  - "insult-service-user1-insult-app.apps.35b7.summit.opentlc.com"
  gateways:
  - insult-app-gateway
  http:
  - match:
    - uri:
        prefix: /api/insult
    rewrite:
      uri: /api/insult
    route:
    - destination:
        port:
          number: 8080
        host: insult-service
  - match:
    - uri:
        prefix: /api/noun
    rewrite:
      uri: /api/noun
    route:
    - destination:
        port:
          number: 8080
        host: insult-nouns
  - match:
    - uri:
        prefix: /api/adjective
    rewrite:
      uri: /api/adjective
    route:
    - destination:
        port:
          number: 8080
        host: insult-adjectives
```
### Gateway: A Gateway configures a load balancer for HTTP/TCP traffic, most commonly operating at the edge of the mesh to enable ingress traffic for an application. The above gateway will direct all the HTTP traffic coming on port 80 at istio-ingressgateway to the insult application.

The selector istio: ingressgateway pull the traffic coming to istio-ingressgateway service in the istio-system project
The parameter hosts:  says that  traffic coming to this insult-app-gateway for any hostname will be consumed. If we want our application to cater to specific hostnames, we should list those here instead of using *

###VirtualService: A VirtualService defines the rules that control how requests for a service are routed within an Istio service mesh. With the above virtualservice configuration:

gateways: - insult-app-gateway configures it to listens to traffic coming to insult-app-gateway defined earlier
host: "*" caters to any hostnames. If we want specific hostname, we can change this to a specific hostname.
URI matching allows it to listen to /insult etc.
