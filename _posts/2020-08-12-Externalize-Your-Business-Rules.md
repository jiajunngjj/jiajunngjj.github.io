---
title: "Externalize Your Business Rules with Red Hat Decision Manager"
tags: 
  - redhat
  - decision manager
  - drools
  - jboss
  - appdev
permalink: /:year/:month/:day/externalize-your-business-rules
---
![Red Hat Decision Manager](https://user-images.githubusercontent.com/25560159/90093092-4d7dab80-dd5d-11ea-842d-2839045814cb.png)

In this post, I will be sharing my experience on using Red Hat Decision Manager (RHDM) as the rules engine, using DRL (Drools Rule Language) rules to isolate decision logic from application 
code.

Recently, I was involved in creating a fraudulent detection demo/workshop which requires decision making based on a certain set of rules. 
The part which involves business rules is when a prediction model computes a score from the raw data received, and the application needs to make a 
decision based on the score. This is the part where a rules engine manages decision processes using pre-defined logic to determine outcomes.

Before going into my setup, let's understand why there is a need for a business rules engine. Usually, business rules are hidden inside an application code and this is a problem for non-technical
folks. Non-technical folks like the business people know the rules and regulations policy but do not understand application code. It will be difficult to manage the rules especially when 
business applications can contain tens of thousands of rules which are continually changing. Therefore, there will be impact when it comes to adding, removing or changing rules.
As such, an external business rules engine helps to decouple application code from business rules and allows a centralized management of rules.

## Concepts
RHDM is based on the [Drools](https://www.drools.org/) project, one of the widest used rules engines in the industry which many in-house applications are already using.

![Drools Engine](https://user-images.githubusercontent.com/25560159/90094510-b74b8480-dd60-11ea-91cf-5f5ed519f2e3.png)

The decision engine operates using the following basic components:
* **Rules**: Business rules or DMN decisions that you define. All rules must contain at a minimum the conditions that trigger the rule and the actions that the rule dictates.
* **Facts**: Data that enters or changes in the decision engine that the decision engine matches to rule conditions to execute applicable rules.
* **Production memory**: Location where rules are stored in the decision engine.
* **Working memory**: Location where facts are stored in the decision engine.
* **Agenda**: Location where activated rules are registered and sorted (if applicable) in preparation for execution.

**What is a rules engine**
> "A business rules engine is a software system that executes one or more business rules in a runtime production environment" - Wikipedia

A rules engine is a collection of objects with conditions and actions. Businesses can run their process model against it and let it run through the conditions.

An example would be:
* Company's credit score of 50 or less will be given the status of "Disallow".

These are a few benefits of using a rules engine:
* Logical separation of rules from application code.
* Human-readable rule which allows non-technical folks to understand the rules easily.
* Centralization of rules provides flexibility to make adjustment easily without the need of going through the application codes.
* Rules can be reused.

**What is a business rule**
> "A business rule defines or constrains some aspect of business and always resolves to either true or false. It specifically involves terms, facts and rules. 
Business rules are intended to assert business structure or to control or influence the behavior of the business." - Wikipedia

Business rules are essential to guide and help in decision making. Businesses would aim to automate them but not every business decision can be automated. The goal is to
automate repeatable operational decisions.

An example would be something like this:
* A bank would disapprove a loan to a customer/company which has a credit rating of 50 or lower.

## Set up
There are many ways to set up RHDM and here is how I set it up:

![RHDM Arch](https://user-images.githubusercontent.com/25560159/90087001-4d29e400-dd4e-11ea-8501-3dc3ebe5eaf3.png)

* Decision Central - An authoring environment with GUI to allow low code development (DMN, DRL rules, decision table, etc). It also manages Kjars deployments to multiple Kie Servers.
* KIE Server - It hosts the decision services which execute rules and processes. KIE server exposes its functionality via REST, JMS and Java interfaces to client applications.
* Git - It stores business assets such as DMN model, decision tables, etc. In my case, it stores the DRL rules. 

My RHDM sits on OpenShift Container Platform where my KIE server runs in a container and is able to leverage on Kubernetes's capabilities (self-healing, etc). 
Whenever I update and commit my rules to the Git server, it automatically triggers the CI/CD flow to deploy the new set of rules embedded into the KIE server, and roll out the
container deployment using a rolling update strategy where the previous instance is terminated only when the new one is ready.

Here is how I write my rules in DRL:

![DRL](https://user-images.githubusercontent.com/25560159/90090035-8ca7fe80-dd55-11ea-996f-611548b7477b.png)

I used the Decision Central to create a Result object and write my DRL rules. My DRL rules are simple, I am bascially writing a condition where a score of less than 70% would mean not fraud and a score of more than 70% means fraud. 
Afterwhich, I built and deployed my KIE server with the DRL rules. When the KIE server is deployed successfully, my application sends a payload with the score and receives a response payload containing the status.


My written rules can be found on my Git Hub, [https://github.com/jiajunngjj/drl-fraud-demo](https://github.com/jiajunngjj/drl-fraud-demo) which can be imported directly onto Decision Central as well.
There are many ways to create your decision service and since my project is just a simple set of rules, DRL rules are sufficient.


## Conclusion
It is always the best practice to externalize rules from applications to allow a more agile and efficient development. 
There are many use cases which RHDM can be part of the solution from fraud detection in banking industry to automated trading in capital markets and etc. 

While driving applications with external business rules is beneficial, 
it is also important to note that it is an additional layer to an application architecture, hence the application and its requirements should be carefully designed to ensure it can leverage on the external engine with
minimal to zero application code change when a rule changes. 


