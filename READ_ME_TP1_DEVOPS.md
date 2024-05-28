# Corentin DOUCHE - DEVOPS TP1

## TP1 - DOCKER 

Creating the DockerFile 

FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

docker network create app-network
docker build -t my_postgres:latest .
docker run --name my_postgres_container -p 5432:5432 --network=app-network -d my_postgres:latest
docker run -p "8090:8080" --net=app-network --name=adminer -d adminer

Now everything is linked, connected, and we can access the db in the local host 8090
Let's create two files:

CREATE TABLE public.departments
(
 id      SERIAL      PRIMARY KEY,
 name    VARCHAR(20) NOT NULL
);

01-CreateScheme.sql
CREATE TABLE public.students
(
 id              SERIAL      PRIMARY KEY,
 department_id   INT         NOT NULL REFERENCES departments (id),
 first_name      VARCHAR(20) NOT NULL,
 last_name       VARCHAR(20) NOT NULL
);

02-InsertData.sql
INSERT INTO departments (name) VALUES ('IRC');
INSERT INTO departments (name) VALUES ('ETI');
INSERT INTO departments (name) VALUES ('CGP');


INSERT INTO students (department_id, first_name, last_name) VALUES (1, 'Eli', 'Copter');
INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Emma', 'Carena');
INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Jack', 'Uzzi');
INSERT INTO students (department_id, first_name, last_name) VALUES (3, 'Aude', 'Javel');

Then we change the docker file, adding two lines:
COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d/
COPY 02-InsertData.sql /docker-entrypoint-initdb.d/

Here we can notify the changes on the adminer local host.

# Persist
Adding the volume using :

docker run --name my_postgres_container -p 5432:5432  -v "C:\Users\Coren\TP1 DEVOPS\postgres\my_postgres_data":/var/lib/postgresql/data --network=app-network -d my_postgres:latest

# Simple backend api

A multistage build in Docker allows you to use multiple FROM statements in a single Dockerfile. Each FROM statement starts a new build stage, and you can selectively copy artifacts from one stage to another, leaving behind unnecessary intermediate build dependencies. This helps to keep the final image size small and optimized.

Let's break down each step of the Dockerfile you provided:


FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
This line sets the base image for the build stage, which is Maven with Amazon Corretto JDK 17.
The AS myapp-build part assigns a name (myapp-build) to this build stage for reference later.

ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
Sets an environment variable MYAPP_HOME to /opt/myapp, which will be used as the application directory inside the container.
Sets the working directory inside the container to /opt/myapp.

COPY pom.xml .
COPY src ./src
Copies the pom.xml file and the source code from the host machine to the working directory in the container.

RUN mvn package -DskipTests
Runs Maven within the container to package the application.
The -DskipTests flag skips running tests during the build process, which can speed up the build if tests are not required at this stage.
Run Stage

FROM amazoncorretto:17
Sets the base image for the runtime stage, which is Amazon Corretto JDK 17.

ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
Sets the environment variable MYAPP_HOME to /opt/myapp.
Sets the working directory inside the container to /opt/myapp.
e
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
Copies the JAR file generated in the build stage to the working directory in the runtime stage.

ENTRYPOINT java -jar myapp.jar

Specifies the command that will be executed when the Docker container starts, which is running the Java application by executing the JAR file myapp.jar.
The multistage build is beneficial here because it allows separation of the build environment (including Maven and dependencies) from the runtime environment (just the JRE). This reduces the final image size by excluding unnecessary build dependencies from the runtime image.

# Docker Compose

There are  2 main commands for Docker Compose: docker-compose build and docker-compose up
Wich are respectively building the image from the docker-compose.yml file and running it.

There is the docker-compose.yml file commented 

version: '3.7'

services:
  backend: ##It's the Student API 
    build:
      context: "C:\\Users\\Coren\\TP1 DEVOPS\\Java2\\simpleapi\\simple-api-student-main\\simple-api-student-main" ##Path of the folder containing the affiliated docker file
    container_name: "my_student_api"  ##Force the name to be my_student_api in order to not break everything 
    networks:
      - app-network ##The network that is linked to everything
    depends_on:
      - database

  database: 
    build:
      context: "C:\\Users\\Coren\\TP1 DEVOPS\\postgres"
    container_name: "my_postgres_container"
    networks:
      - app-network

  httpd:
    build:
      context: "C:\\Users\\Coren\\TP1 DEVOPS\\HTTP"
    ports:
      - "80:80"  ##Port affiliated to the docker compose 
    networks:
      - app-network
    depends_on:
      - backend
      - database

networks:
  app-network: ##This section is telling the network that is use 


The docker compose is running well, let's go the TP 2

## TP 2  DEVOPS










