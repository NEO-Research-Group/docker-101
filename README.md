# Breve introducción a docker

## Conceptos básicos

* ¿Qué es *docker*? Un sistema de gestión de *contenedor*
* ¿Qué son los *contenedores*? Podemos pensar en un contenedor como una máquina virtual pequeña y eficiente
* ¿Por qué son *eficientes*? Porque usan el kernel del sistema operativo de la máquina anfitriona y usan mecanismos para aislar los recursos (_cgroups_ y _namespaces_), como procesos, memoria, usuarios, interfaces de red, etc.

|![Contenedor frente a máquina virtual](https://i1.wp.com/www.docker.com/blog/wp-content/uploads/Blog.-Are-containers-..VM-Image-1-1024x435.png?ssl=1)|
|:--:|
|Contenedores (izquierda) frente a máquina virtual (derecha).<br/>*Fuente: Jenny Fong (https://www.docker.com/blog/containers-replacing-virtual-machines/)*|


* Ventajas:
  * El entorno de ejecución se prepara una vez y se ejecuta muchas veces
  * El aislamiento previene problemas de configuración (múltiples versiones de las mismas dependencias)
  * Hay muchos contenedores preparados para ser usados (por ejemplo en [DockerHub](https://hub.docker.com))
  * Seguridad: si el sistema está comprometido, el contenedor se puede destruir y regenerar de nuevo
  * Se pueden hacer copias de seguridad del entorno completo
  * Se pueden probar las aplicaciones en múltiples entornos de ejecución
  * Permite el escalado sencillo de servicios Web
* Desventajas:
  * Requisitos de CPU y almacenamiento
  * Ligero aumento del tiempo de ejecución de la aplicación (especialmente si no puede usando el mismo kernel)
  * Curva de aprendizaje: no es apropiado para aplicaciones pequeñas si el entorno de desarrollo ya está configurado
  * Toda la interfaz de usuario con la aplicación debe ser a través de línea de comando o Web

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
docker volume create myapp
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

Let's download the [Dockerfile](https://raw.githubusercontent.com/NEO-Research-Group/docker-101/4ab32b2ae452c6debc8c261221574e023951ffd5/Dockerfile) and the [index.html](https://raw.githubusercontent.com/NEO-Research-Group/docker-101/26831f58f4397201effa340cbb9758190a51aa2d/index.html) file; and let's build our own image with:
```
docker build . -t mycont
```

The `-t` option is used to assign a name to the image.
We can see the new image with `docker image ls`. We can now create a container based on our image and check that it shows the content of the `index.html` file:
```
docker run --rm -p 80:80 mycont
```

Once we have the image of our app, we can publish it in Docker repos, like DockerHub (follow [these instructions](https://docs.docker.com/docker-hub/repos/)) or GitHub Packages (follow [these ones](https://docs.github.com/es/packages/working-with-a-github-packages-registry/working-with-the-container-registry)).

### Docker-compose

It is common that we have to use several services to run an application. For example, a typical information system needs a web app running in an application server and a database. We could  install in a container both: the application server and the database manager. But this is NOT the docker-way. In docker each container should run ONE servie (docker containers stop when the process with PID 1 stops). 

`docker-compose` is a tool that allows us to combine several docker containers joined by virtual network. This way we can use the existing images of the individual components of our system and combine them to build the infraestructure we need.

The first step to build our docker infraestructure with `docker-compose` is to write a `docker-compose.yml` file:
```
version: '2'
services:
  web:
    build: ./web
    volumes:
        - "files:/var/www/html"
    ports:
        - "8081:80"
    restart: always
  db:
    image: "mysql:5.7.30"
    environment:
        - MYSQL_ROOT_PASSWORD=root_password
        - MYSQL_DATABASE=secret_db
        - MYSQL_USER=secret_user
        - MYSQL_PASSWORD=secret_password
    volumes:
        - "db:/var/lib/mysql"
    restart: always

volumes:
    db:
    files:
```

A more complex example, with 6 containers [here](https://github.com/jfrchicanog/docker-ewp/blob/baedb47a4841e1a51e65f286dabafb7852cdc0de/ewp/docker-compose.yml).

Let's create a docker infrastructure for a Wordpress site. Use [this docker-compose file](https://raw.githubusercontent.com/NEO-Research-Group/docker-101/master/docker-compose.yml). We can create all the containers, network and volumes and run the containers with:
```
docker-compose up -d
```
The `-d` option is to run the command in background. We can see that the containers are running with `docker container ls`, and the new volumes with `docker volume ls`. We should be able to connect to our wordpress installation using a browser: http://localhost:8080. Add a first user.

We can check that the database has a user:
```
docker exec -it root_db_1 /bin/bash
mysql -u exampleuser -pexamplepass exampledb
select * from wp_users;
```

The command `docker exec` runs one process inside an existing container.

We can stop the containers with `docker-compose stop` and start them again with `docker-compose start`, both of them run in the same directory where the `docker-compose.yml` file is located. In order to stop and/or remove all the containers and virtual networks we can run:
```
docker-compose down
```

Observe that the volumes remain: `docker volume ls` and they contain the files we need:
```
docker run --rm -t -v root_wordpress:/wordpress alpine ls /wordpress
```

It is possible to see the logs of all the containers with `docker-compose logs`.

### How to *dockerize* my app/service?

We can follow the next steps to run in docker containers an app/service running in our system:
1. Identify the services you need: database, application server, web server, etc.
2. Find or prepare a docker image for each service.
3. Identify the files you need to store.
4. Prepare a docker volume containing each logical set of files. In the case of files requiring special import (like databases), it is better create an empty volume and use the import procedure using the appropriate container.
5. Adjust the credentials and hosts of databases (or other services) in the files.
6. Prepare a `docker-compose.yml` file with the instructions to build the docker infrastructure.
7. Iterate over steps 4 to 6 to adjust details until it works.

