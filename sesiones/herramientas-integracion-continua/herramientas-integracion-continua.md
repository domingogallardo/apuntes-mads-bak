
## Herramientas para integración continua

<img src="diapositivas/herramientas-integracion-continua.001.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.002.png" width="800px">

- [Bamboo](https://www.atlassian.com/software/bamboo)
- [Jenkins](https://jenkins.io/index.html)
- [Travis](https://travis-ci.org)

<img src="diapositivas/herramientas-integracion-continua.003.png" width="800px">

- [Continuous delivery with maven](http://www.slideshare.net/wakaleo/continuous-deliverywithmaven)

<img src="diapositivas/herramientas-integracion-continua.004.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.005.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.006.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.007.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.008.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.009.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.010.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.011.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.012.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.013.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.014.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.015.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.016.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.017.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.018.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.019.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.020.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.021.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.022.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.023.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.024.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.025.png" width="800px">


Comprobamos la instalación de docker:


```
$ docker version
$ docker ps
```

Comprobamos las imágenes que hay descargadas en nuestra máquina:

```
$ docker images
```

Ejecutamos una imagen:

```
$ docker run docker/whalesay cowsay Hello world
```

Para generar el mensaje se producen los siguientes pasos:

1. El cliente Docker contacta el demonio Docker que se está ejecutando
   en la máquina host.
2. El demonio comprueba si tenemos la imagen `hello-world` y si no se
   la descarga del _Docker Hub_.
3. El demonio crea un nuevo contenedor a partir de la imagen que
   arranca el ejecutable que produce la salida de texto.
4. El demonio pasa las salida de texto al cliente Docker, el cual la
   envía a la terminal.

Para ver los contenedores en ejecución:

```
$ docker ps
```

Y todos los contenedores (incluyendo los parados):

```
$ docker ps -a
```

Podemos ejecutar un contenedor basado en una imagen Linux Alpine
(Alpine es una distribución Linux muy ligera):

```
$ docker run alpine /bin/echo 'Hello world'
```

O hacerlo de forma interactiva:

```
$ docker run -it alpine /bin/sh
```

También como un demonio, que está en ejecución hasta que lo paramos:

```
$ docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"

1e5535038e285177d5214659a068137486f96ee5c2e85a4ac52dc83f2ebe4147

$ docker ps
CONTAINER ID  IMAGE         COMMAND               CREATED        STATUS       PORTS    NAMES
1e5535038e28  alpine        /bin/sh -c 'while tr  2 minutes ago  Up 1 minute           insane_babbage

$ docker logs insane_babbage

hello world
hello world
hello world
...

$ docker stop insane_babbage
$ docker ps

CONTAINER ID  IMAGE         COMMAND               CREATED        STATUS       PORTS NAMES
```

<img src="diapositivas/herramientas-integracion-continua.026.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.027.png" width="800px">

<img src="diapositivas/herramientas-integracion-continua.028.png" width="800px">

- [https://hub.docker.com](https://hub.docker.com)

<img src="diapositivas/herramientas-integracion-continua.029.png" width="800px">


## Construir una imagen propia

Creamos un directorio:

```
$ mkdir midocker
$ cd midocker
```

Editamos el fichero **Dockerfile**

```
FROM docker/whalesay:latest
RUN apt-get -y update && apt-get install -y fortunes
CMD /usr/games/fortune -a | cowsay
```

Llamamos a `docker build` para construir la nueva imagen y le damos el nombre `docker-whale`:

```
$ docker build -t docker-whale .
```

Comprobamos que se ha añadido al repositorio local de imágenes:

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker-whale        latest              244786599109        13 seconds ago      275 MB
ubuntu              latest              42118e3df429        3 months ago        124.8 MB
python              2.7                 b5c7fb15c9cb        3 months ago        691.6 MB
hello-world         latest              c54a2cc56cbb        4 months ago        1.848 kB
docker/whalesay     latest              6b362a9f73eb        17 months ago       247 MB
```

Y ya podemos ejecutar el contenedor:

```
$ docker run docker-whale
 ________________________________________ 
/ The farther you go, the less you know. \
|                                        |
\ -- Lao Tsu, "Tao Te Ching"             /
 ---------------------------------------- 
    \
     \
      \     
                    ##        .            
              ## ## ##       ==            
           ## ## ## ##      ===            
       /""""""""""""""""___/ ===        
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~   
       \______ o          __/            
        \    \        __/             
          \____\______/   
```


## Servidor web

Es posible también montar en el contenedor un directorio del host. 

Vamos, por ejemplo, a poner en marcha un servidor web y usar un
directorio del host como directorio del sitio web.


Creamos un directorio `webserver` y creamos dentro un fichero `default`:

```
server {
    root /var/www;
    index index.html index.htm;

    # Make site accessible from http://localhost/
    server_name localhost;

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to index.html
        try_files $uri $uri/ /index.html;
    }
}
```

Creamos también el siguiente fichero `Dockerfile`:

```
FROM ubuntu:14.04

RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
RUN apt-get -y install nginx

RUN echo "daemon off;" >> /etc/nginx/nginx.conf
RUN mkdir /etc/nginx/ssl
ADD default /etc/nginx/sites-available/default

EXPOSE 80

CMD ["nginx"]
```

El fichero anterior usa los siguientes comandos para construir la imagen:

- `FROM` le dice a Docker cuál es la imagen base
- `RUN` ejecutará el comando a continuación 
- `ADD` copiará un fichero de la máquina host en la imagen. Es muy útil para ficheros de configuración o scripts que queramos ejecutar.
- `EXPOSE` expondrá un port a la máquina host. Es posible exponer más de un puerto como: `EXPOSE 80 443 8888`
- `CMD` ejecutará un comando cuando se ponga en marcha el contenedor

Una vez definido el fichero `Dockerfile`, podemos construir la imagen:

```
$ docker build -t nginx-example .
```

Podemos comprobar que se ha construido listando las imágenes:

```
$ docker images
```

Por último, lanzamos el servidor web:

```
$ docker run -p 80:80 -d nginx-example
```

El parámetro `p 80:80` liga el puerto 80 del contenedor con el puerto 80 del host.

Hacemos `docker ps` para asegurarnos que el contenedor está
funcionando:

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                NAMES
a377dd528a85        nginx-example       "nginx"             22 seconds ago      Up 21 seconds       0.0.0.0:80->80/tcp   reverent_franklin
```

Si abrimos el navegador en `localhost` veremos que
responde, pero obtenemos un error 500 porque falta el fichero
`index.html`.

Para arreglarlo, creamos el fichero `index.html` en el directorio actual:

**`index.html`**:

```
<h1>Hola desde el contenedor</h1>
```

Y lanzamos el contenedor montando el directorio actual en el
directorio `/var/www` del contenedor (el servidor Nginx usa el
directorio `/var/www` como sitio por defecto donde poner los ficheros
HTML):

```
$ docker stop reverent_franklin
$ docker run -v $(PWD):/var/www:rw -p 80:80 -d nginx-example
```

Si comprobamos ahora el navegador, veremos que muestra el HTML que
hemos guardado. Podemos probar a cambiar el HTML y veremos cómo se
actualiza también en el contenedor.

## Borrar contenedores e imágenes:

- Para borrar un contenedor

    ```
    $ docker rm <ID o nombre contenedor>
    ```

- Para borrar todos los contenedores:

    ```
    $ docker rm $(docker ps -a -q)
    ```

- Para borrar una imagen:

    ```
    $ docker rmi <nombre imagen>
    ```

- Para borrar todas las imágenes:

    ```
    $ docker rmi $(docker images -q)
    ```

## Más información sobre Docker

- [Docker Overview](https://docs.docker.com/engine/understanding-docker/)
- [Get Started with Docker](https://docs.docker.com/engine/getstarted/)


<img src="diapositivas/herramientas-integracion-continua.030.png" width="800px">

- [Travis - Using Docker in builds](https://docs.travis-ci.com/user/docker/)

