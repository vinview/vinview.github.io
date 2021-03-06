---
title: "FAQ"
tags:
date: 2020-01-26
excerpt: "Docker FAQ"
---

1. **Can I SSH to a container?**  
**Ans:** Yes (if SSH server is running in your container), but it is not recommended.  
Use 'docker exec' or 'docker attach'  
Refer:   
[Your containers should not run an SSH server](https://jpetazzo.github.io/2014/06/23/docker-ssh-considered-evil/ "Title").  
Here, there is a mention of 'nsenter' tool. But it is not recommended by docker now, since they have introduced 'docker exec' & 'docker attach'

2. **Can I set a hostname for the container?**  
**Ans:** Yes, use --hostname while creating/running the container. i.e. 'docker run --hostname `<hostname>` `<image_name>`'    
  
3.  **Can I assign a static IP to the container?**  
**Ans:** Yes, by creating a new network with subnet IP range & Run 'docker run' with --ip  
Note: 1) On the default bridge network, you cannot assign a static IP 2) When container is not runing (i.e. exited), IP is not assigned  

4. **Can I allocate RAM/memory to a container?**  
**Ans:** You cannot allocate memory, but you can set the memory limit using '--memory' (for ex: docker run --memory="250m" `<image_name>`)    
Note: By default, container will have access to the entire memory of docker host. 

5. **Can I allocate/restrict CPUs to a container?**  
**Ans:** You can allocate specific CPU or specific core of the CPU  
Note: By default, each container’s access to the host machine’s CPU cycles is unlimited  

6. **What are the different states of a container?**    
**Ans:** created, restarting, running, paused, exited, dead  

7. **Do I lose my data when the container exits?**    
**Ans:** No.  
But, when container is removed data will be lost if it is written to writable layer of the container filesystem i.s.o volumes  

8. **Can I remove a paused container from Docker?**  
**Ans:** No  

9. **Where the docker volumes are stored?**  
**Ans:** /var/lib/docker/volumes  

10. **What are the different types of Docker networking driver?**   
**Ans:** Bridge, Host, Overlay, MacVLAN  

11. **With which user container will run? How to add/create users? How to run a container with a different user?**  
**Ans:** Use '--user' (i.e. docker run --user `<user>` `<image_name>`) or add an instruction in the Dockerfile as 'USER `<user>`'  
By default, container will run as 'root'  
Note: Docker daemon will always run from 'root' user. We cannot change this.    

12. **How to find the size of containers, volumes & images?**  
**Ans:** Docker system df -v 

13. **What is the storage capacity of Docker & the conatiners?**  
**Ans:** It depends on the storage driver.  

14. **Why it is recommended to run only one process in a container?**  
**Ans:** 1) Scaling containers horizontally is much easier if the container is isolated to a single function
2) Easy to re-use for other purpose or projects 3) Portable 4) Predictable

15. **Can I specify multiple ENTRYPOINT in the dockerfile?**
**Ans:** Yes, you can. But it will take only one. i.e. the last one you defined

16. **How to check the ENTRYPOINT & CMD of a given docker image?**
**Ans:** docker image inspect <imageid>
  
17. **What is the default log driver for containers**
**Ans:** json-file

18. **Run a container with 'syslog' log driver**
**Ans:** Docker run --log-driver syslog <image>
  
20. **How to see the container logs**
**Ans:** docker logs

21. **How to get the size/diskspace of running containers**
**Ans:** Docker ps -s <container_name>

22. **Which is the default storage driver & how to check what storage driver is in use?**
**Ans:** default storage driver is: *overlay2*
docker info

23. **What is the path of dockerd config file**
**Ans:** /etc/docker/daemon.json

24. **Where the images are stored locally in docker?**
**Ans:** /var/lib/docker/overlay2
  
25. **Does 'FROM scratch' create an image layer?** NO

26. **What command is used to target a specific build stage out of multi stage builds?**
**Ans:** Docker build --target <build-stage-name> -t <image-name>
  
27. **What all logging drivers are mandatory for docker log command to successfully function?**
**Ans:** Journald & json-file
Tere are other logging drivers like *fluentd*, *splunk* etc

28. **What is *docker0*?**
docker0 is a Linux bridge without any real network adapter attached and configured with ip address 172.17.0.1/16

29. **What is the volume driver used by default?** *local*

30. **Docker commit:** Creates a new image from a container’s changes

31. **Docker save:** Save an image to a tar file. (Image -> Tar)
For ex: *docker save busybox >busybox.tar* OR *docker save --output busybox.tar*

