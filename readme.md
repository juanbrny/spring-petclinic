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

  

Containerized Jenkins controller will be installed with Bitnami's Helm chart: https://bitnami.com/stack/jenkins/helm 

  

The Linux virtual machines will be based in SLES15 (SUSE Linux Enterprise Server 15) with the minimal pattern and we will deploy OpenJDK 11, Maven 3.6.3 and Docker 20.10 on top of them. 

  

### GitHub repository 

Our reference repository will be https://github.com/spring-projects/spring-petclinic. 

Please, fork it to your own GitHub account as we will need to do some modifications. 

  

We will keep the code unmodified but will extend and improve the repository, so it does not only contain the code for our Petclinic application, but also the definition files for our Jenkins pipeline and our Docker container. 

  

### JFrog container registry 

JFrog offers a free hosted artifact management system that not only includes the required container registry functionality, but also provides useful tools like container vulnerability scanning. Being fully hosted it frees us from the burdens of having to deploy this component in our test lab. 

  

This service can also be used as a replacement for Maven Central for our Java artifacts management but that will be covered in a future update. 

  

Just create your free account at https://jfrog.com/start-free/ and you are ready to go. 

There is a predefined repository for containers called "default-docker-virtual" so there is no need to configure another one. 

The only configuration step required will be creating an auth key for our user. We will need those credentials later to be able to push container images to the registry. 

  

### Maven Central

As we explained, for simplicity, we will be using Mave Central as our artifacts repository.  

In corporate environments it is highly recommended that you have full control about the source, availability, and security of your artifacts. That is why we do recommend using JFrog Artifactory to cache and store them. 

The same free service that we described for the container registry can be configured to act as our main artifact's provider. 

  

## Project setup 



### Add Jenkins and Docker support to our project 

The original Petclinic project already contains all the code needed to build the application either with Maven or Graddle but it does not contain any definition for automated build pipelines or container creation. 

  

That is why we added two folders: jenkins and docker. 

  

In those folders you will find all the bits and pieces needed to create our build pipelines in Jenkins 

  

### Define Maven Central as the main repository 

We will modify the original POM file to make sure that all dependencies go through Maven Central. In a future version, we will replace this repository with JFrog Artifactory for better control of our code sources. 

  ```xml

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

In our Jenkins console we create a new item named spring-petclinic-pipeline of type "Pipeline"
![Create Jenkins Pipeline](https://github.com/juanbrny/spring-petclinic/blob/main/doc/blob/jenkins1.png?raw=true)

We will use the default values except in the Pipelines section where we will select Pipeline script from SCM. 
![Jenkins Pipeline Form](https://github.com/juanbrny/spring-petclinic/blob/main/doc/blob/jenkins2.png?raw=true)



This means that the pipeline definition will be grabbed from GitHub, this way we can have full versioning control not only for the application code but also for our build and deployment automation. 

Use this values to complete the Pipeline creation form:
``` 
Git Repo: https://github.com/juanbrny/spring-petclinic.git 

Branch: */main 

Credentials: (not needed as our repo is public, add yours in case of private repo)

