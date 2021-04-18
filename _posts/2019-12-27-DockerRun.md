---
layout: post
title: "Docker Run"
date: 2019-12-27
---

- Docker runs processes in isolated containers.  
- A container is a process which runs on a host.  
When an operator executes ‘docker run’, the container process that runs is, isolated in that it has its own file system, its own networking, and its own isolated process tree separate from the host.  
 
Docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG..]  
Detached(-d)  
By design, containers started in detached mode exit when the root process used to run the container exits, unless you also specify the --rm option  
Do not pass a service x start command to a detached container  
To do input/output with a detached container use network connections or shared volumes. These are required because the container is no longer listening to the command line where docker run was run.  
To reattach to a detached container, use docker attach command.  
For example: docker run -d -p 80:80 my_image nginx -g 'daemon off;'   

**Foreground**  
 docker run will start the process in the container and attach the console to the process’s standard input, output, and standard error  
For example: docker run -a stdin -a stdout -i -t ubuntu /bin/bash  

**Container Identification(--name):**    
If you specify a name, you can use it when referencing the container within a Docker network  
Containers connected to the default bridge network can communicate with each other by IP address. Docker does not support automatic service discovery on the default bridge network. If you want containers to be able to resolve IP addresses by container name, you should use user-defined networks instead. You can link two containers together using the legacy docker run --link option, but this is not recommended in most cases  
For example: docker run –name my_alpine alpine  

**PID settings(--pid)**  
By default, all containers have the PID namespace enabled.  
PID namespace provides separation of processes. The PID Namespace removes the view of the system processes, and allows process ids to be reused including pid 1  
For example, to allow processes within the container to see all of the processes on the system:  
docker run -it --rm --pid=host alpine /bin/sh  

**UTS settings(--uts)**  
The UTS namespace is for setting the hostname and the domain that is visible to running processes in that namespace  
By default, all containers, including those with --network=host, have their own UTS namespace. The host setting will result in the container using the same UTS namespace as the host.  
For example: docker run –it –uts=host alpine /bin/sh (In this case, container will have same hostname as the host machine)  

**IPC settings(--ipc)**  
--ipc="MODE" : Set the IPC mode for the container  
Mode can be “”, none, private, shareable, container:<container_id>, host  
IPC namespace provides separation of named shared memory segments, semaphores and message queues.  
For example:  docker run –it ipc=host alpine /bin/sh (to use host system’s IPC namespace)  

**Network settings**  
--dns  
container will use the same DNS servers as the host by default, but you can override this with –dns  
--network  
By default, all containers have networking enabled and they can make any outgoing connections  
We can completely disable networking with docker run --network none  
--network bridge  
A bridge is setup on the host, commonly named docker0, and a pair of veth interfaces will be created for the container. One side of the veth pair will remain on the host attached to the bridge while the other side of the pair will be placed inside the container’s namespaces in addition to the loopback interface. An IP address will be allocated for containers on the bridge’s network and traffic will be routed though this bridge to the container.  
--network host  
container will share the host’s network stack and all interfaces from the host will be available to the container. The container’s hostname will match the hostname on the host system  
--network:container  
Container will share the network stack of another container  
User defined network:  
You can create a network using a Docker network driver or an external network driver plugin  
For example:  
docker network create -d bridge my-net   
docker run --network=my-net -itd --name=container3 busybox  

**Managing /etc/hosts:**  
--add-host flag can be used to add additional lines to /etc/hosts  
For example: docker run -it --add-host db-static:86.75.30.9 ubuntu cat /etc/hosts   

**Restart Policies(--restart):**  
We can specify how a container should or should not be restarted on exit.  
(Options are: no, on-failure[:max_retries], always, unless-stopped)  
For example: docker run --restart=always alpine  

**Exit Status:**  
The exit code from docker run gives information about why the container failed to run or why it exited.  

**Cleanup (--rm):**  
By default a container’s file system persists even after the container exits.  
We can automatically remove the container when it exits using –rm  

**Runtime constraints on resources:**  
-m or –memory (memory limit)  
--memory-swap (Total memory limit: memory + swap)  
--kernel-memory  
-c or –cpu-shares  
--cpus=0.000: Number of CPUs. Number is a fractional number. 0.000 means no limit.  

**User memory constraints:**  
memory=inf, memory-swap=inf (default): There is no memory limit for the container. The container can use as much memory as needed.  
memory=L<inf, memory-swap=inf: The container is not allowed to use more than L bytes of memory, but can use as much swap as is needed  
memory=L<inf, memory-swap=2*L: The container is not allowed to use more than L bytes of memory, swap plus memory usage is double of that  
memory=L<inf, memory-swap=S<inf, L<=S: The container is not allowed to use more than L bytes of memory, swap plus memory usage is limited by S.  
Examples:  
docker run -it ubuntu:14.04 /bin/bash (container can use as much memory and swap memory as they need)  
docker run -it -m 300M --memory-swap -1 ubuntu:14.04 /bin/bash (the processes in the container can use 300M memory and as much swap memory as they need)  
docker run -it -m 300M ubuntu:14.04 /bin/bash (the processes in the container can use 300M memory and 300M swap memory )  
docker run -it -m 300M --memory-swap 1G ubuntu:14.04 /bin/bash (the processes in the container can use 300M memory and 700M swap memory)  

**CPU share constraints:**  
By default, all containers get the same proportion of CPU cycles. This proportion can be modified by changing the container’s CPU share weighting relative to the weighting of all other running containers.  
To modify the proportion from the default of 1024, use the -c or --cpu-shares flag to set the weighting to 2 or higher  
The proportion will only apply when CPU-intensive processes are running. When tasks in one container are idle, other containers can use the left-over CPU time. The actual amount of CPU time will vary depending on the number of containers running on the system  
For example, consider three containers, one has a cpu-share of 1024 and two others have a cpu-share setting of 512. When processes in all three containers attempt to use 100% of CPU, the first container would receive 50% of the total CPU time. If you add a fourth container with a cpu-share of 1024, the first container only gets 33% of the CPU. The remaining containers receive 16.5%, 16.5% and 33% of the CPU  

**EXPOSE (incoming ports):**  
--expose=[]: Expose a port or a range of ports inside the container.  
             These are additional to those exposed by the `EXPOSE` instruction  
-P         : Publish all exposed ports to the host interfaces  
-p=[]      : Publish a container's port or a range of ports to the host  

For example: docker run -d --name staticwebserver -p 8000:80 nginx  
docker port staticwebserver  
curl localhost:8000  
From the browser: http://192.168.99.100:8000  

docker run -d --name dynamicwebserver -P nginx (It's called dynamic port. It uses any of the port from the range of ports exposed in docker image or during docker run)  




