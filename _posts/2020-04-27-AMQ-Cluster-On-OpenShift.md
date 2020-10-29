---
title: "AMQ Messaging on OpenShift"
tags: 
  - redhat
  - openshift
  - amq broker
  - messaging
  - event-driven
permalink: /:year/:month/:day/amq-openshift
---
More companies are moving towards a microservices approach for their new applications.
This approach suits today’s increasingly distributed hybrid cloud and multi-cloud IT infrastructures. Thanks to advances in container orchestration platform, it is easier to 
edge away from traditional monolithic applications towards distributed and highly-available microservices. Microservices approach would mean more services and there is a need for
 a messaging layer to serve the inter-service communication. In this post, I'm going to share how AMQ brokers can leverage on OpenShift.

## Topology
![Red Hat - AMQ Broker on OpenShift](https://user-images.githubusercontent.com/25560159/80358496-1ae39c00-88af-11ea-8607-421b9d1a9a3b.png)

Unlike traditional deployment of AMQ broker where configuration is needed for high availability (HA) and clustering, AMQ brokers on OpenShift form a cluster automatically when there is more than 
one broker Pod. Note that there is no master/slave failover model on OpenShift so there is no need for message replication between master/slave either.

When determining the number of  brokers required to handle certain workload, OpenShift has auto scaling feature which can be configured based on CPU consumption. For instance, a threshold can be set to 70% and when the CPU utilization reaches 70%, 
OpenShift scales up the AMQ broker Pods; and it scales down when the heavy traffic is gone.

Internal services connecting to the broker within Openshift cluster is straightforward through the Kubernetes concept of Services. However, external services running outside of OpenShift will have some restrictions to connect to the broker.
One easiest option is to use SSL and access the broker from the route, and the other is through NodePort binding.

Demo: [https://github.com/jiajunngjj/amq-openshift-demo](https://github.com/jiajunng/amq-openshift-demo)

## High Availability (HA)
The availability of the brokers and integrity of the messaging data are maintained through Kubernetes concepts of Health Checks, Stateful Sets, Persistent Volumes (PVs) and Persistent Volume Claims (PVCs).
Health checks are configured in OpenShift to detect and restart unhealthy containers. In the HA configuration of AMQ brokers on OpenShift, there will be at least 2 broker Pods running with each writing messaging data to its own PV. If a broker Pod goes offline, the message
data stored in that PV is redistributed to an alternative available broker, which then stores it in its own PV. 

Message migration can be enabled on OpenShift to migrate messages when scaling down broker Pods due to failure. This further protects the integrity of messaging data.

Demo: [https://github.com/jiajunngjj/amq-ha-openshift-demo](https://github.com/jiajunng/amq-ha-openshift-demo)

## Storage
With container-native storage (OpenShift Container Storage) deployed on OpenShift, PV can be provisioned dynamically with the broker Pod. Else, PV has to be manually provisioned and ensure that they are available 
for broker Pod to consume. For instance, a cluster of two broker Pods with persistent storage would required two PVs available. Note that in OpenShift deployment, there is no need for a
shared file system with distributed locking capabilities since each broker Pod has their own PV and message data is redistributed if it goes offline.

## Multi-Cluster
![AMQ Broker - Multi-cluster](https://user-images.githubusercontent.com/25560159/80358992-cc82cd00-88af-11ea-82eb-bd50300ea1c3.png)

With more companies leaning towards hybrid and multi-cloud strategy, it is common to have more than one OpenShift clusters across different data centers. 
AMQ Interconnect is the solution to connect brokers across OpenShift clusters. It is a lightweight AMQP message router which routes messaging data 
between AMQP-enabled endpoints, including clients, brokers, and standalone services. 
AMQ Interconnect is based on Dispatch Router from the Apache Qpid™ project.

Demo: [https://github.com/jiajunngjj/amq-federation-openshift-demo](https://github.com/jiajunng/amq-federation-openshift-demo)

## Conclusion

There are many benefits of deploying AMQ broker on OpenShift as AMQ provides simple configuration for allowing the setup of a Broker Mesh, 
from how configurations are made easier to leveraging on Kubernetes-native capabilities. It is something worth considering when it comes to microservices
strategy, which is also to have middleware workload containerized and deployed on OpenShift.


## References
* [Red Hat AMQ Broker On OpenShift Documentation](https://access.redhat.com/documentation/en-us/red_hat_amq/7.6/html/deploying_amq_broker_on_openshift/index)
* [Red Hat AMQ Interconnect On OpenShift Documentation](https://access.redhat.com/documentation/en-us/red_hat_amq/7.6/html/deploying_amq_interconnect_on_openshift/index)
