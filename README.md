# Docker-Learning


"docker run -ti ubuntu:latest bash"
	-ti:	for terminal interactive

==============================
docker commit <container-id>

doker tag <image-id> <new_image_name>

=>

docker commit <container_name> <new_image_name>



==============================

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



==============================

docker exec -ti <container_name> bash

[ctrl + d]
	exit container and kill it


==============================
There is a 'known' typo in following command which is used to make error while executing
docker run --name example -d ubuntu bash -c "lose /etc/password"

use following to debug
docker logs example


docker kill <container_name>
	stop
docker rm <container_name>
	clear

==============================
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
