---
title: "How to run Snyk test in CircleCI and Argo CD?"
tags:
- DevSecOps
- Devops
- argocd
- circleci
- gitops
- snyk
permalink: /:year/:month/:day/snyk-with-circleci-and-argocd
---
How do you secure GitOps pipeline? In this post today,
I will be sharing on how you can find vulnerabilities using Snyk in GitOps toolset, mainly CircleCI and Argo CD.

![Snyk CircleCI ArgoCD](https://user-images.githubusercontent.com/25560159/171653052-06379084-2a2f-41ed-bf17-1d9f84cac0da.png)

GitOps is an interesting framework to manage and secure the application development pipeline. For instance, the GitOps workflow of an organization which deploys applications onto a K8s platform can be as follows: 
1. Developers  deploy their codes to a Git repository
2. Triggers the CI/CD pipeline once the merge request is approved
3. Updating the declarative application configuration files
4. The CD tool picks it up from Git and detects the configuration drift 
5. Deploy the application onto the K8s platform automatically to make sure that the desired state is achieved

The above workflow can happen without giving developers direct access to the K8s cluster hence GitOps can definitely fill up the security gaps in distributed environments, and using Git as a single source of truth for declarative infrastructure and applications is just one of them.

There are many tools which can implement GitOps in your application lifecycle and in this post today, the focus will be on using CircleCI as the continuous integration tool, and Argo CD as the declarative continuous delivery tool.

## CircleCI Setup with Snyk
CircleCI is a cloud based CI/CD tool which has built support for parallelism and Docker workflow, and CircleCI will be my CI tool.

In my CircleCI, set up the integration with GitHub (I'm using GitHub as my Git repository). My repository can be found here:
[https://github.com/jiajunngjj/circleci-goof](https://github.com/jiajunngjj/circleci-goof). Once the setup is completed, 
you should be able to see the following workflow:

![CircleCI workflow](https://user-images.githubusercontent.com/25560159/171648416-a6f9a29f-4f26-45d1-ae19-fa0d82061621.png)

Here is the config.yml:
```
version: 2.1
orbs:
  snyk: snyk/snyk@1.1.2

jobs:
  build:
    docker:
      - image: circleci/node:latest
    steps:
      - checkout
      - run: npm install -q
  snyk_code_test:
    docker:
    - image: snyk/snyk-cli:npm
    steps:
    - checkout
    - run:
        command: | 
          snyk auth $SNYK_TOKEN
          snyk code test || true
  snyk_oss_test:
    docker:
    - image: circleci/node:4.8.2
    steps:
    - checkout
    - snyk/scan:
        token-variable: SNYK_TOKEN
        fail-on-issues: false
        monitor-on-build: false      
  docker_build_push:
    docker:
      - image: cimg/node:18.0.0
    steps:
      - checkout
      - setup_remote_docker
      - run: |
          TAG=0.1.<< pipeline.number >>
          docker build -t $DOCKER_USERNAME/docker-goof:$TAG .
          docker login -u $DOCKER_USERNAME --password $DOCKER_PASSWORD
          docker push $DOCKER_USERNAME/docker-goof:$TAG 
  snyk_container_test:
    docker:
    - image: snyk/snyk-cli:npm
    steps:
    - checkout
    - run:
        command: | 
          TAG=0.1.<< pipeline.number >>
          snyk auth $SNYK_TOKEN
          snyk container test jiajunngjj/docker-goof:$TAG --file=./Dockerfile || true
  trigger_argocd:
    docker:
      - image: cimg/base:2022.04
    steps:
      - run : |
          TAG=0.1.<< pipeline.number >>
          git clone https://github.com/jiajunngjj/argocd-goof.git
          cd argocd-goof
          sed -i 's/\(docker-goof\)\(.*\)/\1:'$TAG'/' goof/goof-deployment.yaml
          git config user.email "ngjiajun13@gmail.com"
          git config user.name "jiajunngjj"
          git add goof/goof-deployment.yaml
          git commit -m "update from circleci to trigger argocd"
          git push https://$GITHUB_PERSONAL_TOKEN@github.com/jiajunngjj/argocd-goof.git 
workflows:
  build_and_test:
    jobs:
      - build
      - snyk_code_test:
          requires:
            - build
      - snyk_oss_test:
          requires:
            - build
      - docker_build_push:
          requires:
            - snyk_code_test
            - snyk_oss_test
      - snyk_container_test:
          requires:
            - docker_build_push
      - trigger_argocd:
          requires:
            - snyk_container_test
```
Note: I appended `|| true` to ensure my Snyk test succeeds so it can move to the next stage.

### Requirements
* Snyk account
* DockerHub account
* Set up a variable, `SNYK_TOKEN`, using Snyk API token
* Set up a variable, `GITHUB_PERSONAL_TOKEN`, to store your GitHub personal access token
* Set up a variable, `DOCKER_USERNAME`, to store your DockerHub username
* Set up a variable, `DOCKER_PASSWORD`, to store your DockerHub password

### CI Workflow
1. *build* - Build/Compile the application
2. *snyk_code_test* - Run Snyk Code test (Static Application Security Testing (SAST))
3. *snyk_oss_test* - Run Snyk Open Source test (Software Composition Analysis (SCA))
4. *docker_build_push* - Run Docker workflow to build container image and push into my DockerHub repository
5. *snyk_container_test* - Run Snyk Container test to scan the built image 
6. *trigger_argocd* - Update the git repository with the new image tag which Argo CD tracks

## Argo CD Setup with Snyk
Argo CD is a K8s native CD tool which tracks a repository and compares the application state with the current state of the K8s cluster. Then, it applies the required changes to the cluster configuration.
I will be using Argo CD as the CD tool to deploy the application directly onto a K8s cluster. 
Argo CD can be deployed either on K8s or VMs. My GitHub repository which CircleCI updates and Argo CD tracks, can be found here:
[https://github.com/jiajunngjj/argocd-goof](https://github.com/jiajunngjj/argocd-goof)

1. Creates an application on your Argo CD: 
![Argo CD creates app](https://user-images.githubusercontent.com/25560159/171633101-eba77e3e-efbb-4c5d-b04a-8cb99dece1bb.png)
2. Click on the `Sync` button

The above steps should create all the K8s objects (as per my GitHub repository) in your cluster.

To run Snyk IaC scan on the K8s yaml files, you will need to create a K8s job object:
```

   
apiVersion: batch/v1
kind: Job
metadata:
  name: snyk-iac-scan
  annotations:
    argocd.argoproj.io/hook: PreSync
spec:
  ttlSecondsAfterFinished: 600
  template:
    spec:
      containers:
      - name: snyk-cli
        image: snyk/snyk-cli:npm
        command: ["/bin/sh","-c"]
        args:
          - git clone https://github.com/jiajunngjj/argocd-goof.git;
            snyk auth $SNYK_TOKEN;
            snyk iac test argocd-goof/goof/goof-deployment.yaml || true;
        env:
          - name: SNYK_TOKEN
            valueFrom:
              secretKeyRef:
                name: snyk-token
                key: token
      restartPolicy: Never
  backoffLimit: 0
```
Note that you will need to create a K8s secret, `SNYK_TOKEN` in the namespace (in my case is apples) which you're going to deploy the applications.
The `argocd.argoproj.io/hook: PreSync` will run the `snyk-cli` container, which runs the Snyk IaC test on my `goof-deployment.yaml` before the actual
deployment of my `goof-deployment.yaml`. Any detection of vulnerabilities would result in a failure in deployment.

## Sample Output
**CircleCI**

Pipeline running:
![circleci ss1](https://user-images.githubusercontent.com/25560159/171661268-39ea519b-ca94-4b68-afc6-1cbd3500755d.png)

Snyk Code scan:
![circleci ss2](https://user-images.githubusercontent.com/25560159/171661544-e6c9069c-4d17-4008-a836-f0594858880d.png)

**Argo CD**

Before configuration drift detection:
![argocd ss1](https://user-images.githubusercontent.com/25560159/171661806-d8de0b57-a42a-40d5-ade4-c4b1d7fc19d4.png)

After configuration drift detection, it starts to sync and run the job (Snyk IaC scan):
![argocd ss2](https://user-images.githubusercontent.com/25560159/171662981-e338f59c-57fc-40d3-9603-9a7ee5cee97e.png)

Snyk IaC scan:
![argocd ss3](https://user-images.githubusercontent.com/25560159/171662686-04c80cd2-1865-4d57-807c-322710d0073d.png)

## Wrapping up
The entire CI/CD workflow is as follows:

**CircleCI**
1. Build/Compile the application
2. Run Snyk Code test (SAST)
3. Run Snyk Open Source test (SCA)
4. Run Docker workflow to build container image and push into my DockerHub repository
5. Run Snyk Container test to scan the built image
6. Update the goof-deployment.yaml in my GitHub with the new image tag, which Argo CD tracks

**Argo CD**
7. Detects the configuration drift
8. Starts to sync to achieve the new state
9. Triggers Snyk IaC scan
10. If Snyk IaC detects no vulnerabilities, goof-deployment.yaml will be deployed

The GitOps workflow will fail whenever Snyk detects any vulnerabilities with severity above the threshold you defined.
And this will ensure that vulnerable codes will never reach production environment.