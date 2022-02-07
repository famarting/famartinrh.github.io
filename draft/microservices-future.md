# Microservies Techniques

This document aims to show off some of the diferent ways that can be used to develop microservice based applications.

## "Classic" Microservices

The beginning of this architectural pattern, simply writing separate and small applications
that communicate between each other using synchronous communication via REST APIs.

For Java developers the tipical technologies used are Spring Boot or any other modern framework, altough much younger and modern Quakus can fit here. For other languages there are plenty of variety, NodeJS is kind of a thing in this area...

## Service Mesh

Tipicaly you will start doing "classic" microservices and your system will reach to a point where 
the service mesh it's pretty much a must. Service Mesh frameworks like Istio help manage microservices based applications with hundreds or thousands of microservices.

## Event Driven Architectures

Beleive it or not event driven architectures fit pretty well with microservices based applications.
Among other things event driven means introducing asynchronous communication between your microservices via message passing through queues and topics provided by messaging brokers. Here projects like Kafka and strimzi or enmasse in the AMQP world play a very important role providing and managing the messaging infrastructure.

### Event Driven Serverless

Projects like knative allows to run serverless loads in kubernetes cluster, with knative it's possible to trigger our serverless applications in response to http requests or with new kafka messages produced to some topic... This is thanks to knative eventing which enables to develop event driven applications based on serverless to run on kubernetes.

## Multi-Runtime Microservices Architecture

Inspired by this [article](https://www.infoq.com/articles/multi-runtime-microservice-architecture/)
there could be a new trend where, once Service Mesh and Serverless is spread, the infrastructure have new capabilities so the developers will care about writting business logic for the microservices and wire up, via the infrastructure used, all the diferent components that compose their systems.

## Example Applications

### Classis Microservices
//TODO microprofile-conference???

### Service Mesh with Istio
//TODO ???

### Event Driven Serverless
//TODO ???

### Event Driven with Enmasse + Quarkus
[link to github repository](https://github.com/famartinrh/enmasse-quarkus-demo)

### Event Driven with kafka streams and strimzi -- TBD 
???

### Multi-Runtime Microservices with Dapr
//TODO ???

### Multi-Runtime Microservices with camel-k
//TODO ???





