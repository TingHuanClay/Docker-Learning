# Docker-Learning

"docker run -ti ubuntu:latest bash"
-ti: for terminal interactive

docker commit <container-id>[:<version-name>]

doker tag <image-id> <new_image_name>

=>

docker commit <container_name> <new_image_name>

docker run --rm -ti ubuntu sleep 5

docker run --rm -ti ubuntu bash -c "sleep 3; echo all done"
--rm: delete the container after leaving it
bash -c: execute multiple commands in the container when it it initialized

docker run -d -ti ubuntu bash
-d: run in background

docker attah <container_name>
docker attah xenodochial_volhard
jump into the container in another terminal

[ctrl + p] + [ctrl + q]
detach and keep running
and you can attach back to the container again

docker exec -ti <container_name> bash

[ctrl + d]
exit container and kill it

There is a 'known' typo in following command which is used to make error while executing
docker run --name example -d ubuntu bash -c "lose /etc/password"

use following to debug
docker logs example

docker kill <container_name>
stop
docker rm <container_name>
clear

run docker with Resource Constraints

    Memory Limits:
    	docker run --memory maximum-allowed -memory <image_name> command


    CPU Limits:
    	docker run --cpu-shares
    		relative to other containers

    	docker run --cpuquota
    		limit in general

Lessons from the Field

    - Don't let your containers fetch dependencies when they start

    - Don't leave important things in unnamed stoppped containers

---

### 2.5 Exposing ports

docker run --rm -ti -p 45678:45678 -p 45679:45679 --name echo-server ubuntu:14.04 bash

    publish 45678 for outside and inside
    publish 45679 for outside and inside

use 'Netcat' for network debugging

    'nc lp 45678 | nc -lp 45679

    	the data will get passed to the system from port 45678 and soit out on port 45679
    	lp: listen on port


    nc localhost 45678


    """
    It's NOT allowed to use ip address in the nc command for the MAC environment
    so we use host.docker.internal
    """

    docker run --rm -ti ubuntu:14.04 bash
    nc host.docker.internal 45678

    docker run --rm -ti ubuntu:14.04 bash
    nc host.docker.internal 45679


    docker port <container_name>
    docker port echo-server


    You can also export ports Dynamically

    terminal A: We don't assigned thr ourtput port and leave it to docker qutomatically
    	docker run --rm -ti -p 45678 -p 45679 --name echo-server ubuntu:14.04 bash
    	nc -lp 45678 | nc -lp 45679

    terminal B:
    	'docker port echo-server
    		output:
    			45678/tcp -> 0.0.0.0:32769
    			45679/tcp -> 0.0.0.0:32768

    	'nc localhost 32769

    terminal C:

    	'nc localhost 32768



    Besides TCP ports, we can also export the UDP ports


    docker run -p <outside-port>:<inside-port>/<protocal (tcp/udp)>
    	e.g.
    		docker run -p 1234:1234/udp


    		terminal A:
    			'docker run --rm -ti -p 45678/udp --name echo-server ubuntu:14.04 bash

    			'nc -ulp 45678

    		terminal B:
    			'docker port echo-server
    				output:
    					45678/udp -> 0.0.0.0:32768

    			'nc -u localhost 32768

### 2.6 Container Networking

docker network ls
docker network create <network-name>
docker network create learning

    	$ docker network ls
    	NETWORK ID          NAME                DRIVER              SCOPE
    	0304d8c3522b        bridge              bridge              local
    	b8cf4536d9e2        host                host                local
    	2acb670254a9        none                null                local
    	clayhsiao ~
    	$ docker network create learning
    	27678977c1d1988c88a10237da0b4b8a4520791f6c59b7f2f9f037a458c3a24b
    	clayhsiao ~
    	$ docker network ls
    	NETWORK ID          NAME                DRIVER              SCOPE
    	0304d8c3522b        bridge              bridge              local
    	b8cf4536d9e2        host                host                local
    	27678977c1d1        learning            bridge              local
    	2acb670254a9        none                null                local

docker run --rm -ti --net learning --name catserver ubuntu:14.04 bash

docker run --rm -ti --net learning --name dogserver ubuntu:14.04 bash

docker network create catsonly

docker network connect catsonly catserver

### 2.7 Legacy Linking

Legacy Linking:
when you use the 'Link' for docker
it's defined only one way
the environment variable of the linked server would be copied to the linking server

Terminal A:
docker run --rm -ti -e SECRET=theinternalnetlovecats --name catserver ubuntu:14.04 bash
'-e SECRET=theinternalnetlovecats':
setting an environment vairable
(name,value) = (SECRET,theinternalnetlovecats)

    nc -lp 1234

Terminal B:
docker run --rm -ti --link catserver --name dogserver ubuntu:14.04 bash

    nc catserver 1234

### 2.8 Images

docker pull

docker push

docker rmi <image-name>:<tag>
docker rmi <image-id>

### 2.9 Volumes

Docker provides Volumes, which is similar to shared folder between containers.

Virtual "discs" would store data in 2 ways
Persistant: the data would still available on the host even the container goes away
Ephemeral: Data would be erased when NO container is using it.

