### Traffic Shifting - AB testing

In this lab, we will learn to shift specific amount of traffic to specific version of a service not by changing code but just by using routing rules. Note that this feature is available in OpenShift router by default. However, Istio enables this for services that may not not have been exposed via the router.

Pre-Requisites
A running Istio Cluster
Insult Application deployed
DestinationRules applied


### Traffic to V1

Route all the traffic is routed to version 1 by default as before by running

create a file called route-all-traffic-v1.yml in the location of choice.

``` xml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: insult-service
spec:
  hosts:
  - insult-service
  http:
  - route:
    - destination:
        host: insult-service
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: insult-nouns
spec:
  hosts:
  - insult-nouns
  http:
  - route:
    - destination:
        host: insult-nouns
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: insult-adjectives
spec:
  hosts:
  - details
  http:
  - route:
    - destination:
        host: insult-adjectives
        subset: v1
---


```
On the Kiali Service Graph, click on Graph Settings dropdown and select the check box to show Request Percent of Total. This will be useful for us to measure the amount of traffic flowing to specific version.

Use the application a few times to generate traffic. Also check the Kiali service graph. You will notice that all the traffic is going to insult noun version v1 as below.



![](./images/kaili-all-traffic.png)

#### AB Testing with 50% Split between Noun service V1 and V2


Now, we will replace the default routing rule for nouns  virtual service to share the traffic across versions reviews v1 and v2

see the description of the virtual service on how it distributes the traffic between versions v1 and v2 50% each.

create a file named virtual-service-nouns-50.yml in the location of your choice

```xml

apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: insult-nouns
spec:
  hosts:
    - insult-nouns
  http:
  - route:
    - destination:
        host: insult-nouns
        subset: v1
      weight: 50
    - destination:
        host: insult-nouns
        subset: v2
      weight: 50


```
Now let's apply this routing rule by running

```bash
Rams-MacBook-Pro-2:resources rmaddali$ oc apply -f virtual-service-nouns-50-v4.yml
virtualservice.networking.istio.io "insult-nouns" configured

```
