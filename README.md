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

* *Volume*: Storage unit, path in the host file system or temporal filesystem that we can mount at any point in a container.

## Basic commands

You can use your own docker installation to follow the tutorial or go to [play with docker](https://labs.play-with-docker.com) if you don't want to install anything.

### Container management

Create and run a new container (alpine is the name of the *image*, a 5MB Linux distribution):
```
docker run -it alpine
```
The `-i` option connects the standard input of the shell with teh standard input of the container. The `-t` option creates a (virtual) terminal to the standard input and output of the docker process to allow easy interaction with the user.

Check that the container still exists after exiting it:
```
docker container ls -a
```
Without option `-a`, the `ls` command only returns running containers. Example:
```
docker run -d --name myweb httpd:alpine
docker container ls
```
We can assign names to containers using `--name` when creating it (otherwise, a random name is created by the docker engine). We can run it as a background process using `-d` (otherwise, the shell wil wait until the container stops).

Stop a container using:
```
docker container stop myweb
```

Start a container using:
```
docker container start myweb
```
All containers have a unique ID in addition to the name. We can use this ID also to manage the container (in `rm`, `stop`, `start`, etc.)

Let's see the log (standard output) of the container:
```
docker container logs -f myweb
```
The `-f` option makes the docker command to wait for new content. Without this option the current log is shown and the command ends.

Let's remove all the containers using:
```
docker container rm myweb
```

We can run as many containers based on the same image as we want:
```
for i in `seq 1 3`; do docker run -d httpd:alpine; done
```

When we stop these containers they still exist, becasue they keep come *state* (a whole filesystem), and they requires disk space. If we are not interested in the state (changes in the internal filesystem) we can get rid of the containers when they stop. We can do this with the `--rm` option:
```
docker run --rm -it alpine
```
```
docker container ls
```

### Port mapping

Containers have a network namespace completely separated from the host system. We can *bind* ports in the host system with ports in the container. This way we can map services in the container to different ports of our host machine.

We can use option `-p` to do such port bindings:

```
docker run --rm -d -p 80:80 httpd:alpine
```

Observe the name of the image: `httpd:alpine`. The word before the colon is the `name` of the image, while the word after the colon is the `tag`. In this case we are downloading an image of an HTTP server based on the alpine distribution. Tags ease marking the image variants of well-known services.

### Volume management

The file system of the containers and the host system are separated. We can mount virtual storage units, called `volumes` to any path in the container. Volumes' lifecycle is different from containers' lifecycle: they survive container removal.

Let's create a volume:
```
docker create volume myapp
```

Let's mount that volume in a docker container with option `-v`:
```
docker run --rm -it -v myapp:/app alpine
```
The previous command mounts the volume called `myapp` in `/app`. Let's see what is there:
```
ls /app
```
We can see that it is empty (what did you expect?).

Let's add a file to the `/app` folder:
```
cat > /app/file.txt << EOF
hello world
EOF
```

Once we exit and remove the container, the volume persist and we can mount it at anypoint in another container:

```
docker run --rm -it -v myapp:/data alpine
ls /data
cat /data/file.txt
```
In the previous example we can see the file that we created in a new mount point of this new container.

An alternative to docker volumes is to *bind* a file or directory in the host filesystem to a mount point in the container:
```
docker run --rm -it -v $(pwd):/app alpine
cat > /app/file.txt << EOF
hello world
EOF
exit
```

The format to specify a bind mount is `-v <host-path>:<container-path>`. In the previous example, the host path is provided by the `pwd` command to mount the current directory in `/app`. If we now list the files in the current drectory we will find `file.txt`:
```
ls
```

We can use *bind mounts* to easily backup a docker volume:
```
docker run --rm -v $(pwd):/backup -v myapp:/data alpine sh -c "tar czf /backup/archive.tgz -C /data ."
```

We can list all the volumes managed by docker with:
```
docker volume ls
```
We can remove a volume using:
```
docker volume rm myapp
```

### Image management

We can see the images downloaded in our docker engine with:
```
docker image ls
```
We can remove an image with:
```
docker image rm alpine
```

We can download an image without running a container based on it with:
```
docker pull alpine
```

All these images we are working with come from [DockerHub](https://hub.docker.com), a repository of Docker images. In particular, we have used only *official images*. Non-official images can be uploaded to DockerHub (previous registration) and are also available to any Docker user. In order to donwload them we must prepend the DockerHub username to the image name:
```
docker pull jfrchicanog/graybox
```

We can also build our own Docker images. The first step is to create a `Dockerfile` that contains the base image, a set of instructions to prepare the filesystem with the appropriate files and some metadata. One example follows:

```
FROM httpd:alpine
COPY index.html /usr/local/apache2/htdocs
```

Let's download the [Dockerfile](https://github.com/NEO-Research-Group/docker-101/blob/4ab32b2ae452c6debc8c261221574e023951ffd5/Dockerfile) and the [index.html](https://github.com/NEO-Research-Group/docker-101/blob/26831f58f4397201effa340cbb9758190a51aa2d/index.html) file.

### Guía

Creando receta para contenedor
- Dockerfile (ejemplo sencillo que yo tenga con App)
- Docker build
- Publicar contenedores, en Docker hub y github

Docker-compose:
- Docker no está pensado para meter todo en el mismo contenedor. Perdemos muchas de las ventajas
- Se pueden combinar contenedor en redes virtuales: docker-compose
- Mostrar ejemplos: web-neo, EWP, 
- Comando docker-compose (up, down, start, stop, logs)

Cómo dockerizar mi sistema
- Ejemplo práctico con Web Wordpress: tiene archivos (volumen) base de datos, y dos contenedores, añadir proxy y aprovechar para explicar el alias y DN en redes internas

Más allá: docker swarm y kubernetes

