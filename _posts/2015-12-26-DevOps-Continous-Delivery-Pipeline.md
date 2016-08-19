---
layout: post
title: DevOps Continuous Delivery Pipeline with Docker, Jenkins, Gitlab, Snap Creator and NetApp Clustered Data ONTAP
author: Kapil Arora
---

### What is Continuous Delivery ?
Continuous Delivery is automation and streamlining of the complete Software release process such that the delivery process of the software is reliable, repeatable, frequent, and provides immediate feedback. The process starts from  development and ends in the production deployment.

### What are we doing?
We will showcase a simple continuous delivery pipeline using some popular DevOps tools and show you how we can take application backups and create clone environments  from production data using NetApp Snap Creator and Clustered Data ONTAP.
![Image](/images/devops-tools.png)
* **Docker**: Helps automate and orchestrate packaging and deployment of applications in Software containers.
* **Docker-compose**: Tool to define and run multi-container applications.
* **Jenkins**: Continous Integration tool. Helps build the complete pipeline.
* **GitLab**: Source code management.
* **Docker Registry**: To store and manage Docker Images.
* **Snap Creator**: To create storage and time efficient database backups on NetApp Clustered Data ONTAP.
* **NetApp Manageability SDK**: To communicate with NetApp Clustered Data ONTAP.

### What is the Sample application ?
We developed a simple Ruby and MySQL application. The Database backend is provided by MySQL and Ruby exposes a REST API to display Database table contents in JSON format.

### What is the build pipeline like?
The Build Pipeline is completed in 3 steps:
1. **Development**: This step is triggered by **git push** to the git repository. That means as soon as there is a code change.
This step will build our application and eventually package it as a Docker image and push this image to our Docker Image Repository.
2.  **Test**: This step will pull the Docker image from the Docker Image repository and deploy it along with the mysql database with test data on the Server and run tests on it and eventually underlay it.
3.  **Production deployment**: This step will pull the latest image and deploy it on the production server.
![Image](/images/continuous-delivery-pipeline.png)
 For each step we have created a dedicated GIT repository and a Jenkins Job.

### How do we backup the application?
As part of the production database deployment we add Snap Creator Agent to the Container. This Agent helps us take Application consistent backups of MySQL Database.Further these Backups can be moved to a Secondary site(On Premise, NPS or Cloud ONTAP) using Snap Creator as well.

### How do we create test environments from Production MySQL data?
We created a new GIT repository and Jenkins Job for this Operation and this Job can be triggered OnDemand.
We will use NetApp Manageability SDK to connect with the Storage Controller (Storage Virtual Machine) and create volume clone which are space and time efficient using NetApp's Flex Clone technology.
The job will also take care of mounting this new flex clone on our Server and deploy the application and run tests.

### How and where is the pipeline created?
We define the Pipeline in Jenkins using Git push as the trigger for the Development Step. The next jobs start as soon as the previous one is successfully completed.
![Image](/images/jenkins-pipeline.png)
### Where is the code?
[Github Repo](https://github.com/kapilarora/continuous-delivery-pipeline)
 
_**Warning:**_
This is a very simplified form of deployment for demonstartion purpose. Many aspects of a real deployment e.g. security have not been paid much attention to. 
