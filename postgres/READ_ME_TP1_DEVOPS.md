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

