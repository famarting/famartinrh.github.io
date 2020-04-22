---
layout: post
title:  "Event-Driven microservices with Enmasse"
subtitle: "The joy of doing Event-Driven Microservices using Enmasse and Quarkus"
description: >
  The purpose of this blog is to talk about Event-Driven based microservices and explore it's combination with Enmasse, a Kubernetes operator to manage AMQP based messaging infrastructure. Because of microservices Quarkus, container-first Java stack, is also showcased.
  <br>
  This blog is for people in Kubernetes world, software developers interested in the full picture of applications and systems or any other interested.
date:   2020-04-22
categories: [blog]
excerpt_separator: <!--more-->
---

The Event-Driven Architecture can be described as an architectural pattern for designing software systems where the different components of the system send and receive events in real time. With this architecture the way the system components operate is by producing events or reacting in response to events that are sent or received through channels.

<!--more-->

In this architecture the events are the representation of a meaninfull change in the system, like "a new user has been registered" or "a new order has been placed". Usually event's payload contain meaninfull information related to the entity the event is related to. Events can be classified in three types: Volatile, Durable and Replayable.
- Volatile events are those that are not stored by the messaging middelware and are lost if there is not connected consumer in the moment they are produced.
- Durable events, the most typical and common, here events are stored durably until read by all registered consumers.
- Replayable events, similar to durable events, but in this case the events are kept stored for a determined period of time and there is the possibility for consumers to replay the stored sequence of events.

As described, events are sent through channels, by channels I mean the piece of middelware we use to send our events through. Here I'm talking about queues and topics implemented by messaging brokers. There are a big variety of messaging brokers each one providing it's own set of features and behaviours and in many cases different protocols to communicate with the broker. This is important because this makes all the different event types that we know or the different eventing models or messaging patterns that our applications can make use of. In simple words, the messaging broker we choose determines the kind of events we can use in our application, therefore we should choose in concordance with our application needs.

If talking about microservices, Event-Driven Architectures can bring some advantages such as:
- decoupling microservices, separating and abstracting between each other, this is the separation of producers of events and consumers of events. As an example, a user service will produce an event when a new user is created and other microservices in the application, doesn't matter which ones, will consume that event and do whatever action they need to do in response to that.
- better scalability, there is nothing new in buffering high loads using messaging queues and Event-Driven of course allows that, apart from that, going Event-Driven should make the scalability process much straightforward as your new microservices instances spawn up the messaging infrastructure will spread the load between the consumers.
- ease of system extensibility, that's it, in the example above including new behaviours when a user is created is straightforward, some microservice will have to listen for the user created event and then do the appropiate logic.

With that being said looks like an Event-Diven Architecture is a killer approach for microservices applications, allowing the application to scale and helping to improve maintainability by decoupling microservices and easing extensibility. But with great power comes great responsibility and for Event-Driven it's not less, therefore this kind of architecture brings not an easy task that adds to the already large pile of microservices architecture downsides. I'm talking about providing, managing and maintaining the messaging infrastructure required by the new architecture, and not only that, just imagine enterprises with big event-driven microservices applications, they need HA deployments with failure recovery or increased security measures...

# Messaging Infrastructure

About messaging infrastructure, if you are doing microservices you are likely using Kubernetes, so you may want to run your messaging infrastructure inside Kubernetes as well, and if the installation and management of that infrastructure can be automated using Kubernetes operators that's even better. There are two options here that I would like to remark, each one with it's own specific characteristics making a difference in the kind of events that our application will be able to use.

