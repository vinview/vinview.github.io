---
title: "Docker Swarm"
tags:
date: 2020-01-o2
excerpt: "Docker Swarm"
---
On the Master: gateway, registry & service descovery services  
Gateway-> Nginx is the default Notary  
Registry-> Ingress  is for querying the services  
Service descovery-> Stack is for giving info about services  

docker-machine create -d virtualbox master  
docker-machine create -d virtualbox reachable  
docker-machine create -d virtualbox worker1  
docker-machine create -d virtualbox worker2  
docker-machine ls  

**Step0:**  
docker-machine env master  
eval $("C:\Program Files\Docker Toolbox\docker-machine.exe" env master)  
docker-machine active  

docker-machine env reachable  
eval $("C:\Program Files\Docker Toolbox\docker-machine.exe" env reachable)  
docker-machine active  

docker-machine env worker1  
eval $("C:\Program Files\Docker Toolbox\docker-machine.exe" env worker1)  
docker-machine active  

docker-machine ip master -> to get the IP of Master (192.168.99.101)  

**Step1: Master machine announcement**    
Note: All docker orchestration are done from master terminal only  
docker-machine ssh master "docker swarm init --advertise-addr 192.168.99.101"   
docker network ls -> it will list a network called 'ingress'  
docker inspect ingress  
docker-machine ls -> this will not tell whether it's part of SWARM/cluster in community edition  
Docker node ls -> to get the cluster details  

**Step2: generate token & save**    
docker-machine ssh master "docker swarm join-token manager" -> get the reachable token  
docker-machine ssh master "docker swarm join-token worker" -> get the worker token  

**Step3: Connect reachable**  
docker-machine ssh reachable "docker swarm join --token  SWMTKN-1-39qfr5t6kkncs26kkdcfls03jvzznhck2xl4aqzv3q3ruufw61-2hey69drql9lkqykc7zdjqrq4 192.168.99.101:2377"  
Docker node ls -> will show master as leader & reachable node as reachable  
From the reachable terminal, if you run 'docker network ls' it will list all the n/ws. i.e. All master capabilities are available here  
From worker terminal - 'docker network ls' & 'docker node ls' will not show/ will not run. Do not have all capabilities  

**Step4: Connect worker1/2**   
(run from master terminal)  
docker-machine ssh worker1 "docker swarm join --token SWMTKN-1-39qfr5t6kkncs26kkdcfls03jvzznhck2xl4aqzv3q3ruufw61-c2snkhoaip3nfyyu5go3rr8u6 192.168.99.101:2377"  
docker-machine ssh worker2 "docker swarm join --token SWMTKN-1-39qfr5t6kkncs26kkdcfls03jvzznhck2xl4aqzv3q3ruufw61-c2snkhoaip3nfyyu5go3rr8u6 192.168.99.101:2377"  
Docker node ls -> should list all the machines  

**Step 5: Health check**    
Docker inspect ingress -> should list all the 4 machines  
Note: In the replica creation, 'limit:' is for set of containers & 'reservations:' is for DM  
Say, if you have 1GB of RAM in DM & available RAM is 500MB  
Create a docker-compose.yml file from [Docker Compose file]](https://github.com/vinaydhegde/DockerStuff/blob/master/Swarm%20docker-compose)    
docker stack deploy -c docker-compose.yml <name of stack>  
docker stack deploy -c docker-compose.yml swarmstack  
docker network ls  
docker service ls  
Docker ps -> should list the containers in master. Only one conatiner will be there  
docker stack ps <name of stack>  
docker stack ps swarmstack -> should list all the containers & we can see who(which node) is running which container  
docker-machine ssh worker1 "docker swarm leave --force" -> remove worker1 node from the stack  
docker node ls -> will show that 'worker1' node is down  
docker stack ps swarmstack -> will show that containers in worker1 is moved to worker2 node  


Come back to default VM: docker-machine env default  
docker-machine rm master reachable worker1 worker2  

