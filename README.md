# Short introduction to docker

## Basic concepts

* ¿What is *docker*? A *container* management system
* ¿What are *containers*? We can think in a container as a small and very efficient virtual machine
* ¿Why are they *efficient*? Because they share the kernel of the host machine and use some mechanisms for isolating resources (cgroups and namespaces), like processes, memory, users, network interfaces, etc.

|![Container vs. virtual machine](https://i1.wp.com/www.docker.com/blog/wp-content/uploads/Blog.-Are-containers-..VM-Image-1-1024x435.png?ssl=1)|
|:--:|
|Containers (left) vd. virtual machines (right).<br/>*Source: Jenny Fong (https://www.docker.com/blog/containers-replacing-virtual-machines/)*|


* Advantages:
  * The running environment is prepared once and run many times
  * Isolation avoids configuration problems (multiple-versions of the same dependencies)
  * Many containers ready to use (e.g., in [DockerHub](https://hub.docker.com))
  * Security: if the system is compromised, the container can be destroyed and regenerated
  * Easy backups of the complete environment
  * Testing apps in multiple running environments
  * It allows easy scaling of Web services
* Drawbacks:
  * CPU and storage requirements
  * Slight increase in runtime of the app (specially if it is not using the same kernel)
  * Learning curve: not appropriate for small apps if your development environment is alread configured
  * All UI must be through command line or Web (no desktop UI).

* *Image*: An image is a *container template*. There can be many running containers based on the same image. The relationship between container and image is the same as process and program (or object and class).

## Basic commands

You can use your own docker installation to follow the tutorial or go to [play with docker](https://labs.play-with-docker.com) if you don't want to install anything.

### Container management

* Create and run a new container (alpine is the name of the *image*, a 5MB Linux distribution):
```
docker run -it alpine
```
The `-i` option connects the standard input of the shell with teh standard input of the container. The `-t` option creates a (virtual) terminal to the standard input and output of the docker process to allow easy interaction with the user.

* Check that the container still exists after exiting it:
```
docker container ls -a
```
Without option `-a`, the `ls` command only returns running containers. Example:
```
docker run -d --name myweb httpd:alpine
docker container ls
```
We can assign names to containers using `--name` when creating it (otherwise, a random name is created by the docker engine). We can run it as a background process using `-d` (otherwise, the shell wil wait until the container stops).
* Stop a container using:
```
docker container stop myweb
```
* Start a container using:
```
docker container start myweb
```
All containers have a unique ID in addition to the name. We can use this ID also to manage the container (in `rm`, `stop`, `start`, etc.)
* Let's see the log (standard output) of the container:
```
docker container logs -f myweb
```
The `-f` option makes the docker command to wait for new content. Without this option the current log is shown and the command ends.
* Let's remove all the containers using:
```
docker container rm myweb
```
* We can run as many containers based on the same image as we want:
```
for i in `seq 1 3`; do docker run -d httpd:alpine; done
```
* When we stop these containers they still exist, becasue they keep come *state* (a whole filesystem), and they requires disk space. If we are not interested in the state (changes in the internal filesystem) we can get rid of the containers when they stop. We can do this with the `--rm` option:
```
docker run --rm -it alpine
docker container ls
```

### Image management

* We can see the images downloaded in our docker engine with:
```
docker image ls
```
* We can remove an image with:
```
docker image rm alpine
```

### 

Comandos básicos para gestión de contenedores: docker run, docker container (ls, rm)
- Run - - rm (ejemplo hola mundo ver qué pasa si no se pone rm)
- Docker container ls
- Run - i t

* Conceptos: contenedor, imagen, volumen

Añadiendo volúmenes: 
- Volúmenes: -v
- Con nombre, enlazando a sistemas de ficheros, o ficheros concretos
- Docker volume (ls, rm, create)

Interacción por Web
- Puertos: -p

Creando receta para contenedor
- Dockerfile (ejemplo sencillo que yo tenga con App)
- Docker build
- Docker logs
- Publicar contenedores, en Docker hub y github

Docker-compose:
- Docker no está pensado para meter todo en el mismo contenedor. Perdemos muchas de las ventajas
- Se pueden combinar contenedor en redes virtuales: docker-compose
- Mostrar ejemplos: web-neo, EWP, 
- Comando docker-compose (up, down, start, stop, logs)

Cómo dockerizar mi sistema
- Ejemplo práctico con Web Wordpress: tiene archivos (volumen) base de datos, y dos contenedores, añadir proxy y aprovechar para explicar el alias y DN en redes internas

Más allá: docker swarm y kubernetes

