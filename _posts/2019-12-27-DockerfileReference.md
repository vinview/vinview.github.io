---
title: "Dockerfile Reference"
toc: true
toc_label: "Content"
toc_sticky: true
tags:
  - Dockerfile Reference

date: 2019-12-27
header:
  teaser: /assets/images/nature.jpg
excerpt: "Dockerfile reference document"
---

A Dockerfile must begin with a `FROM` instruction.

Instructions are not case sensitive.

**Parser directives:**  
Parser directives are optional  
Parser directives do not add layers to the build  
#directive =  value  

**Environmnet variables:**  
Environment variables are declared with ENV statement  
For example:  
ENV abc=hello  
ENV foo bar  
Environment variables are accessed with ${variable_name} or $variable_name  
Environment variables are supported by the following list of instructions.  
ADD,COPY,ENV,EXPOSE,FROM,LABEL,STOPSIGNAL,USER,VOLUME,WORKDIR  

**.dockerignore file**  
Similar to .gitignore  

**FROM:**  
The FROM instruction initializes a new build stage and sets the Base Image for subsequent instructions.  
FROM <image> [AS <name>]   

**RUN:**    
RUN has 2 forms:    
RUN <command> (shell form, the command is run in a shell, which by default is /bin/sh -c on Linux or cmd /S /C on Windows)   
RUN ["executable", "param1", "param2"] (exec form)  
The RUN instruction will execute any commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the Dockerfile.  

**CMD:**  
The CMD instruction has three forms:  
CMD ["executable","param1","param2"] (exec form, this is the preferred form)   
CMD ["param1","param2"] (as default parameters to ENTRYPOINT)   
CMD command param1 param2 (shell form)   
There can only be one CMD instruction in a Dockerfile. If you list more than one CMD then only the last CMD will take effect.  
The main purpose of a CMD is to provide defaults for an executing container.  
Note: Don’t confuse RUN with CMD. RUN actually runs a command and commits the result; CMD does not execute anything at build time, but specifies the intended command for the image.  

**LABEL:**  
LABEL <key>=<value> <key>=<value> <key>=<value> ...  
The LABEL instruction adds metadata to an image.   
A LABEL is a key-value pair.  

**MAINTAINER:**  
MAINTAINER <name>  
The MAINTAINER instruction sets the Author field of the generated images   

**EXPOSE:**  
EXPOSE <port> [<port>/<protocol>..]  
The EXPOSE instruction informs Docker that the container listens on the specified network ports at runtime.  

**ADD:**  
ADD [--chown=<user>:<group>] <src>,… <dest>  
The ADD instruction copies new files, directories or remote file URLs from <src> and adds them to the filesystem of the image at the path <dest>   

**COPY:**  
COPY [--chown=<user>:<group>] <src>,… <dest>  
The COPY instruction copies new files or directories from <src> and adds them to the filesystem of the container at the path <dest>.   

**ENTRYPOINT:**  
ENTRYPOINT has two forms  
ENTRYPOINT [“executable”, “param1”, “param2”] (exec form)  
ENTRYPOINT command param1 param2 (shell form)  
Note: Dockerfile should specify at least one of CMD or ENTRYPOINT commands. 
![EntryPointnCmd](/images/EntryPointnCmd.jpg){:class="img-responsive"}   
 
**VOLUME:**  
VOLUME [“/data”]  
The VOLUME instruction creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers
![volume-example](/images/volume-example.bmp){:class="img-responsive"}    
Create a Volume & mount it from Docker CLI:  
docker volume create my-vol  
docker volume ls  
docker volume inspect my-vol  
docker run -d --name devtest --mount source=myvol,target=/app nginx:latest  

**USER:**  
USER <user>[:<group>]  
USER <UID>[:GID>]  
The USER instruction sets the username & optionally the user group to use when running the image and for any RUN, CMD & ENTRYPOINT instructions that follow it in the Dockerfile.  

**WORKDIR:**  
WORKDIR /path/to/workdir  
The WORKDIR instruction sets the working directory for any RUN, CMD, ENTRYPOINT, COPY and ADD instructions that follow it in the Dockerfile  

**ARG:**  
ARG <name>[=<default value>]  
The ARG instruction defines a variable that users can pass at build-time to the builder with the docker build command using the --build-arg <varname>=<value> flag. If a user specifies a build argument that was not defined in the Dockerfile, the build outputs a warning   
Predefined ARGS are: HTTP_PROXY, HTTPS_PROXY, FTP_PROXY & NO_PROXY  

**ONBUILD:**  
ONBUILD [INSTRUCTION]  
The ONBUILD instruction adds to the image a trigger instruction to be executed at a later time, when the image is used as the base for another build.   

**STOPSIGNAL:**  
STOPSIGNAL signal  

**HEALTHCHECK:**  
The HEALTHCHECK instruction has 2 forms  
HEALTHCHECK [OPTIONS] CMD command (check container health by running a command inside the container) 
HEALTHCHECK NONE (disable any healthcheck inherited from the base image)  

**SHELL:**  
SHELL [“executable”, “parameters”]  
The SHELL instruction allows the default shell used for the shell form of commands to be overridden.   


