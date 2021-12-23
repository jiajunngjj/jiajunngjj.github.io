---
title: "Integrate Snyk Into Google Cloud Build"
tags:
- DevSecOps
- Devops
- GCP
- Snyk
- GoogleCloudBuild
permalink: /:year/:month/:day/integrate-snyk-into-google-cloud-build
---
Are you using Google Cloud Build as your CI/CD pipeline to build your applications? If so, how do you mandate security across the different stages?

![Snyk Google Cloud Build](https://user-images.githubusercontent.com/25560159/147042759-5e92873d-e1db-41f8-ad09-65f410698766.png)

In this post today, I will be sharing on how to integrate Snyk into Google Cloud Build as there isn't any native support available yet.

There are way too many options when it comes to choosing your pipeline engine. The obvious choice should be of a balance of integration options and team expertise.
Previously, I have written blog posts on [DevOps](https://jjblog.io/2020/10/14/intro-to-devops) and
[DevSecOps](https://jjblog.io/2021/03/04/intro-to-devsecops), and have shared examples for some of the tools to set up the pipeline. So why Google Cloud Build?
I believe it is because of its native cloud nature (serverless) which makes it simple to set up. I do have seen cloud native organisations moving towards Google Cloud Platform (GCP) with many having to like the idea of using
with the GCP stack which includes the Google Cloud Build.

Today's topic is on Snyk integration; although Snyk supports a wide variety of CI/CD tools (e.g Jenkins) but there isn't native support for Google Cloud Build yet.
In such cases, the integration with Snyk will have to be through the Snyk command-line interface (Snyk CLI).

## What is Snyk CLI
![Snyk CLI](https://user-images.githubusercontent.com/25560159/147044601-7770a8e8-e353-42ec-867f-59d4d90dda57.png)
<sub><sup>[Image Source](https://docs.snyk.io/features/snyk-cli)

> "**Snyk CLI** brings functionality of Snyk into your development workflow. It can be run locally, or in your CI/CD pipeline, to scan your projects for security issues." - Snyk

Snyk CLI allows "plug and play" integration to any pipeline workflow with the flexibility to trigger a scan in different stages.
Snyk CLI is a Node.JS application which you can install using `npm` or `yarn`.

There are other methods to install such as using a Docker image which I will be using in my setup.
In the official documentation [here](https://docs.snyk.io/features/snyk-cli/install-the-snyk-cli), you will be able to find the different installation methods.

## Setup
In my setup, I will be building a Node.js application which is hosted on my personal GitHub, [https://github.com/jiajunngjj/goof](https://github.com/jiajunngjj/goof).
I will be running Snyk Container in Google Cloud Build to run a couple of Snyk tests :
* Code analysis (SAST)
* Software composition analysis (SCA)
* Scanning of container images
* Scanning of infrastructure as code (IaC) files

Also, I will show how to generate a HTML report for my SCA portion using [snyk-to-html](https://github.com/snyk/snyk-to-html), a Snyk utility to convert JSON output to HTML,
and store the artifact in my Google Cloud Storage.

### Snyk Setup
On Snyk Platform, retrieve the access token from the service account (sign up for an account if you don't have one).
Go to Settings -> Service accounts, create a new service account to generate the token.

![Snyk Settings](https://user-images.githubusercontent.com/25560159/147032928-c484b746-2bae-42b6-a3c6-338da9640543.png)

### Google Cloud Build Setup

2. In the GCP console, navigate to Cloud Build. Go to Triggers, creates a new trigger.

3. In the Trigger page, add the Node.js application GitHub repository under Source. Follow the steps in Connect repository to connect.

4. In the Trigger page, add a variable, `_SNYK_TOKEN`, with the access token from Snyk Setup.

5. In the Trigger page, create the Inline YAML with the following content under Configuration (*cloudbuild.yaml*):

#### Run Snyk Open Source Test

```
steps:
  - name: snyk/snyk-cli:npm
    args:
      - '-c'
      - |-
        snyk config set api=${_SNYK_TOKEN}
        snyk test --severity-threshold=medium
    id: Snyk Open Source Test
    entrypoint: bash
```

#### Run Snyk Code Test

```
steps:
  - name: snyk/snyk-cli:npm
    args:
      - '-c'
      - |-
        snyk config set api=${_SNYK_TOKEN}
        snyk code test --severity-threshold=medium
    id: Snyk Code Test
    entrypoint: bash
```

#### Run Snyk Container Test
```
steps:
  - name: snyk/snyk-cli:npm
    args:
      - '-c'
      - |-
        snyk config set api=${_SNYK_TOKEN}
        snyk container test --severity-threshold=medium jiajunngjj/docker-goof:latest
    id: Snyk Container Test
    entrypoint: bash
```

#### Run Snyk IaC Test
```
steps:
  - name: snyk/snyk-cli:npm
    args:
      - '-c'
      - |-
        snyk config set api=${_SNYK_TOKEN}
        snyk iac test --severity-threshold=medium main.tf
    id: Snyk IaC Test
    entrypoint: bash
```

#### Create HTML report and store it in your Google Cloud Storage

```
steps:
  - name: snyk/snyk-cli:npm
    args:
      - '-c'
      - |-
        snyk config set api=${_SNYK_TOKEN}
        set -o pipefail
        snyk test --severity-threshold=medium --json | snyk-to-html -o results.html || true
    id: Create HTML artifact
    entrypoint: bash
artifacts:
  objects:
    location: 'gs://jjng-store/scan_output'
    paths:
      - results.html
```
Here are some notes for my configurations above:
* Snyk Docker image, `snyk/snyk-cli:npm`, is used to run Snyk CLI
* `--severity-threshold=medium` to flag out vulnerabilities of medium or higher severity, meaning if there is any of vulnerabilities of severity medium or higher, the build will fail
* `|| true` to provide exit code 0 as the build needs to be successful in order to save the artifact in the Google Cloud Storage
* The assumption that you have built your container image and pushed it to a container registry (Docker Hub in my example) hence you're running
the image scan by specifying the container image directly on the registry

Here is the combined version of the above Snyk tests, which makes sense in the software development lifecycle (SDLC) where you scan application codebase for SAST and SCA, scan container base images before building your containers, and scan the IaC files before provisioning the infrastructure:
```
steps:
  - name: snyk/snyk-cli:npm
    args:
      - '-c'
      - |-
        snyk config set api=${_SNYK_TOKEN}
        snyk test --severity-threshold=medium || true
    id: Snyk Open Source Test
    entrypoint: bash
  - name: snyk/snyk-cli:npm
    args:
      - '-c'
      - |-
        snyk config set api=${_SNYK_TOKEN}
        snyk code test --severity-threshold=medium || true
    id: Snyk Code Test
    entrypoint: bash
  - name: snyk/snyk-cli:npm
    args:
      - '-c'
      - |-
        snyk config set api=${_SNYK_TOKEN}
        snyk container test --severity-threshold=medium jiajunngjj/docker-goof:latest
    id: Snyk Container Test
    entrypoint: bash
  - name: snyk/snyk-cli:npm
    args:
      - '-c'
      - |-
        snyk config set api=${_SNYK_TOKEN}
        snyk iac test --severity-threshold=medium main.tf || true
    id: Snyk IaC Test
    entrypoint: bash
  - name: snyk/snyk-cli:npm
    args:
      - '-c'
      - |-
        snyk config set api=${_SNYK_TOKEN}
        set -o pipefail
        snyk test --severity-threshold=medium --json | snyk-to-html -o results.html || true
    id: Create HTML artifact
    entrypoint: bash
artifacts:
  objects:
    location: 'gs://jjng-store/scan_output'
    paths:
      - results.html
```
Note that I have set `|| true` above to make sure all tests pass.


## Sample Output
Here is a sample output of a failed build of Snyk Code Test:

![Failed build](https://user-images.githubusercontent.com/25560159/147050780-e81c535b-0178-4a30-ab40-8c43acc7874f.png)

Here is a sample output of my Google Cloud Storage:

![HTML report storage](https://user-images.githubusercontent.com/25560159/147051058-23a77950-92bd-40b2-8105-eb5e03d98adf.png)

Here is a sample output of my HTML artifact:

![HTML report](https://user-images.githubusercontent.com/25560159/147051128-4044c783-9383-4949-89c0-1cb997ab847e.png)

## Conclusion
Mandating security is a must when it comes to building software, especially in the agile world today where there are a lot of moving pieces. Snyk CLI helps to streamline various security tests in the SDLC, covering different
aspects of security from SAST and SCA to building container and provisioning infrastructure. 

Most importantly, Snyk CLI makes it possible to integrate with any CI/CD tool and all these tests can be automated (just like the example above), bringing agility to the development team.