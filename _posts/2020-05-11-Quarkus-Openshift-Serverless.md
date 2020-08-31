---
title: "Quarkus, a lightweight Java for Serverless"
tags:
  - redhat
  - openshift
  - serverless
  - quarkus
  - java
  - event-driven 
  - knative
permalink: /:year/:month/:day/quarkus-openshift-serverless
---
Java has been around for more than 20 years and is still popular among developers today. However, with the digital landscape moving towards cloud, Java
is often seems as an unfavourable option because of its heavy-duty performance.

![Quarkus Logo](https://user-images.githubusercontent.com/25560159/81423897-ad295100-9187-11ea-8baf-94e0e232fb88.png)

However, fear not. Quarkus is here to bring Java into the cloud-native world. Quarkus is an open source, full stack, Kubernetes-native Java framework which optimizes Java for
the cloud future. Quarkus promises:

 1. Fast startup (~0.055 Seconds for REST + CRUD)
 2. Low memory (~35 MB for REST + CRUD)
 3. Small footprint (higher density)

![Quarkus_metrics from http://quarkus.io/](https://user-images.githubusercontent.com/25560159/81463365-b3511900-91eb-11ea-9d14-f549c45b1a8f.png)
  
Hence it is very suitable for containers and serverless. For instance, when running a simple hello world Quarkus application, it starts in ~1s:

![command line output](https://user-images.githubusercontent.com/25560159/81528510-76be2280-938f-11ea-9948-e4ea099e6ad6.png)

This simple hello world Quarkus application can found on my [GitHub](https://github.com/jiajunngjj/quarkus-helloworld). 

## Overview
Quarkus works out-of-the-box with popular Java standards, frameworks, and libraries like Eclipse MicroProfile and Vert.x. Quarkus dependency injection is based 
on CDI hence Java developers are able to use JAX-RS, JPA and many others. Quarkus also includes an extension framework which allows 
developers to write extensions for integration and enhancement. A list of existing available extensions can be found [here](https://github.com/quarkusio/quarkus/tree/master/extensions).

Run-time wise, Quarkus offers two run modes: 
1. JVM 
2. Native

JVM mode packages Quarkus applications as JAR files and run on vanilla OpenJDK HotSpot while native mode uses GraalVM to create a standalone executable that does not
run in a Java VM. Both modes have its pros and cons, and it depends on use case to see which mode is the best fit. Considering an event-driven scenario where events trigger the function 
to spin up a service in real-time to react to the event. It will make sense to use native mode because JVM takes a while to start.

An interesting feature is the hot-reload capability (dev mode) where any changes made to any Java file, application config or static 
resources will be compiled automatically once the browser is refreshed. This is made possible with Quarkus augmentation phase where almost everything (metadata, 
config parsing, logics) is already configured in build-time, the whole reload takes less than a second when changes are made in run-time.  This is enabled by default and to use it, just need to run:
```
mvn compile quarkus:dev
```
This improves developer's experience especially when it comes to debugging. 

## Serverless Architecture
As Quarkus has extremely short startup time, it brings Java developers to the serverless world where workloads can be scaled or scaled to zero based on demand. 
Quarkus is a good fit for serverless applications and is event-driven in nature. Together with Kubernetes, the serverless pattern can be implemented easily where
an event triggers the application and Kubernetes starts a container to handle that event. The application might produce some results from that compute or 
processing and once idle for enough time, that container will be scaled down to zero. This solves the under-provisioning and over-provisioning problems. I've written
a demo using Quarkus and Knative to create a serverless application on OpenShift (Red Hat's flavour of Kubernetes). OpenShift Serverless is based on Knative, an open 
source Kubernetes serverless project, which is generally available today. 

![Serverless Pattern](https://user-images.githubusercontent.com/25560159/81521596-40c27380-937a-11ea-9e3d-bb3ec9a048b1.png)

Demo: [https://github.com/jiajunngjj/knative-quarkus-openshift-demo](https://github.com/jiajunngjj/knative-quarkus-openshift-demo)

## Conclusion
All in all, Quarkus aims to optimize Java development for distributed application architectures. Its container-first approach is aligned with microservices and event-driven
architecture, and digital transformation strategies. Also, serverless computing model plays an important role in the cloud because it controls computing cost.
Quarkus in Java ecosystem seems to be a game-changer, and moving forward, I'm excited to see how Quarkus can bring fast innovation and help enterprises to stay ahead of competitors.

## References
* [http://quarkus.io/](http://quarkus.io/)
* [https://github.com/quarkusio/quarkus](https://github.com/quarkusio/quarkus)
