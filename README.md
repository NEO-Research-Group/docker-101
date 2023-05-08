# Breve introducción a docker

## Conceptos básicos

* ¿Qué es *docker*? Un sistema de gestión de *contenedor*
* ¿Qué son los *contenedores*? Podemos pensar en un contenedor como una máquina virtual pequeña y eficiente
* ¿Por qué son *eficientes*? Porque usan el kernel del sistema operativo de la máquina anfitriona y usan mecanismos para aislar los recursos (_cgroups_ y _namespaces_), como procesos, memoria, usuarios, interfaces de red, etc.

|![Contenedor frente a máquina virtual](./Docker-vs..png)|
|:--:|
|Contenedores (derecha) frente a máquina virtual (izquierda).<br/>*Fuente: Simran Arora (https://cloudacademy.com/blog/docker-vs-virtual-machines-differences-you-should-know/#:~:text=Docker%20containers%20are%20considered%20suitable,to%20run%20on%20different%20OS.)*|


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

* *Imagen*: Una imagen es una *plantilla de contenedor*. Puede haber muchos contenedores en ejecución basados en la misma imagen. La relación entre un contenedor y una imagen es la misma que entre un proceso y un programa (o un objeto y una clase).

* *Volumen*: Unidad de almacenamiento, ruta en el sistema de ficheros de lamáquina anfitriona o sistema de ficheros temporal que podemos montar en cualquier punto del sistema de ficheros del contenedor.

## Comandos básicos

Puede usar docker instalado en tu máquina para seguir el tutorial o ir al sitio [play with docker](https://labs.play-with-docker.com) si no quiere instalar nada.

### Gestión de contenedores

Creemos y ejecutemos un nuevo contenedor (`alpine` es el nombre de la *imagen*, una distribución de Linux de 5MB):
```
docker run -it alpine
```
La opción `-i` conecta la entrada estándar del shell con la entrada estándar del contenedor. La opción `-t` crea un terminal (virtual) y lo conecta a la entrada y salida estándar del proceso que se ejecuta en docker para permitir una interacción con el usuario.

Comprobemos que el contenedor aún existe tras salir de él:
```
docker container ls -a
```
Sin la opción `-a`, el comando `ls` solo muestra los contenedores en ejecución (no los detenidos). Por ejemplo:
```
docker run -d --name myweb httpd:alpine
docker container ls
```
Se pueden asignar nombres a los contenedores usando la opticón `--name` cuando se crean (en caso contrario, el motor de docker les asignará un nombre aleatorio). Podemos ejecutar los contenedores en segundo plano usando la opción `-d` (en otro caso, el shell esperará hasta que el contenedor se detenga, lo cual hace cuando termine el proceso principal del contenedor).

Para detener un contenedor podemos usar:
```
docker container stop myweb
```

Los contenedores detenidos pueden arrancarse de nuevo con:
```
docker container start myweb
```
Todos los contenedores tienen un ID único además de su nombre. Podemos usar ese ID también para referirnos al contenedor en los comandos de gestión de contenedores (como `rm`, `stop`, `start`, etc.)

Para ver el log (salida estándar) de un contenedor hacemos:
```
docker container logs -f myweb
```
La opción `-f` se usa para que el comando espere a que se produzca nuevo contenido (en el log). Si no se usa esta opción se muestra el contenido del log hasta el momento y termina.

Podemos eliminar un contenedor usando:
```
docker container rm myweb
```

Podemos ejecutar varios contenedores basados en la misma imagen:
```
for i in `seq 1 3`; do docker run -d httpd:alpine; done
```

Cuando detenemos los contenedores ocupan espacio en disco porque mantienen *estado* (un sistema de ficheros completo). Si no estamos interesados en este estado (cambios en el sistema de ficheros del contenedor) podemos eliminar los contenedores cuando temrinen. Podemos hacer esto ejecutádolos con la opción `--rm`:
```
docker run --rm -it alpine
```
```
docker container ls
```

### Mapeo de puertos

Los contenedores tienen un espacio de nombres de red completamente separado del espacio de red de la máquina anfitriona. Podemos *vincular* puertos del sistema anfitrión con puertos del contenedor. De esta forma podemos mapear servicios del contenedor a puertos diferentes de nuestra máquina anfitriona.

Debemos usar la opción `-p` para implementar estos vínculos:

```
docker run --rm -d -p 80:80 httpd:alpine
```

Observemos el nombre de la imagen: `httpd:alpine`. La palabra delante de los dos puntos es el *nombre* de la imagen, mientras que la palabra tras los dos puntos es la *etiqueta*. En este caso estamos descargando una imagen de un servidor HTTP basado en la distribución alpine de Linux. Las etiquetas permiten marcar las variantes de imágenes de servicios conocidos.

### Gestión de volúmenes

El sistema de ficheros de los contenedores y del sistema anfitrión están separados. Podemos montar unidades de almacenamiento virtuales, llamadas *volúmenes* en cualquier ruta del contenedor. El ciclo de vida de los volúmenes es diferente del ciclo de vida de los contenedores: sobreviven la eliminación del contenedor.

Vamos a crear un volumen:
```
docker volume create myapp
```

Montemos el volumen en un contenedor docker con la opción `-v`:
```
docker run --rm -it -v myapp:/app alpine
```
El comando anterior monta el volumen llamado `myapp` en la ruta `/app`. Veamos lo que hay allí:
```
ls /app
```
Podemos ver que está vacío.

Vamos a añadir un fichero al directorio `/app`:
```
cat > /app/file.txt << EOF
hello world
EOF
```

Una vez que salimos y eliminamos el contenedor el volumen persiste y podemos montarlo en cualquier punto de otro contenedor:

```
docker run --rm -it -v myapp:/data alpine
ls /data
cat /data/file.txt
```
En el ejemplo anterior podemos ver el fichero que hemos creado en otro contenedor en una ruta diferente en este nuevo contenedor.

Una alternativa a los volúmenes de docker es *vincular* un fichero o directorio del sistema anfitrión a un punto de montaje (ruta) en el contenedor:
```
docker run --rm -it -v $(pwd):/app alpine
cat > /app/file.txt << EOF
hello world
EOF
exit
```

La forma de especificar estos esos vínculos, llamados *bind mounts* en inglés, es mediante la opción `-v <host-path>:<container-path>`. En el ejemplo anterior, la ruta del sistema anfitrión es proporcionada por el comando `pwd` (funciona en Linux) para montar el directorio actual en `/app`. Si listamos los ficheros del directorio actual encontraremos el fichero `file.txt`:
```
ls
```

Podemos usar *bind mounts* para hacer una copia de seguridad de un volumen:
```
docker run --rm -v $(pwd):/backup -v myapp:/data alpine sh -c "tar czf /backup/archive.tgz -C /data ."
```

Podemos listar todos los volúmenes gestiondos por docker con:
```
docker volume ls
```
Podemos eliminar un volumen usando:
```
docker volume rm myapp
```

### Gestión de imágenes

Podemos ver las imágenes descargadas y creadas en el motor de docker con:
```
docker image ls
```
Podemos eliminar imágenes con:
```
docker image rm alpine
```

Podemos descargar una imagen sin ejecutar un contenedor basado en ella usando `docker pull`:
```
docker pull alpine
```

Todas las imágenes con las que trabajamos se descargan de [DockerHub](https://hub.docker.com), un repositorio de imágenes Docker. En particular, hemos usado solamente *imágenes oficiales*. Se pueden subir imágenes no oficiales a DockerHub (previo registro) y también están disponibles para cualquier usuario de docker. Para descargarlas debemos anteponer el nombre del usuario de DockerHub al nombre de la imagen:

```
docker pull jfrchicanog/graybox
```

Podemos construir nuestras propias imágenes de Docker. El primer paso consiste en crear un `Dockerfile` que contiene la imagen base, un conjunto de instrucciones pra preparar el sistema de ficheros con los ficheros apropiados y metadatos. He aquí un ejemplo:
```
FROM httpd:alpine
COPY index.html /usr/local/apache2/htdocs
```

Descarguemos el fichero [Dockerfile](https://raw.githubusercontent.com/NEO-Research-Group/docker-101/4ab32b2ae452c6debc8c261221574e023951ffd5/Dockerfile) y el fichero [index.html](https://raw.githubusercontent.com/NEO-Research-Group/docker-101/26831f58f4397201effa340cbb9758190a51aa2d/index.html); y construyamos nuestra propia imagen con:
```
docker build . -t mycont
```

La opción `-t` se usa para asignar un nombre a la imagen.
Podemos ver la nueva imagen con `docker image ls`. Podemos crear un contenedor basado en nuestra imagen para comprobar que se muestra el contenido del fichero `index.html`:
```
docker run --rm -p 80:80 mycont
```

Una vez que tenemos la imagen de nuestra aplicación, podemos publicarla en repositorios de Docker, como DockerHub (siguiendo  [estas instrucciones](https://docs.docker.com/docker-hub/repos/)) o GitHub Packages (siguiendo [estas otras instrucciones](https://docs.github.com/es/packages/working-with-a-github-packages-registry/working-with-the-container-registry)).

### Docker-compose

Es habitual que tengamos que usar varios servicios para ejecutar una aplicación. Por ejemplo, un sistema de información típico está formado por una aplicación Web que corre en un servidor de aplicaciones y una base de datos. Podríamos instalar en un único contenedor ambos: el servidor de aplicaciones y el sistema gestor de bases de datos. Pero esta no es la forma de hacerlo en docker. En docker cada contenedor debería ejecutar solo UN servicio (los contenedores docker se detienen cuando el proceso pricnipal, con PID 1, termina).

`docker-compose` es una herramienta que nos permite combinar varios contenedores docker conectados mediante una red virtual. De esta forma podemos usar las imágenes existentes de los componentes individuales de nuestro sistema y combinarlos para contruir la infraestructura que necesitamos.

El primer paso para construir nuestra infraestructura docker con `docker-compose` es escribir un fichero `docker-compose.yml`:
```
version: '3.9'
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

Un ejemplo más complejo, con 6 contenedores se presenta [aquí](https://github.com/jfrchicanog/docker-ewp/blob/baedb47a4841e1a51e65f286dabafb7852cdc0de/ewp/docker-compose.yml).

Vamos a crear una infraestructura docker para un sitio de Wordpress. Usemos [este fichero docker-compose](https://raw.githubusercontent.com/NEO-Research-Group/docker-101/master/docker-compose.yml). Podemos crear todos los contenedores, redes y volúmenes y ejecutar los contenedores con:
```
docker-compose up -d
```
La opción `-d` permite ejecutar el comando en segundo plano. Podemos ver que los contenedores están corriendo con el comando `docker container ls`, y los nuevos volúmenes con `docker volume ls`. Deberíamos ser capaces de conectarnos a nuestra instalación de wordpress usando el navegador: http://localhost:8080. Vamos a añadir un primer usuario.

Podemos comprobar que la base de datos ha almacenado la información de ese primer usuario haciendo:
```
docker exec -it root_db_1 /bin/bash
mysql -u exampleuser -pexamplepass exampledb
select * from wp_users;
```

El comando `docker exec` ejecuta un proceso dentro de un contenedor activo.

Podemos detener los contenedores con `docker-compose stop` y arrancarlos de nuevo con `docker-compose start`. Ambos deben ejecutarse en el mismo directorio donde se encuentra el fichero `docker-compose.yml`. Para detener y/o eliminar todos los contenedores y redes virtuales podemos hacer:
```
docker-compose down
```

Observemos que los volúmenes sobreviven al comando anterior: `docker volume ls` y contienen los ficheros que necesitamos:
```
docker run --rm -t -v root_wordpress:/wordpress alpine ls /wordpress
```

Es posible ver los logs de todos los contenedores con `docker-compose logs`.

### ¿Cómo *dockerizar* una aplicación/servicio?

Podemos seguir los siguientes pasos para ejecutar en contenedores docker una aplicación/servicio que se ejecuta en nuestro sistema:
1. Identificar los servicios que neceistamos: base de datos, servidor de aplicaciones, servidor web, etc.
2. Encontrar o preparar una imagen docker para cada servicio.
3. Identificar los ficheros que necesitamos almacenar.
4. Preparar un volumen de docker que contenga cada conjunto lógico de ficheros. En el caso de ficheros que requieren una forma especial de importación (como bases de datos), es mejor crear un volumen vacío y ejecutar el procedimiento de importación contenedor apropiado.
5. Ajustar las credenciales y nombres de máquinas de las bases de datos (u otros servicios) en los ficheros de configuración apropiados.
6. Preparar un fichero `docker-compose.yml` con las instrucciones para construir la infraestructura docker.
7. Iterar sobre los pasos 5 a 6 para ajustar detalles hasta que funcione.

### Docker en modo enjambre (swarm)

