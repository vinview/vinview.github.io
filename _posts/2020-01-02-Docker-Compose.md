---
title: "Docker Compose"
date: 2020-01-02
header:
  teaser: /assets/images/nature.jpg
excerpt: "Some instruction and commands in Docker Compose"
---

Docker versions: 1-12, 13-16, 17++  
Docker compose version1 = Docker version 1-12  
Docker compose version2 = Docker version 13-16  
Docker compose version3 = Docker version 17++  

Fast track DIS(Docker Images) management is achieved using Dockerfile  
Fast track service management is achieved using YAML  

| XML                          | JSON                               | YAML                               |
|------------------------------|------------------------------------|------------------------------------|
| DSL (Data serialization Lang | DSL + DAL (Data   Automation Lang) | DSL + DAL (Data   Automation Lang) |

.yaml-> for App automation  
.yml -> Infra auto  

Create a docker compose file locally (docker-compose.yml) from  
[https://github.com/vinaydhegde/DockerStuff/blob/master/Example1-docker-compose.yml]  

Run 'docker-compose up' command  
docker network ls -> it will list a new network called <CurrentDir>-default. For ex: dockercompose_default.  
This is because, we are using version3 & it creates a new network for every project  

docker exec -it dockercompose_databases_1 bash  
Mysql -u root -p  

In case SOA model, 'network' is not used. We use 'link'    
Below are the example for 'link' with docker-compose:  
[https://github.com/vinaydhegde/DockerStuff/blob/master/Example1-docker-compose.yml]  
[https://github.com/vinaydhegde/DockerStuff/blob/master/Example2-docker-compose.yml]  
[https://github.com/vinaydhegde/DockerStuff/blob/master/Example3-docker-compose.yml]  

docker-compose.override.yml -> is used to re-create containers in an incremental/iteration way.  
i.e. it will refer both docker-compose & docker-compose.override.yml to recreate containers. docker-compose.override.yml  doesn't contain image names  

docker-compose up -d  
docker exec -it dockercompose2_web_1 bash  
docker exec -it dockercompose2_web_1 bash  
docker exec -it dockercompose2_web_1 bash  
docker-compose -f docker-compose.yml -f docker-compose.override.yml -f docker-compose.prod.yml up -d  