32. **Docker load:** load an image from a tar archive or STDIN (Tar -> Image)
	*Docker load < busybox.tar.gz*
	*Docker load --input busybox.tar.gz* 
  
33. **Docker export:** export a container's file system as a tar archive (Container -> Tar)
*Docker export red_panda >latest.tar*
*Docker export --output="latest.tar"*
**NOTE:** Exported container's file system (tar file) contains single layer (It doesn't retain layers). So it cannot be **Loaded**.

34. **Docker import:** Import the contents of a  tarball to create a filesystem image ( Tar -> Image)
	*docker import http://example.com/exampleimage.tgz* (This will create a new untagged image)
	*cat exampleimage.tgz | docker import - exampleimagelocal:new* (import via pipe & STDIN)
	*docker import /path/to/exampleimage.tgz*
  
35. **How to check a hostname of a container?**
**Ans:** docker inspect --format="{{.Config.Hostname}}" <container-id>
  
36. **Dokcer stats:** Display a live stream of container(s) resource usage statistics

37. **Where container logs are available?**
**Ans:** /var/lib/docker/containers/<container-id>/<>.log
  
38. **Docker top:** Display the running processes of a container

39. **What is *containerd*?**
**Ans:** An industry-standard *container runtime* with an emphasis on simplicity, robustness and portability.
- containerd is available as a daemon for Linux and Windows. It manages the complete container lifecycle of its host system, from image transfer and storage to container execution and supervision to low-level storage to network attachments and beyond.
- Containerd was designed to be used by Docker and Kubernetes as well as any other container platform that wants to abstract away syscalls or OS specific functionality to run containers on linux, windows, solaris, or other OSes

40. **What is *runc*?**
 **Ans:** A lightweight universal container runtime, is a command-line tool for spawning and running containers according to the Open Container Initiative (OCI) specification.
 Under the hood, *containerd* uses *runc* to do all the linux work.
 
41. **What is *containerd-shim*?**
 **Ans:** It is the parent process of every container started and it also allows daemon-less containers (e.g. upgrading docker daemon without restarting all your containers).
 
42. **What is *dockerd*?**
 **Ans:** A self-sufficient runtime for containers
 
43. **What is *container runtime*?**
https://www.ianlewis.org/en/container-runtimes-part-1-introduction-container-r
  
44.** what is *docker0*?** 
**Ans:** docker0 is a Linux bridge without any real network adapter attached, and configured with ip address 172.17.0.1/16
- https://developer.ibm.com/recipes/tutorials/networking-your-docker-containers-using-docker0-bridge/
- https://stackoverflow.com/questions/37536687/what-is-the-relation-between-docker0-and-eth0
- https://medium.com/@xiaopeng163/docker-bridge-networking-deep-dive-3e2e0549e8a0

45. **What is *libcontainer*?**
**Ans:** It is the default docker execution environment. It is driver (named native) and a library.
- http://jancorg.github.io/blog/2015/01/03/libcontainer-overview/

46. **What is *Lxc*?**
LXC (Linux Containers) is an operating-system-level virtualization method for running multiple isolated Linux systems (containers) on a control host using a single Linux kernel.
The Linux kernel provides the cgroups functionality that allows limitation and prioritization of resources (CPU, memory, block I/O, network, etc.) without the need for starting any virtual machines, and also namespace isolation functionality that allows complete isolation of an application's view of the operating environment, including process trees, networking, user IDs and mounted file systems.
Early versions of Docker used LXC as the container execution driver, though LXC was made optional in v0.9 and support was dropped in Docker v1.10.

47. **What commands/instruction in Dockerfile creates image layers?**
Only *RUN, COPY & ADD* instructions creates image layers. The other instructions will create intermediate layers and do not influence the size of your image. 

48. **What are *dangling images* & how do you list the dangling images?**
dangling image just means that you've created the new build of the image, but it wasn't given a new name. So the old images you have becomes, the "dangling image". Those old image are the ones that are untagged and displays "<none>" on its name when you run docker images.
*docker images -f dangling=true*

49. **What are unused images**?
Images which are not referenced by any containers.
*docker images prune -a* will remove both unused & dangling images

50. Multistage build
https://docs.docker.com/develop/develop-images/multistage-build/

51. **What is *AUFS*?**
AUFS is a union filesystem. The aufs storage driver was previously the default storage driver used for managing images and layers on Docker for Ubuntu/Debian family.
If your Linux kernel is version 4.0 or higher, and you use Docker Engine - Community, consider using the newer *overlay2*, which has potential performance advantages over the aufs storage driver

52. 
	














 