Sharing data between the Host and the container.
similar to the shared folder in virtual machine to share the data with the host.

mkdir example
docker run -ti -v /Users/clayhsiao/example:/shared-folder ubuntu: bash
"-v [folder-name-in-the=host]:[folder-name-in-the-container]"

Sharing data between containers
volumes-from
shared "discs" that exist only as long as they are being used

Terminal A:
'docker run -ti -v /shared-data ubuntu bash
create a shared folder
'echo hello > /shared-data/data-file
create a file in the shared folder

Terminal B:
'docker run -ti --volumes-from <container-name> ubuntu bash

### 2.10 Docker registries

'docker search <keyword>
'docker login

### Quiz

1. In Docker, programs run in containers which are comprised of images .

2. Run this command to see all stopped and running containers.
   docker ps -a

3. A Docker container continues to run until the process that started it exits .

4. Before you can reuse a stopped container's name you must run command.
   docker rm old-container-name

5. To make port 80 inside a container accessible from the internet on port 8080 run docker run -p 8080:80 .

6. What connections are allowed by this Docker command: docker run --link XYZ --name QRS ubuntu bash
   from QRS to XYZ only

7. To use a private network, where must you add the command? add --net=network-name to
   to both the client and server

8. The command docker pull is used to download images while the command docker push is used to upload them.

9. The command "docker run -v volume-name ubuntu bash" creates a volume that is ephemeral and is deleted when no container is using it .

10. What commands are used to upload a public image?
    docker login and docker push

---

## 3. Building Docker images

### 3.1 What are Dockerfiles

    'docker build -t <name-of-result> .
    		Note the '.' here means the location of the dockerfile would be saved in current directory.
    		You can assign specific path to replace the '.'

### 3.2 Building Docker Dockerfiles

    In this lesson, you'll learn how to build a docker filer abd chain them up like inheritance
    	e.g. debian:sid >> example/nanoer

    Besides, You can use the command to define the default action when the container is initialized

    FROM busybox
    RUN echo "Building simple docker image."
    CMD echo "Hello Container"



    FROM debian:sid
    RUN apt-get -y update
    RUN apt-get install nano
    CMD ["/bin/nano", "/tmp/notes"]



    FROM example/nanoer
    ADD  notes.txt /notes.txt
    CMD "nano" "/notes.txt"

### 3.3 Dockerfile Syntax

- FROM statement
  the docker file should always starts with **From**
  e.g.
  'FROM java:8

- MAINTAINER statement
  Define the author of this dockerfile
  e.g.
  'MAINTAINER Firstname Lastname <email@example.com>

- RUN statement
  Runs the command line, waits for it to finish and saves the result
  e.g.
  'RUN unzip install.zip /opt/install/
  or
  'RUN echo hell docker

- ADD Statement 1. Add local files
  e.g.
  'ADD run.sh /run.sh 2. Add the content of tar archives
  e.g.
  'ADD project.tar.gz /install/ 3. Works with URLs as well
  e.g.
  'ADD htttps://project.example.com/download/1.0/project.rpm /project/

- ENV Statement
  Sets environment variables both during the build and when running the result
  e.g.
  'ENV DB_HOST=db.production.example.com
  'ENV DB_PORT=5432

- ENTRYPOINT and CMD Statement
  ENTRYPOINT RUN and CMD can use either **Shell Form** or **Exec Form** 1. Shell Form
  'nano notes.txt 2. Exec Form
  '["bin/nano", "notes.txt"]
  this cause the nano to be run directly but not surrounded with bash to make it more efficiently

- EXPOSE Statement
  map a port into container, similar to '-p 1234:1234
  'EXPOSE 8080

- VOLUME Statement
  Defines shared or ephmeral Volumes
  'VOLUME ["host/path/", "/container/path/"]
  'VOLUME ["/shared-data"]

      		*Avoid defining shared folders in Dockerfiles, due to you might share it someday*

* WORKDIR Statement
  similar to **CD** which will set the directory the container starts in and in the build
  'WORKDIR /install/

- USER Statement
  Set whihch user the container will run as
  'USER clayhsiao
  'USER 1000

### 3.4 Multi-Project Docker files

You can use multiple FROM to split the docker file for quick daily build and a complete release build
`
FROM ubuntu:16.04 as builder
RUN apt-get update
RUN apt-get -y install curl
RUN curl https://google.com | wc -c > google-size

FROM alpine
COPY --from=builder /google-size /google-size
ENTRYPOINT echo google is this big; cat google-size
`

### 3.5 Avoid Golden Images

- include installers in your project
- Have a canonical build that builds everything completely from scratch
- Tag you builds with git hash of that code that code that built it
- Use small base images, such as Alpine
- Build images you share publicly from Dockerfiles, always

### Quiz

- Each line of a docker file makes a new independent image based on the previous line's image .

- What does the **CMD** command in a Docker file do?
  A: it sets the program to run when the container starts

- The Docker file **RUN** command starts a program that runs only for one line of the Docker file.

- Where does the Docker file **ENV** command set environment variables?
  A: both in the rest of the Docker file, and in the finished image

- The Docker file **WORKDIR** command changes directories both for the rest of the Docker file, and in the finished image .