You probabily have heard of [Apache Kafka](https://kafka.apache.org/), a high performing commit log based messaging broker that use it's own protocol and set of clients. Very popular nowadays and a solid choice for most event driven applications. To highlight, it allows for replayable events thanks to it's commit log based implementation. For the Kubernetes world there is [Strimzi] an open-source Kubernetes operator to deploy and manage kafka brokers installations.

Other option I would like to highlight is going for an AMQP based broker, [AMQP](https://www.amqp.org/) is a standarized and battle tested protocol for messaging. There are plenty of options here but if we want a solution inside the Kubernetes world a good choice is [Enmasse], claimed as self-service messaging on Kubernetes. Enmasse is an open-source Kubernetes operator to deploy and manage AMQP based messaging infrastructure.Enmasse is based in other open-source projects such as [Apache ActiveMQ](https://activemq.apache.org/), [Apache Qpid Dispatch Router](https://qpid.apache.org/components/dispatch-router/index.html)... Enmasse provides a series of abstractions that allow developers to carelessly request messaging resources for their applications and at the same time allowing admins to manage resource limits. As per an Event-Driven architecture, and opposite to Strimzi, Enmasse does not support replayable events but supports volatile and durable event types.

As stated in the title of this blog, the purpose here is to explore Enmasse so I'm going to focus on showcase some of it's functionalities as well as show that it is useful as the messaging infrastructure provider for an Event-Driven based microservices architecture.

## Enmasse

![Enmasse logo](https://raw.githubusercontent.com/EnMasseProject/enmasse/master/documentation/_images/logo/enmasse_logo.png)

Enmasse has different concepts that is important to understand. I'm not going to do a deep explanation of Enmasse concepts, that can be found in the [Enmasse documentation]. Here I just want to highligh some basic ideas so we are all on the same page.

To start easy just think Enmasse users can have two different roles, service admin and messaging tenant. Service admins use some APIs to configure the operator and predefine infrastructure configuration as well as resource limits for the usage of the infrastructure. In the other hand, messaging tenants can request messaging resources which will use some of the plans created by service admins. This way admins can configure the limits of the infrastructure and tenants can safely create it's queues and topics. I like to think of this like, ops people configuring the Kubernetes cluster and developers requesting queues and topics for it's applications.

Focusing on the messaging tenant side, in the current Enmasse implementation there are three APIs available: AddressSpace, Addresse and MessagingUser. Ideally developers will only need to take care of requesting Addresses, and that may be the case in near future Enmasse implementation.

The AddressSpace concept is the starting point for messaging tenants when requesting resources, one AddressSpace holds a collection of Addresses. There are two types of AddressSpaces: standard and brokered. Messaging infrastructure is deployed differently for each AddressSpace type, standard type relies in a topology based of a router mesh in front of a series of AMQP brokers and brokered type just deploys AMQP brokers. We will be focusing on standard AddressSpace type. It's important to highlight that, among other limitations, standard AddressSpace type doesn't guarantee message ordering, more info can be found in [Enmasse documentation].

The concept of Address represents, as described earlier in this blog, a channel. Addresses have a type that determines the semantics it will have once deployed. The possible Address types, for the standard AddressSpace type, are: anycast, multicast, queue, topic and subscription. Anycast and Multicast addresses allow for direct communication without store-and-forward, both producer and consumer/s have to be connected at the moment of transmitting the message. Queue and Topic address types allow for store-and-forward messaging pattern, so consumer doesn't need to be connected when the producer sends the message. Subscription is kind of a subtype of Topic address, this type won't be covered in this blog. Related to Event-Driven this address types map to event types as follow, anycast and multicast allow to implement the volatile event type and queue and topic implement the durable event type. 

MessagingUser is the last API I would like to highlight. Enmasse integrates authentication and authorization capabilities for the usage of the Addresses, this is configured through MessagingUser CRD. MessagingUser determines the credentials which will be used by our applications to authenticate to the AMQP endpoint as well as what operations can our applications do in certain addresses.

# Learning by example

Let's continue learning about Enmasse by showing it's usage in an example microservices application, take a look at the high level diagram of a completely non realistic application for the management of a warehouse. Disclaimer, please take this as an example with demo purposes just to write some microservices using the different functionalities of Enmasse.

![High level diagram](/assets/images/diagram.png)

In the image above we can see the different components of the example application. 
Quickly described: A REST API receives orders requesting some stock for one item in the warehouse, each order is enqueued in a "orders" queue, after that orders-processor receives them one by one and sends a request to the stock-service, which stores in a database the stock values, requesting the appropiate stock for a particular item. Then stock-service replies to orders-processor which receives the result of the stock request and at that point orders-processor consider this order as processed, generating an event in "processed-orders" topic allowing for other systems to plug in and act in response. This has been a pretty quick and high level description of what the example application do.

## What is used?

Following I will show and describe the different resources used to build the example application.

### Enmasse side

There is a queue created for buffering the load between the rest api and the orders-processor, this is the Enmasse Address definition used:
{% highlight yml %}
apiVersion: enmasse.io/v1beta1
kind: Address
metadata:
  name: orders
spec:
  address: orders
  type: queue
  plan: standard-medium-queue
{% endhighlight %}

For the output of orders-processor there is a topic created.
{% highlight yml %}
apiVersion: enmasse.io/v1beta1
kind: Address
metadata:
  name: processed-orders
spec:
  address: processed-orders
  type: topic
  plan: standard-small-topic
{% endhighlight %}

As shown in the first diagram, request-response pattern is used between orders-processor and stock-service. As most microservices architectures do, this request-response can be implemented with a http request from orders-processor to stock-service, but because we are playing with Enmasse we will use an Anycast Address which allows to implement the request-response pattern using our AMQP infrastructure.

{% highlight yml %}
apiVersion: enmasse.io/v1beta1
kind: Address
metadata:
  name: stocks
spec:
  address: stocks
  type: anycast
  plan: standard-medium-anycast
{% endhighlight %}

However the communication between orders-processor and stock-service can be implemented in several ways, in a more Event-Driven manner we can get rid of the direct communication between these two services by using more queues, leaving us with a flow as follows.

![High level diagram, all queues](/assets/images/diagram-all-queues.png)

In addition to the addresses shown, my [implementation] adds a Multicast Address used as a common logs channel, I implemented all my components to send some logging information to a Multicast address so I can follow in real time the status of the system by consuming messages from that address. 

{% highlight yml %}
apiVersion: enmasse.io/v1beta1
kind: Address
metadata:
  name: events
spec:
  address: events
  type: multicast
  plan: standard-medium-multicast
{% endhighlight %}

Multicast and Anycast address, because of it's volatile nature, are really useful for some usecases where the information sent it's not critical and the amount of messages sent over time is quite large. A common example is telemetry data in IoT usecases, which btw is also covered by Enmasse (check the [iot documentation]), because of the volume of telemetry data can have over time you may not want to send it through queues or topics which will store and forward the messages to the consumers, this is expensive, due to the data it's not critical you may prefer just send it to the consumer or throw it if there isn't one available, the consumer will receive the info in the next telemetry loop iteration.

Something important not mentioned yet about Enmasse configuration for this microservices example is the AddressSpace resource, it is neccesary to hold all the addresses that we want to create.

{% highlight yml %}
apiVersion: enmasse.io/v1beta1
kind: AddressSpace
metadata:
  name: warehouse-address-space
spec:
  type: standard
  plan: standard-medium
  endpoints:
  - name: messaging
    service: messaging
    expose:
      type: route
      routeServicePort: amqps
      routeTlsTermination: passthrough
    exports:
    - name: messaging-config
      kind: ConfigMap
{% endhighlight %}

Notice in the AddressSpace example yaml the "exports" part, this is really useful for easily configure the connection details of our microservices. With that setting Enmasse will create a ConfigMap called "messaging-config" which we can use in our microservices deployments to pick the AMQP endpoint host and port values so our clients know where they have to connect to. Following, there is an example of a deployment taking the values from the ConfigMap and setting those as environment variables.

{% highlight yml %}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: stocks-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: stocks-service
  template:yml
        app: stocks-service
    spec:
      containers:
      - image: quay.io/famargon/stocks-service:latest
        env:
          - name: amqp-host
            valueFrom:
              configMapKeyRef:
                key: service.host
                name: messaging-config
          - name: amqp-port
            valueFrom:
              configMapKeyRef:
                key: service.port.amqp
                name: messaging-config
          - name: amqp-username
            value: demo-user
          - name: amqp-password
            value: demo-user
{% endhighlight %}

### Microservices side

In my example [implementation] all the microservices are implemented in Java using [Quarkus]. Why Quarkus? Mainly because it's cool and fun to play with but actually Quarkus has some benefits that come quite handy for the implementation of this example application. A really good description of what quarkus is can be found [here](https://www.redhat.com/en/topics/cloud-native-apps/what-is-quarkus).

Quarkus is based on [Vert.x](https://vertx.io/) and then we can easily use Vert.x AMQP client which is one of the best clients for AMQP in the Java world and the good news don't finish here, we can use [Eclipse Microprofile Reactive Messaging] with Quarkus which gives us a more simple API and allows for an annotation based programming model.

```java
    @Inject
    @Channel("processed-orders")
    Emitter<JsonObject>  processedOrders;

    @Incoming("orders")
    public CompletionStage<Void> processOrder(JsonObject order) {
        return stocks.requestStock(order.getString("order-id"), order.getString("item-id"), order.getInteger("quantity"))
            .thenAccept(result -> {
                JsonObject processedResult = processResult(order, result);
                processedOrders.send(processedResult);
            });
    }
```

The possibility of using Microprofile Reactive Messaging is a really good addition as it allows to abstract the application from the protocol used. So in the future moving to a different messaging infrastructure, based on other protocols, won't require a massive rewrite of the application. 

However nothing is perfect and solutions such as Microprofile Reactive Messaging are not less, you probably will always face situations where all the functionalities of the AMQP protocol, or whatever other protocol you are using with Microprofile Reactive Messaging, are needed in your applications, in those cases you will likely use Vert.x AMQP client to take advantage of all of it's functionalities.

# Conclusions

Enmasse is a useful project for enterprise level management of AMQP based messaging infrastructure and used to back an Event-Driven microservices architecture provides a safe base to build microservices applications, all of this combined with Quarkus microservices with the flexibility to connect with the AMQP infrastructure and all the benefits Quarkus brings to microservices by itself make a really powerfull recipe to do microservices in Kubernetes.


[Quarkus]: <https://quarkus.io/>
[Enmasse]: <https://enmasse.io/>
[Strimzi]: <https://strimzi.io/>
[here]: <https://github.com/famartinrh/enmasse-quarkus-warehouse-demo>
[implementation]: <https://github.com/famartinrh/enmasse-quarkus-warehouse-demo>
[iot documentation]: <https://enmasse.io/documentation/master/openshift/#iot-guide-messaging-iot>
[Enmasse documentation]: <https://enmasse.io/documentation/master/openshift/>
[Eclipse Microprofile Reactive Messaging]: <https://github.com/eclipse/microprofile-reactive-messaging>