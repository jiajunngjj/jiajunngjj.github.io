---
title: "Micro Frontend For Multi Tenancy"
tags:
- appdev
- microservices
permalink: /:year/:month/:day/micro-frontend-for-multi-tenancy
---
Well, I share a passion for all things Kubernetes and have been working on it for quite some time. Recently I have had the opportunity to work on something different, something more application related. In this simple post, I am going pen down my thoughts on micro frontend architecture, something which I have proposed in my solution.

## Introduction to Micro Frontend
Micro frontend (MFE) has a been around for quite some time and it is part of the best practices under the microservices architecture design. While microservices approach focuses on backend services, MFE is responsible for the frontend portion. MFE breaks the monolithic frontend application into smaller and loosely couple services.

![MFE architecture-1](https://github.com/user-attachments/assets/432dd124-b867-4abe-a4ac-068f67473364)

In my recent solutioning, I have proposed the use of MFE to tackle one of the key challenges, multi-tenancy. In my current work, solutions have to be secure, and air gapped/disconnected, with only certain protocols allowed for interfaces. Multi-tenancy is challenging because there is a need for modularization of the frontend and the delivery has to be agile.

## Design Considerations
With the above mentioned, MFE is suitable in the solution as it addresses multi-tenancy and agile delivery well. 
![MFE architecture-2](https://github.com/user-attachments/assets/a8c8441a-bb04-4eea-aea1-6916c3854a4c)

### Host and Remote MFEs
In MFE, Host application is responsible for orchestrating and managing the lifecycle of MFE and it dynamically loads the remote MFE. Host MFE handles routing and navigation, providing the overall look and feel of the application. 
On the other hand, Remote MFEs will be developed by the tenant. With this, each tenant can design their own page (remote MFE) independently within the Host MFE. Reusable components are also built and make available to tenant so there is certain consistency. 

### Module Federation And Shared Libraries/Dependencies
Module Federation has significantly changed the MFE landscape. As the frontend is designed in a single SPA, the use of module federation can slice the SPA into multiple smaller remote MFEs. In the design, we are proposing vite module federation which allows the sharing of dependencies and resources across MFE, facilitating integrations. This way of sharing dependencies allows each MFE to use the correct versions of the shared libraries, avoiding conflict.

### Communication Strategies
![MFE communication strategies ](https://github.com/user-attachments/assets/d3ba0d97-1c51-41a0-82b9-e27564841e41)
There are many ways to allow MFE to communicate with each other such as URL/query parameters or through a shared global state management etc. In my solution, MFEs use a shared service layer to act as a mediator for communication. This is a shared backend service which allows MFE to communicate by making API calls, and it serves as the data exchange layer.

### Deployment Strategies
![sample cicd](https://github.com/user-attachments/assets/defedc95-7baa-4c29-af8f-5844c50369e2)

With this scalable and distributed architecture design, development works can be decoupled where each MFE can be developed, deployed and maintained independently by each tenant. With a centralized DevSecOps environment, each tenant development team can leverage on a centralized CI/CD pipeline for deployment. This allows independent team to innovate faster and scale efficiently while reducing risk.

### Application Development Strategies
![app languages](https://github.com/user-attachments/assets/f347e23e-dd2b-487d-901d-a53d91cc5f13)

The solution itself uses Vue.js. However, MFE architecture allows multiple frameworks, for instance, a tenant development team might be more familiar with React, and they can build their Remote MFE with React. The Host MFE will be able to consume the Remote MFE.

## Wrapping Up
While there are many benefits to the MFE design, there are also some challenges as well. Complexity can kick in in terms of integration and communication. Styling consistency can be difficult to maintain if different frameworks are used, and there is no proper standardization. 

Personally, I like the concept of MFE. And I will put into my solution if the use case calls for it. However, in the event that there is a only a single team building the application and the team is unfamiliar with microservices application development, MFE might not be recommended.


