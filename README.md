# Project description

This is a simple Demo of using `Spring Boot` and `MongoDB`. Application has a simple form on the main page. 
Entered data on the form will be saved `users` collection in `MongoDB`. 

## Project architecture

Application consists of:

* `UserController` which handles request from `index.html` page
* `UserResource` - is `REST` endpoint for retrieving `Users`
* `UserRepository` - repository interface for persistence and retrieving `Users` from `MongoDB`

Because it's a simple demo for storing and retrieving users to/from database, `UserRepository` injected directly in 
`UserController` and `UserResource`.

## How to build project

To build, or run project `java` and `MongoDB` should be installed.

From the root folder of the project just run `./gradlew build`
Compiled `springboot-mongo-demo.jar` file can be found at `build/libs`

## How to run project locally

1. To run project from the built `.jar` file:

From the root folder of the project run the following command in terminal:

```bash
java -jar build/libs/springboot-mongo-demo.jar
```

2. To run the project with gradle:

From the root folder of the project run the following command in terminal:

```bash
gradlew bootRun
```

3. To run the project from the Intellij Idea:

Open the project in Intellij Idea  
Open `DemoApp.java` file. Right click on the `main` method and select `Run 'DemoApp.main()'` on the pop-up window.

Running the project locally assumes that `java` and `MongoDB` are installed locally.

## How to work with the application

After application is started open the following into any browser [http://localhost:8080](http://localhost:8080)

Application also provides `REST` API for retrieving all users or user by `userID`.  
To retrieve all users:
[http://localhost:8080/api/users](http://localhost:8080/api/users)   
To retrieve user by ID:   
[http://localhost:8080/api/users/user_id](http://localhost:8080/api/users/user_id)   
`user_id` should be replaced by real `ID` from the main page.

## How to build and run application using `Docker` file

On how to run and build application using `Docker` please read this [README](docker/README.md) file or this tutorial:
[How to run Spring Boot and MongoDB in Docker container](https://dev-pages.info/how-to-run-spring-boot-and-mongodb-in-docker-container/)# springboot-mongo

How to run in Docker
As was mentioned above we need 2 containers: the first container will be used to run our Java application. The second container will be used to run MongoDB.

Create Docker Network
Our 2 containers should communicate with each other. To do so we need to create a new docker Network named spring_demo_net:

docker network create spring_demo_net
On the start our containers will connect to this network.

How to run MongoDB in Docker.
For MongoDB we'll use the official image from Docker Hub.

Before starting the MongoDB container we need to create a folder where Mongo will store all the data:

mkdir -p ~/mongo-data
Now Let's start the Mongo container into the following way:

docker run --name spring-demo-mongo --network=spring_demo_net -v /home/ubuntu/mongo-data:/data/db -d mongo
Let'stake a look at this command closely:

docker run starts the container
--name spring-demo-mongo specifies the name of the container.
--network=spring_demo_net means to which network container should be connected
-v /home/ubuntu/mongo-data:/data/db shares the host's /home/ubuntu/mongo-data folder and mounts it to the container's /data/db folder.
-d means to start the container as a daemon.
mongo is a docker image name.
To verify our Mongo container is running we can use the following command:

docker ps
The output will look into the following way:

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
1b6ff814b73c        mongo               "/entrypoint.sh mongo"   4 seconds ago       Up 3 seconds        27017/tcp           spring-demo-mongo
How to run Spring Boot in Docker.
To run our Demo application we'll use the official Oracle image for Open JDK. Actually we'll build our own image upon the official openjdk Docker image.

Create Dockerfile
To build our own image we can use the following Dockerfile file:

FROM openjdk:8-jdk

ADD springboot-mongo-demo.jar springboot-mongo-demo.jar
RUN sh -c 'touch /springboot-mongo-demo.jar'
ENTRYPOINT ["java", "-Dspring.data.mongodb.uri=mongodb://spring-demo-mongo/users","-Djava.security.egd=file:/dev/./urandom","-jar","/springboot-mongo-demo.jar"]
Dockerfile contains steps for building our image.

FROM openjdk:8-jdk - means to build our image upon openjdk image from Docker Hub.
ADD springboot-mongo-demo.jar springboot-mongo-demo.jar - adds our application springboot-mongo-demo.jar file to the container.
RUN sh -c 'touch /springboot-mongo-demo.jar' - runs added jar file inside the container.
ENTRYPOINT .... - adds (ENTRYPOINT) the configuration for running our jar file.
I'd like to pay your attention on the following Entrypoint's parameter: -Dspring.data.mongodb.uri=mongodb://spring-demo-mongo/users. This parameter specifies URI for MongoDB. So, we can create different images for different environment, e.g. for QA and PROD with different URI.
And also spring-demo-mongo should be the same as named our MongoDB container.
Dockerfile could be placed at any folder. For demo purpose we've placed Dockerfile into springboot-mongo-demo/docker/ folder of our project.

Build Java project
We are almost ready to build our image, but first we need to build our Java project and place the jar file into the springboot-mongo-demo/docker folder of our project:

./gradlew clean build && cp build/libs/springboot-mongo-demo.jar docker/
Build Docker image
Now let's build docker image. Change the directory to springboot-mongo-demo/docker and run the following command:

docker build --tag=spring-demo-1.0 .
--tag=spring-demo-1.0 - specifies the name of our container. It can be used for starting, stopping, restarting, removing container in easier way.

The output will look like this:

Step 1 : FROM openjdk:8-jdk
 ---> 861e95c114d6
Step 2 : MAINTAINER Aliaksei Bahdanau lex.nox@gmail.com
 ---> Using cache
 ---> 0069240de54a
Step 3 : ADD springboot-mongo-demo.jar springboot-mongo-demo.jar
 ---> f689c6b5d60c
Removing intermediate container 8c1b8536fd37
Step 4 : RUN sh -c 'touch /springboot-mongo-demo.jar'
 ---> Running in 04e627b54b98
 ---> 69b94c770ad5
Removing intermediate container 04e627b54b98
Step 5 : ENTRYPOINT java -Dspring.data.mongodb.uri=mongodb://spring-demo-mongo/users -Djava.security.egd=file:/dev/./urandom -jar /springboot-mongo-demo.jar
 ---> Running in 0135836ed1bf
 ---> c78a5f6516d3
Removing intermediate container 0135836ed1bf
Successfully built c78a5f6516d3
Now we can verify that our spring-demo-1.0 image is available locally:

docker images
The output will look like:

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
spring-demo-1.0     latest              c78a5f6516d3        3 minutes ago       679.5 MB
openjdk             8-jdk               861e95c114d6        20 minutes ago      643.2 MB
mongo               latest              7f09d45df511        20 minutes ago      336.1 MB
Run Docker container
If our spring-demo-1.0 image was build successfully and available locally, we can run container with our Spring Boot application as follows:

docker run -d --name spring-demo --network=spring_demo_net -p 8080:8080  spring-demo-1.0
The command for running Spring Boot application inside docker looks very similar as for starting MongoDB container but with the following difference:
-p 8080:8080 - publishes a container's port(s) to the host.

To see containers log file we can use the following command:

docker logs spring-demo

