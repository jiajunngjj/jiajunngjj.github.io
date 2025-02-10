---
title: "Secure your application from Argo CD to Kubernetes"
tags:
- DevSecOps
- Devops
- argocd
- gitops
- snyk
permalink: /:year/:month/:day/secure-your-application-from-argocd-to-kubernetes
---

GitOps is a popular framework for managing and securing the application development pipeline. For many who have embarked on a GitOps journey, a common question is: “how can I secure my pipeline when everything is automated?”

The GitOps framework is a concept where any code commits or changes are done through Git, which then triggers an automated pipeline that builds and deploys applications on Kubernetes. Because there are few touch points for development and security teams in the pipeline, its security needs to be mandated to ensure the deployed applications have as few vulnerabilities as possible.

This blog covers how Snyk can provide application security in GitOps, focusing on a popular tool, Argo CD. In this scenario, Snyk runs an IaC scan to ensure the to-be-deployed application is safe before deployment, and stops the build if it is not. Snyk also can monitor the deployed applications across different namespaces in Kubernetes in an automated fashion.

[Full blog here...](https://snyk.io/blog/secure-apps-from-argocd-to-kubernetes/)
