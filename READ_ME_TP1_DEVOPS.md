# Corentin DOUCHE - DEVOPS TP1

## TP1 PART 1 - DOCKER 

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

## TP 1 PART 2 - DEVOPS

Testcontainers is a Java library that provides lightweight, disposable instances of common databases, Selenium web browsers, or anything else that can run in a Docker container. It is primarily used for integration testing and makes it easier to manage dependencies and configurations for tests that require external resources.

2.2 Honnestly, we struggled a bit to do the main.yml working nicely.

there is final version of it 

name: CI devops 2024
on:
  #to begin you want to launch this job in main and develop
  push:
    branches: 
     - main
     - develop 
  pull_request:

jobs:
  test-backend: 
    runs-on: ubuntu-22.04
    steps:
     #checkout your github code using actions/checkout@v2.5.0
      - uses: actions/checkout@v2.5.0

     #do the same with another action (actions/setup-java@v3) that enable to setup jdk 17
      - name: Set up JDK 17
        uses: actions/setup-java@v3  
        with:
          java-version: '17'
          distribution: 'adopt'      ##Here we add some specificity to make everything work
     
     #finally build your app with the latest command
      - name: Build and test with Maven
        run: |
          cd Java2/simpleapi/simple-api-student-main/simple-api-student-main ##The path was hard to put
          mvn clean verify


Now we added a names to  the 3 images that we want to do, we also add the changment,connecting to docker hub using the token that we put into the secrets of the github
The docker repo have the 3 new images, so we know it's working.

Now let's do the Sonar Cloud thing.
1- Create a Sonar Cloud Account
2- Create an organization and keep the key
3- Create a project in this organization and keep the key 
4-link the project to our main.yml using the github secrets variable
5- We can see that the Sonar Cloud is working, as my report is here on sonar cloud. Tho I failed to pass the test to be a clean code.


For the bonus, we splitted the main.yml in two parts, so it is doing the ttest-backend at first, and then build the images and push them if it's compiling everything.
This is due to the on command of the two .yml files.

## PART 3 : ANSIBLE SERVER 

Inventory
The inventory file for this Ansible project is located at TP1 DEVOPS/ansible/inventories/setup.yml. This file defines the hosts and groups that Ansible will manage, along with any variables associated with them.

Comments:
The inventory file organizes hosts into groups and sets variables for them.
Hosts can be grouped by their roles or environments.
Variables can be set at both the group and host level to customize Ansible behavior.
Base Commands
Test Inventory with Ping Command
The ping module verifies connectivity to the hosts defined in the inventory file. It's a simple way to check if Ansible can communicate with the hosts.

Gather Facts
The setup module collects facts about the hosts, such as OS distribution, network interfaces, and hardware details. Facts are useful for understanding the environment before making changes.

Remove Apache HTTP Server
The yum module can uninstall packages from the hosts. This command ensures that the specified package is absent from the servers.

Comments:
Base commands allow you to interact with hosts defined in the inventory.
Commands can be executed to test connectivity, gather information, and perform tasks on remote hosts.
Understanding how to use base commands is essential for managing hosts effectively with Ansible.

Here is the advanced_playbook commented 

- hosts: all  # Specifies that the tasks in this playbook will be applied to all hosts defined in the inventory.
  gather_facts: false  # Disables the gathering of facts for each host to improve playbook performance.
  become: true  # Activates privilege escalation, allowing Ansible to execute tasks with root privileges using sudo.

# Install Docker
  tasks:

  - name: Install device-mapper-persistent-data
    yum:
      name: device-mapper-persistent-data
      state: latest  # Ensures that the package is installed to the latest version available.

  - name: Install lvm2
    yum:
      name: lvm2
      state: latest  # Installs the lvm2 package to manage logical volumes.

  - name: add repo docker
    command:
      cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo  # Adds the Docker repository to the system.

  - name: Install Docker
    yum:
      name: docker-ce
      state: present  # Ensures that the docker-ce package is present on the system.

  - name: Install python3
    yum:
      name: python3
      state: present  # Ensures that Python 3 is available on the system.

  - name: Install docker with Python 3
    pip:
      name: docker
      executable: pip3
    vars:
      ansible_python_interpreter: /usr/bin/python3  # Installs the docker Python package with Python 3.

  - name: Make sure Docker is running
    service: name=docker state=started  # Ensures that the Docker service is running on the hosts.
    tags: docker  # Tags this task for easy identification.








