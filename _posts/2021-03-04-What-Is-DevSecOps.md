---
title: "Introduction to Dev + Sec + Ops"
tags:
- devsecops
- appdev 
permalink: /:year/:month/:day/intro-to-devsecops
---
## What is DevSecOps
DevSecOps builds on top of the DevOps methodology where it addresses code quality and reliability assurance. You probably would have already guessed it, "DevSecOps" expands "DevOps" and has an additional focus, **security**; eliminating silos between development, operations and security.

![DevSecOps](https://user-images.githubusercontent.com/25560159/109853707-2b179500-7c91-11eb-9d0e-6bdaf16967b5.png)
<sub><sup>[Image Source](https://sevaa.com/app/uploads/2018/10/ft-devsecops.png)

**Definition of DevSecOps**
> "DevSecOps is the integration of security into emerging agile IT and DevOps development as seamlessly and as transparently as possible. Ideally, this is done without reducing the agility or speed of developers or requiring them to leave their development toolchain environment." - Gartner

Security has been in the spotlight for the past couple of years where software vulnerabilities are being exploited at both the application and infrastructure layers; on-premise and on cloud. One of such attacks was the SolarWinds hack in 2020. Over the years,
organisations have constantly transforming and changing the way they deploy applications and where they deploy them. With things moving so fast, organisations are looking for ways to tackle security challenges. 
Fundamentally with DevSecOps, security is baked into the end-to-end software development lifecycle (SDLC), and vulnerabilities can be caught and fixed before anyone can exploit them. 

In this article, I will be sharing some of the practices around DevSecOps and some tools that can be integrated into the pipeline. This is an expansion to my
previous article, [Introduction to DevOps](https://jjblog.io/2020/10/14/intro-to-devops) which covers the basics of DevOps.

## Benefits of DevSecOps
![Detect Vulnerabilities](https://user-images.githubusercontent.com/25560159/109900151-db0cf280-7cd1-11eb-86ca-423e3ef79796.png)
<sub><sup>[Image Source](https://www.tenable.com/sites/drupal.dmz.tenablesecurity.com/files/styles/640x360/public/images/articles/shadow_brokers_vulnerability_detection_v1.png?itok=akRCxXFk)

Adoption of DevSecOps has the following main benefits:
* **Early detection of vulnerabilities** - spot bugs at an earlier stage with an automated workflow embed with security tools.
* **Improve application overall security** - with security built into the application, it will have lesser bugs.
* **Save costs on resource management** - risk and vulnerability assessment can be done from the start and through development instead of waiting until after a release.


## How to implement / transiting to DevSecOps
![DevSecOps](https://user-images.githubusercontent.com/25560159/109900515-769e6300-7cd2-11eb-85bc-5fbfaff73a66.png)
<sub><sup>[Image Source](https://www.plutora.com/wp-content/uploads/2019/03/DevSecOps-Diagram.png)

To integrate security into DevOps successfully, the concept of **building security into the product** must be applied. It is never about having security integrated to a finished product.

DevSecOps adopts a few practices in its life cycle (simplified with examples):
* **Culture** - security has to be prioritised at the beginning (top-down approach), usually starting from the planning and designing stage. Every team needs to understand their security responsibilities and how to manage risks/threats.
* **Coding** - prioritizing the security requirements as a part of the product backlog, instill security knowledge into developers, they need to take ownership and practise secure coding, writing clean codes which adhered to secured coding standards.
* **Automation** - automate development and operations side of things which include applying/provisioning of repeatable security best practices through the orchestration of containers (Kubernetes),
  integrate security standards and guidelines into IDEs.
* **Testing** - automated security, penetration, functional and performance tests. Define your own static application security testing (SAST), dynamic application security testing (DAST), runtime application self-protection (RASP); find the right tools for these tests.
* **Deployment** - scanning of container images for vulnerabilities and automated deployment through secured CI/CD and APIs.
* **Measurement and Monitoring** - Real-time and continuous monitoring on production workloads with alerts and notifications; and intrusion detection. Gather feedback to improve the next release of the application.

Adoption of DevSecOps will have some challenges because it is more of a culture to technology. DevSecOps should start with a culture shift and getting buy-in from top management should help to drive the culture within the organisation.
To effectively drive adoption, here are some suggestions:
* Evaluate current security measures and conclude on how to improve them 
* Mandate security at every stage of the pipeline 
* Processes should be automated as much as possible 
* Developers to be trained to code securely

## Tools that can be integrated into the pipeline
There are many tools available, both commercial or open source, that can be integrated into existing CI/CD pipeline and below are some of my suggestions:

**SonarQube** - continuous inspection of code quality to perform automatic reviews with static analysis of code.  
**Harbor** - open source registry that secures artifacts with policies and role-based access control, ensures images are scanned and free from vulnerabilities, and signs images as trusted.  
**HashiCorp Vault** - secures, stores, and tightly controls access to tokens, passwords, certificates, API keys, and other secrets in modern computing.  
**ThreatModeler** - identify, predict and define threats across the entire attack surface to make proactive security decisions and minimize overall risk.   
**Aqua Security** - provides security for containers, serverless and cloud-native applications throughout the DevSecOps pipeline.

## Conclusion
DevSecOps builds trust and security into an application. As organisations are undergoing digital transformation, moving into the cloud native world with containers and microservices, it introduces new 
things to worry about and security remains prominent. There will never be a fully secured application but fostering the DevSecOps mindset and culture will be the key to tackle security challenges in the unfamiliar environment.
Like many security programmes, the successful implementation is heavily dependent on people, processes and technologies; everyone has a role to play.