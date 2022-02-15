## Introduction 

  

This is an example of how to test and build a Java application and have it packaged as a container using an automated pipeline. 

  

Our reference Java application will be Petclinic (for this example we'll use the version built with Spring Boot). 

  

These are the components that we'll be using for our build automation environment: 

  

- A continuous integration tool: Jenkins 

- An artifact repository: Maven Central 

- A source code manager: GitHub 

- A container registry: JFrog's Registry 

  

## Environment setup 

  

### Jenkins 

Our pipeline infrastructure consists of a Jenkins controller deployed in Rancher Kubernetes Cluster and a set of Jenkins Agents deployed on Linux virtual machines. 

  

Containerized Jenkins controller will be deployed with Helm using Bitnami's chart: https://bitnami.com/stack/jenkins/helm 

  

The Linux virtual machines will be based in SLES15 (SUSE Linux Enterprise Server 15) with the minimal pattern and we will deploy OpenJDK 11, Maven 3.6.3 and Docker 20.10 on top of them. 

  

### GitHub repository 

Our reference repository will be https://github.com/spring-projects/spring-petclinic. Please, fork it to your own GitHub account as we will need to do some modifications. 

  

We will keep the code unmodified but will extend and improve the repository, so it does not only contain the code for our Petclinic application, but also the definition files for our Jenkins pipeline and our Docker container. 

  

### JFrog container registry 

JFrog offers a free hosted artifact management system that not only includes the required container registry functionality, but also provides useful tools like container vulnerability scanning. Being fully hosted it frees us from the burdens of having to deploy this component in our test lab. 

  

This service can also be used as a replacement for Maven Central for our Java artifacts management but that will be covered in a future update. 

  

Just create your free account at https://jfrog.com/start-free/ and you are ready to go. 

There is a predefined repository for containers called "default-docker-virtual" so there is no need to configure another one. 

The only configuration step required will be creating an auth key for our user. We will need those credentials later to be able to push container images to the registry. 

  

### Maven Centra 

As we explained, for simplicity, we will be using Mave Central as our artifacts repository.  

In corporate environments it is highly recommended that you have full control about the source, availability, and security of your artifacts. That is why we do recommend using JFrog Artifactory to cache and store your artifacts. 

The same free service that we described for the container registry can be configured to act as our main artifact's provider. 

  

## Project setup 

  

### Add Jenkins and Docker support to our project 

The original Petclinic project already contains all the code needed to build the application either with Maven or Graddle but it does not contain any definition for automated build pipelines or container creation. 

  

That is why we added two folders: jenkins and docker. 

  

In those folders you will find all the bits and pieces needed to create our build pipelines in Jenkins 

  

### Define Maven Central as the main repository 

We will modify the original POM file to make sure that all dependencies go through Maven Central. In a future version, we will replace this repository with JFrog Artifactory for better control of our code sources. 

  ``` 

<repositories> 

   <repository> 

    <id>central</id> 

    <name>Maven Central</name> 

    <layout>default</layout> 

    <url>https://repo1.maven.org/maven2</url> 

    <releases> 

    <enabled>false</enabled> 

    </releases> 

    <snapshots> 

    <enabled>false</enabled> 

    </snapshots> 

   </repository> 

  </repositories> 

``` 

  

### Create your build pipeline 

  

Our build pipeline will have 4 stages. The first two stages will focus compiling the code, running the unit tests and package the application. 

Once the application jar is created the remaining steps will take care of creating a container with a Java runtime environment that will execute the application. Once the container is built, it'll be properly tagged and pushed to the remote container registry. 

  

### Pipeline execution 

![4269f3501839f3378427941297f67b18.png](:/83284a55aa5f4c9fa8e3215064405ff4) 

screen1 

We will use the default values except in the Pipelines section where we will select Pipeline script from SCM. 

This means that the pipeline definition will be grabbed from GitHub, this way we can have full versioning control not only for the application code but also for our build and deployment automation. 

  

Git Repo: https://github.com/juanbrny/spring-petclinic.git 

Branch: */main 

Credentials: not needed as our repo is public. 

Script Path: jenkins/Jenkinsfile 

screen2 

Click on Save and we are ready to go. 

  

In the main screen of the newly created pipeline, we now can select "Build Now" to test the execution.  

During the execution we can monitor the progress using the "Console Output" 

scren4 

Once finished we can check the result and we can see how the container has been pushed to the container registry: 

sceren5 

  

As a final check we will go to our container registry to double check the new image is published. 

screen6 

  

## Running our brand-new container 

Now we do have our Petclinic application built, packaged, and published as a container. 

  

Let's try it. 

  

As we can see in our registry, our newly created image is tagged as: juanbrny.jfrog.io/default-docker-local/spring-petclinic:22.02.3 

  

We only need a workstation with Docker enabled to run it: 

docker run -p 0.0.0.0:8080:8080 --rm juanbrny.jfrog.io/default-docker-local/spring-petclinic:22.02.3 

  

And here we go. Our Petclinic is up and running! 

j7 

  

# Summary 

This is an extremely basic example, but it presents the core concept around modern software building and deployment strategies that every DevOps engineer should master. 

It covers source control, software artifacts management and container creation and execution. All glued together by Jenkins to provide a fully automated approach to software construction. 

Do you want to know more? 

In future examples we will review: 

- How to integrate JFrog's Jenkins Artifactory plugin to deploy artifacts and resolve dependencies to and from your Artifactory repositories 

- Better integrate Jenkins in Kubernetes to use containerized agents instead of virtual machines 

- Target Kubernetes as the deployment environment of our containers. 

  

Stay tuned! 

  

  

 
