---
title: "Why vSphere with Tanzu?"
tags:
- kubernetes
- containers
- vsphere
- vmware 
- tanzu
permalink: /:year/:month/:day/why-vsphere-with-tanzu
---

Are you running your own Kubernetes cluster already? If so, how are you managing it currently?

![kubernetes cluster](https://user-images.githubusercontent.com/25560159/121776669-c25d8400-cbc0-11eb-9e03-1740919b10f3.png)

Kubernetes was born in 2014 and since then, it has become the default way of managing and scaling containerized applications. Many organisations have begun their journey
towards microservices architecture and scalable solutions for their applications, going the cloud native way. Kubernetes plays a part in the cloud native journey alongside with many other open source tools, not to mention the
native cloud providers which provide the underlying infrastructure. 

During this journey, many infrastructure and operation folks face steep learning curves in managing and operating Kubernetes while the developers are happily exploring
the new way of application development, maybe. Kubernetes is a complicated and interesting platform on their own and every add-on can be an open source project. Essential functionalities like monitoring and 
logging in Kubernetes are also achieved through open source projects. Of course if you have enterprise software which provides logging feature, you can integrate with Kubernetes as well.

![CNCF Landscape](https://user-images.githubusercontent.com/25560159/122045558-7c96fa80-ce10-11eb-9e9f-ed84f217721a.png)
<sub><sup>[Image Source](https://landscape.cncf.io/)

Above is a picture of the cloud native landscape from Cloud Native Computing Foundation (CNCF). This landscape of open source projects 
provides end-user communities with viable options for building cloud native applications. This should give you an idea of the complexity
one has to go through to go the cloud native way. 

Embarking on a journey with Kubernetes is just a start of cloud native development, and many software vendors have already released 
its own distribution of Kubernetes, packaging with its own selected list of open source projects, providing supportability. 

There are many vendors to choose from, and today, I would like to share a unique feature from the VMware Tanzu portfolio, vSphere with Tanzu, which addresses the steep learning curve of Kubernetes from an
infrastructure perspective.

## What is vSphere with Tanzu
![vsphere and kubernetes](https://user-images.githubusercontent.com/25560159/121643488-74635600-cac4-11eb-9d0b-3d502ce9a67a.png)

> "**vSphere with Tanzu** is the new generation of vSphere for containerized applications. This single, streamlined solution bridges the gap between IT operations and developers with a new kind of infrastructure for modern, cloud-native applications both on premises and in public clouds." - VMware

vSphere with Tanzu integrates Kubernetes and containers into the vSphere ecosystem, providing an unique capability to run Kubernetes workloads directly on ESXi hosts.

With the Tanzu feature turned on in vSphere, you can start running Kubernetes in two ways:
* run containers directly in ESXi host via **vSphere Pod** service (requires NSX-T)
* run Tanzu Kubernetes cluster via **Tanzu Kubernetes Grid Service**

### vSphere Pod
> "**vSphere Pod** allows container workloads to be deployed directly in the ESXi host. It is Open Container Initiative (OCI) compatible and can run containers from any operating system as long as the containers are OCI compatible."

For a detailed explanation on vSphere Pod, you can visit [here](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-276F809D-2015-4FC6-92D8-8539D491815E.html).

### Tanzu Kubernetes Grid Service
> "**Tanzu Kubernetes Grid Service** allows Kubernetes cluster to be deployed or consumed in a self-service manner."

For a detailed explanation on Tanzu Kubernetes Grid service, you can visit [here](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-4D0D375F-C001-4F1D-AAB1-1789C5577A94.html#GUID-4D0D375F-C001-4F1D-AAB1-1789C5577A94).

## Architecture
![Vsphere with Tanzu Architecture](https://user-images.githubusercontent.com/25560159/121887969-0334d480-cd4a-11eb-9f99-18816e7be2eb.png)


With the Tanzu feature turned on in vSphere:
* Kubernetes control planes are deployed into the ESXi hosts which enables the capability to run Kubernetes workloads within ESXi
* It creates a Supervisor cluster, a layer which overlays the ESXi hosts
* Within the Supervisor cluster, an administrator would need to create vSphere namespace to either run vSphere Pods or Tanzu Kubernetes cluster, which provides quota management and isolation, very similar to [Kubernetes's namespace concept](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/).

## Use case
For organizations who are running a lot of virtual machine (VM) workloads and are looking to start modernizing their applications, vSphere with Tanzu removes the need to go through complicated steps to create a Kubernetes cluster manually. It also provides a single
pane of glass to manage both VM and container workloads. This is an awesome experience from an operating perspective where you can manage resources for Kubernetes workloads the same way VM resources are managed.

Apart from organizations who are starting their Kubernetes journey, vSphere with Tanzu is also ideal as an environment setup for testing/development even if one already has a production Kubernetes cluster. One key feature on vSphere is that the Tanzu Kubernetes cluster can be provisioned
in a self-service manner which is highly efficient for testing applications. This could possibly reduce the need for an infrastructure as code (IaC) tool to a certain extent as vSphere takes care of it now. Also, vSphere with Tanzu comes with many out of the box capabilities like load balancing, container networking, container registry etc. All these features make the setup of a 
Kubernetes cluster easy.

For organizations who are looking to run a production Kubernetes cluster, vSphere with Tanzu can also be a starting point because you can extend the capabilities whenever you need it. Referring back to the CNCF landscape, with so many tools out there, the question is how do you know what you need? 
You will only know when you start using it. Tanzu Kubernetes comes with different editions where you can add on the capabilities. The concept here is to start small, and grow it when the capability is there.

## Conclusion
The idea is to have a single platform to manage different kinds of applications, from monolithic applications to cloud native
applications. With Kubernetes integrated into the vSphere ecosystem, Kubernetes workload can leverage on vSphere features (vMotion, Distributed Resource Scheduler (DRS), etc) to further provide stability to the applications.

vSphere with Tanzu is a good start for organizations who are looking to kickstart their Kubernetes journey. 
Depending on the Kubernetes capability, this feature can be a bridge to the containers world, upskilling the infrastructure folks, bringing the organization forward together as one unit. 


