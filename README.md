# Short introduction to docker

## Basic concepts

* ¿What is *docker*? A *container* management system
* ¿What are *containers*? We can think in a container as a small and very efficient virtual machine
* ¿Why are they *efficient*? Because they share the kernel of the host machine and use some mechanisms for isolating resources (cgroups and namespaces), like processes, memory, users, network interfaces, etc.

|![Container vs. virtual machine](https://i1.wp.com/www.docker.com/blog/wp-content/uploads/Blog.-Are-containers-..VM-Image-1-1024x435.png?ssl=1)|
|:--:|
|Containers (left) vd. virtual machines (right).<br/>*Source: Jenny Fong (https://www.docker.com/blog/containers-replacing-virtual-machines/)*|


4. Ventajas: entornos montados rápidos, ideal para reproducir entornos de ejecución sin problemas de configuración, muchos contenedores listos para usar, seguridad, copias sencillas, múltiples versiones, escalado de aplicaciones masivo
5. Inconvenientes: carga en tiempo de ejecución
6. Conceptos: contenedor, imagen, volumen

Comandos básicos para gestión de contenedores: docker run, docker container (ls, rm)
- Run - - rm (ejemplo hola mundo ver qué pasa si no se pone rm)
- Docker container ls
- Run - i t

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

