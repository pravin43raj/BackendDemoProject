# BackendDemoProject
#######Dockerfile for Nodejs application.

FROM node:14

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies

COPY package*.json ./

RUN npm install

# Bundle app source
COPY . .

EXPOSE 51005
CMD [ "node", "index.js" ]

###################################################################
Steps
1) Create the GitHub repository called BackendDemoProject.
2) Add the source code in it.
3) Add the Above Dockerfile in the repo.
4) Decide the deployment location (eg. /iauro/project)
5) Go to the deployment localtion and create the "docker-compose.yml" file
6) cd /iauro/project
7) nano docker-compose.yml
8) Add the following code into it.

---

version: '3.1'

services:
  nodejsapp:
    image: nodejsapp
    restart: always
    container_name: nodejsapp
    network_mode: "host"
    depends_on:
      - "mysqldb"
      - "mongodb"
    ports:
      - "51005:51005"

  mysqldb:
    image: mysql
    command: --default-authentication-plugin=mysql_native_password
    container_name: mysqldb
    restart: always
    network_mode: "host"
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
    volumes:
      - ./docker/mysqldata:/var/lib/mysql
    env_file:
      - .env
    ports:
      - "3306:3306"

  mongodb:
    image: mongo:latest
    container_name: mongodb
    restart: always
    network_mode: "host"
    environment:
      MONGO_INITDB_DATABASE: ${MONGO_INITDB_DATABASE}
    volumes:
      - ./docker/mongodb/mongod.conf:/etc/mongod.conf
      - ./docker/mongodb/initdb.d/:/docker-entrypoint-initdb.d/
      - ./docker/mongodb/data/db/:/data/db/
    env_file:
      - .env
    ports:
      - "27017:27017"
    command: ["-f", "/etc/mongod.conf"] 

9) we have to create the docker directory

   docker
       .env  //for storing the env. variables values
       mysqldata  //for mysql database
         
       mongodb  //for mongodb
          mongod.conf //mongodb configuration file
          initdb.d
          data
             db  // it stores mongodb databases.

    #########content of .env file
  
    MONGO_INITDB_DATABASE=backend_demo
    MYSQL_ROOT_PASSWORD=password
    MYSQL_DATABASE=backend_demo
    MONGO_INITDB_USERNAME=user1
    MONGO_INITDB_PASSWORD=user1
    MONGO_INITDB_ROOT_USERNAME=root
    MONGO_INITDB_ROOT_PASSWORD=root

10) Create jenkins job for build and deploy 
    (The jenkins job should be executed after every commit is made)
    for this we need to add webhook for repository.
    go to the settings->webhooks and click on add webhook button.
   
    1) Payload URL
       http://<server ip>:8088/github-webhook/
    2) Content type
       application/json

    3) for which event trigger this webhook
       select option =Just the push event

    4) Click on add webhook button 

11) create jenkins job (free style Project)
    
    
    1)Source code management (section)
       in this select the source code mangement "git"
       specify the repository URL and credentials.
      next specify the branch to build.

    2)Build triggers (section)
      tick on "GitHub hook trigger for GITScm polling"
    
    3) Build (section)
       In this click on add build steps and select "Execute shell"
       and add following commands in it.

       #!/bin/bash
       docker build -t nodejsapp .

       cd /iauro/project
       docker-compose down -v  //to remove the running containers
       docker-compose up -d   // to start the containers.

12) Open the browser and type the url: localhost:51005
   

   