Script Path: jenkins/Jenkinsfile 
``` 

Click on Save and we are ready to go. 
 
In the main screen of the newly created pipeline, we now can select "Build Now" to test the execution.  

![Building](https://github.com/juanbrny/spring-petclinic/blob/main/doc/blob/jenkins3.png?raw=true)

During the execution we can monitor the progress using the "Console Output" 

![Output](https://github.com/juanbrny/spring-petclinic/blob/main/doc/blob/jenkins4.png?raw=true) 

Once finished we can analyze the result and see how the container has been pushed to the container registry: 

![Result](https://github.com/juanbrny/spring-petclinic/blob/main/doc/blob/jenkins5.png?raw=true)


Final step, go to our container registry to double check the new image is published. 

![Registry](https://github.com/juanbrny/spring-petclinic/blob/main/doc/blob/jfrog6.png?raw=true)
  

## Running our brand-new container 

Now we do have our Petclinic application built, packaged, and published as a container. 

  

Let's try it. 

  

As we can see in our registry, our newly created image is tagged as: juanbrny.jfrog.io/default-docker-local/spring-petclinic:22.02.3 

  

We only need a workstation with Docker enabled to run it: 

```bash
docker run -p 0.0.0.0:8080:8080 --rm juanbrny.jfrog.io/default-docker-local/spring-petclinic:22.02.3 
```
  

And here we go. Our Petclinic is up and running! 
```bash
jenkinsbuild01:~ # docker run -p 0.0.0.0:8080:8080 --rm juanbrny.jfrog.io/default-docker-local/spring-petclinic:22.02.3
Unable to find image 'juanbrny.jfrog.io/default-docker-local/spring-petclinic:22.02.3' locally
22.02.3: Pulling from default-docker-local/spring-petclinic
Digest: sha256:56818840cc0fe9e3505d148a040c02d48523396c34c609d8e15339a845ff4ae6
Status: Downloaded newer image for juanbrny.jfrog.io/default-docker-local/spring-petclinic:22.02.3


              |\      _,,,--,,_
             /,`.-'`'   ._  \-;;,_
  _______ __|,4-  ) )_   .;.(__`'-'__     ___ __    _ ___ _______
 |       | '---''(_/._)-'(_\_)   |   |   |   |  |  | |   |       |
 |    _  |    ___|_     _|       |   |   |   |   |_| |   |       | __ _ _
 |   |_| |   |___  |   | |       |   |   |   |       |   |       | \ \ \ \
 |    ___|    ___| |   | |      _|   |___|   |  _    |   |      _|  \ \ \ \
 |   |   |   |___  |   | |     |_|       |   | | |   |   |     |_    ) ) ) )
 |___|   |_______| |___| |_______|_______|___|_|  |__|___|_______|  / / / /
 ==================================================================/_/_/_/

:: Built with Spring Boot :: 2.6.3


2022-02-15 15:36:02.311  INFO 1 --- [           main] o.s.s.petclinic.PetClinicApplication     : Starting PetClinicApplication v2.6.0-SNAPSHOT using Java 11.0.13 on 5ba308d7b1c8 with PID 1 (/application.jar started by root in /)
2022-02-15 15:36:02.315  INFO 1 --- [           main] o.s.s.petclinic.PetClinicApplication     : No active profile set, falling back to default profiles: default
2022-02-15 15:36:03.652  INFO 1 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
2022-02-15 15:36:03.720  INFO 1 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 57 ms. Found 2 JPA repository interfaces.
2022-02-15 15:36:04.510  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2022-02-15 15:36:04.523  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2022-02-15 15:36:04.524  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.56]
2022-02-15 15:36:04.584  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2022-02-15 15:36:04.585  INFO 1 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 2188 ms
2022-02-15 15:36:04.835  INFO 1 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2022-02-15 15:36:05.133  INFO 1 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2022-02-15 15:36:05.550  INFO 1 --- [           main] org.ehcache.core.EhcacheManager          : Cache 'vets' created in EhcacheManager.
2022-02-15 15:36:05.564  INFO 1 --- [           main] org.ehcache.jsr107.Eh107CacheManager     : Registering Ehcache MBean javax.cache:type=CacheStatistics,CacheManager=urn.X-ehcache.jsr107-default-config,Cache=vets
2022-02-15 15:36:05.570  INFO 1 --- [           main] org.ehcache.jsr107.Eh107CacheManager     : Registering Ehcache MBean javax.cache:type=CacheStatistics,CacheManager=urn.X-ehcache.jsr107-default-config,Cache=vets
2022-02-15 15:36:05.641  INFO 1 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
2022-02-15 15:36:05.707  INFO 1 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 5.6.4.Final
2022-02-15 15:36:05.904  INFO 1 --- [           main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.2.Final}
2022-02-15 15:36:06.052  INFO 1 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
2022-02-15 15:36:06.924  INFO 1 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
2022-02-15 15:36:06.935  INFO 1 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2022-02-15 15:36:08.769  INFO 1 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 13 endpoint(s) beneath base path '/actuator'
2022-02-15 15:36:08.834  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2022-02-15 15:36:08.852  INFO 1 --- [           main] o.s.s.petclinic.PetClinicApplication     : Started PetClinicApplication in 7.146 seconds (JVM running for 7.859)
2022-02-15 15:36:34.756  INFO 1 --- [0.0-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2022-02-15 15:36:34.756  INFO 1 --- [0.0-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2022-02-15 15:36:34.758  INFO 1 --- [0.0-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 2 ms
```

Let's access with the browser:

![Petclinic](https://github.com/juanbrny/spring-petclinic/blob/main/doc/blob/petclinic7.png?raw=true)

  

# Summary 

This is an extremely basic example, but it presents some key concepts around modern software building and deployment strategies that every DevOps engineer should master. 

It covers source control, software artifacts management and container creation and execution. All glued together by Jenkins to provide a fully automated approach to software construction. 

Do you want to know more? 

In future examples we will review: 

- How to integrate JFrog's Jenkins Artifactory plugin to deploy artifacts and resolve dependencies to and from your Artifactory repositories 

- Better integrate Jenkins in Kubernetes to use containerized agents instead of virtual machines 

- Target Kubernetes as the deployment environment for our containers. 

  

Stay tuned! 

  

  

 
