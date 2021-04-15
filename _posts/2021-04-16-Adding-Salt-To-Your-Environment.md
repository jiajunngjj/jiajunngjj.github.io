---
title: "Adding Salt To Your Environment"
tags:
- devsops
- automation
permalink: /:year/:month/:day/add-salt-to-your-environment
---
![Salt Shaker](https://user-images.githubusercontent.com/25560159/114913607-f7b15400-9e53-11eb-91a9-7a3694de1b1f.png)

Automation, orchestration and configuration management tools are important foundation of an IT day-to-day operation. Especially in today's world, where organizations are moving towards agile practices to deliver
applications faster. These responsibilities are often given to the development teams, empowering them to provision, configure, and manage their own infrastructure. 

There are many tools out there which does the job
and in this post, I will be sharing about SaltStack, an open source tool which is acquired by VMware in 2020.

## What is SaltStack
![SaltStack](https://user-images.githubusercontent.com/25560159/114822677-01f03580-9df5-11eb-8224-b8de8d5dc8b1.png)

> "**SaltStack, also known as Salt** is a Python-based, open-source software for event-driven IT automation, remote task execution, and configuration management. " - Wikipedia

Salt automates the configuration of your infrastructure from on-premise to cloud. Repetitive manual system administrative tasks or processes can be automated with Salt to reduce unnecessary errors when it comes to configuring 
systems. It simplifies management of systems as you can execute commands remotely to multiple machines directly in parallel. Not to mention that Salt supports different operating systems, including various distributions of GNU/Linux, Microsoft Windows, and macOS.

Salt also leverages on YAML programming language, and most of the time, users will be writing in YAML for their configurations unless users would like to write their own modules in Python.

## SaltStack Architecture
Salt uses a publisher-subscriber model and follows a concept of "**Master**" and "**Minion**", where Master is the controlling node and Minion is the node that is being controlled. Master and Minion daemon service will be installed upon setting up Salt. 

Communication between Master and Minion is through an event bus backed by [ZeroMQ](https://zeromq.org/) which does not require any message broker and creates an asynchronous network topology to provide the fastest communication possible.

<img width="1016" alt="Salt architecture" src="https://user-images.githubusercontent.com/25560159/114836757-8ea2ef80-9e05-11eb-90f6-759791f90a31.png">

Adding on to the above architecture, multi master nodes can be configured to provide high availability. 

For users who do not like the idea of having an agent, agentless Salt is available as well. In such setup, minion daemon does not need to be installed in the remote host. Instead, the only requirements on the remote host are SSH and Python.
Agentless Salt can be set up in conjunction with a master-minion environment as well. 

<img width="756" alt="Salt agentless" src="https://user-images.githubusercontent.com/25560159/114838796-a7140980-9e07-11eb-91ed-f94d68ad2db2.png">

The recommended setup is to operate in a Master and Minion approach as execution via SSH is substantially slower, users would have already experienced this when managing large number of nodes.

## SaltStack Use Case
**DevOps**

Apart from the typical use case of automating day-to-day tasks and managing multiple systems, Salt can play a part in the DevOps space as well. It can automate everything a developer requires, 
from managing source code and configuration, scheduling jobs, to cloud provisioning. Salt is vendor agnostic and highly pluggable, it is able to interface with other CI tools and version-control systems. 

An example is to integrate Jenkins with Salt where Salt can be used to deploy and configure environment such as deploying a docker container, and Jenkins will execute the tests upon the new Salt change automatically. 
In such example, Salt also provides security to DevOps as it allows organisations to provide public and private keys for all masters and minions, guaranteeing their authenticity and providing a secure channel for master/minion communications without sacrificing performance or scalability.

**Event Driven Automation**

Salt is built around event driven infrastructure, you can monitor events with Salt Runner, and non-Salt related events (system load, service status, etc) with Salt Beacon, and Salt Reactor to trigger actions in response to any event.
An example is to watch for computing resources such as disk, processor, ram, and other system stats and take action if it exceeds defined thresholds.

## VMware's Acquisition - SaltStack Config and SaltStack SecOps
With VMware's announcement of the acquisition, Salt is integrated into the vRealize Automation (vRA) solution as features known as SaltStack Config and SaltStack SecOps. Salt further expands
vRA capabilities into an enterprise-grade management tool which includes policy and governance, Infrastructure as Code and DevSecOps.

![vRealize Automation Features](https://user-images.githubusercontent.com/25560159/114902820-c5e6c000-9e48-11eb-92fb-da7cde44abb6.png)

The capabilities are summed up as below:  

**SaltStack Config**
* Simple and modern languages python and YAML to manage applications and infrastructure
* Self-healing with event driven automation to detect events such as errors and apply solution automatically
* Agent or agentless management of systems

**SaltStack SecOps**
* Enforce continuous compliance with pre-built CIS and DISA STIGs content
* Scan and remediate critical vulnerabilities with automation
* Real-time insights of the state of systems and applications

## Conclusion
SaltStack is pretty well known when it comes to automation and its unique capabilities. With SaltStack integrated into vRA, there is no doubt that vRA can provide
a more complete solution when it comes to event driven automation and compliance management within the organization's operations.

You can read more about the vRA with SaltStack:
* [VMware vRealize Automation Adds Native Configuration Management with vRealize Automation SaltStack Config](https://blogs.vmware.com/management/2020/11/vmware-vrealize-automation-saltstack-config-launch.html)
* [VMware vRealize Automation SaltStack SecOps : Technical Overview Part 1 – Compliance Management](https://blogs.vmware.com/management/2021/04/vra-saltstack-config-secops-compliance-management.html)

And some good SaltStack references:
* [A Guide to Infrastructure Automation in 2020](https://saltproject.io/infrastructure-automation-2020-a-quick-guide/)
* [SaltStack GitHub](https://github.com/saltstack/salt)
