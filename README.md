# Proxy-reverse
## Requisitos necesarios:

Para la realización de esta práctica necesitaremos tener previamente instalado las siguientes
herramientas:

**- Docker:** es un conjunto de productos de **plataforma como servicio (PaaS)** que utilizan la
virtualización a nivel de sistema operativo para entregar software en paquetes llamados
**contenedores**.
**- Docker-Compose:** es una herramienta para definir y ejecutar aplicaciones **Docker** de varios
contenedores. Con **Docker-Compose** , usa un archivo **YAML** para configurar los servicios de
su aplicación. Luego, con un solo comando, crea e inicia todos los servicios desde su
configuración.

Para instalar las herramientas solo tendremos que utilizar la siguiente sentencia:

```
“sudo apt install *Nombre de la herramienta*”
```
## Configuración sitio 1 y sitio 2:

Lo primero que haremos será crear nuestros directorios donde almacenaremos los archivos que nos
ayudaran a la hora de crear nuestro **Proxy reverso** , para ello utilizaremos el comando **“mkdir
*Nombre del directorio*”** para crear los directorios, una vez creados accederemos al directorio
**“site1”** con el comando **“cd *Directorio*”** :

Ahora lo que haremos tanto en el **site1** como en el **site2** será crear un archivo **.html** donde
crearemos una **pequeña** página web y un archivo **.yml** donde crearemos un **contenedor** para esa
página web, para crear los archivos utilizaremos el editor de texto con el que estemos más cómodos,
yo utilizaré “ **nano** ”, utilizamos el siguiente comando para crear y editar el archivo .html:

```
“sudo nano *Nombre del archivo*. *Extensión*”
```
### Index.html:

En el archivo **.html** introduciremos las siguientes líneas y lo guardaremos:
```
<!DOCTYPE html>
<html>
  <head>
    <title>site1.example.com</title>
   </head>
   <body>
      <h1>site1.example.com</h1>
   </body>
</html>
```


### Docker-compose.yml:

En el archivo **.yml** introduciremos las siguientes líneas y lo guardaremos:
```
version: '3'
services:
  app:
    image: nginx:1.12
    volumes:
      - .:/usr/share/nginx/html/
    expose:
      - "80"
```

Una vez tengamos los archivos listos utilizaremos los siguientes comandos:

- **sudo docker-compose build** : con este comando crearemos imágenes de los servicios
indicados en el **Docker-Compose**.
- **sudo docker-compose up –d** : con este comando levantaremos el contenedor.

```
*Estos pasos los realizaremos también en site2*
```
### Comprobación:

Una vez levantados los dos contendores podremos verlos mediante el siguiente comando:

```
“sudo docker ps -a”
```

## Configuración Proxy reverso:

Ahora accederemos al directorio **“reverse-proxy”** y crearemos los siguientes archivos con sus
respectivas líneas de código:

### Backend-not-found.html:

Aquí tendremos un **.html** el cual aparecerá solo si ocurre algún error en el proxy:
```
<html>
  <head><title>Proxy Backend Not Found</title></head>
  <body >
    <h2>Proxy Backend Not Found</h2>
  </body>
</html>
```

### Docker-compose.yml:

Aquí tenemos el archivo que levantará el contenedor del proxy y configure los dos sitios web:
```
version: '3'
services:
  proxy:
    build: ./
    networks:
      - site1
      - site2
    ports:
      - 80:80

networks:
  site1:
    external:
      name: site1_default
  site2:
    external:
      name: site2_default
```

### Dockerfile:

Este archivo lleva los comandos que se ejecutaran cuando se active el **docker-compose.yml** :
```
FROM nginx:1.12

#  default conf for proxy service
COPY ./default.conf /etc/nginx/conf.d/default.conf

# NOT FOUND response
COPY ./backend-not-found.html /var/www/html/backend-not-found.html

# Proxy configurations
COPY ./includes/ /etc/nginx/includes/
```


### Default.conf:

En este archivo encontraremos la configuración necesaria que utilizará el Dockerfile para poder
levantar el contendor:
```
# web service1 config.
server {
  listen 80;
  server_name site1.example.com;

  location / {
    include /etc/nginx/includes/proxy.conf;
    proxy_pass http://site1_app_1;
  }

  access_log off;
  error_log  /var/log/nginx/error.log error;
}

# web service2 config.
server {
  listen 80;
  server_name site2.example.com;

  location / {
    include /etc/nginx/includes/proxy.conf;
    proxy_pass http://site2_app_1;
  }

  access_log off;
  error_log  /var/log/nginx/error.log error;
}

# Default
server {
  listen 80 default_server;

  server_name _;
  root /var/www/html;

  charset UTF-8;

  error_page 404 /backend-not-found.html;
  location = /backend-not-found.html {
    allow   all;
  }
  location / {
    return 404;
  }

  access_log off;
  log_not_found off;
  error_log  /var/log/nginx/error.log error;
}
```
### Includes:

Crearemos una carpeta donde crearemos un archivo con la configuración del proxy.

#### Proxy.conf:

Aquí encontraremos toda la configuración del proxy:
```
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_buffering off;
proxy_request_buffering off;
proxy_http_version 1.1;
proxy_intercept_errors on;
```


### Ejecución:

Una vez creados todos los archivos necesarios utilizaremos los siguientes comandos:

- **sudo docker-compose build** : con este comando crearemos imágenes de los servicios
indicados en el **Docker-Compose**.
- **sudo docker-compose up –d** : con este comando levantaremos el contenedor.

### Comprobación:

Ahora utilizaremos el comando **“sudo docker ps”** para ver que se ha levantado el nuevo contendor:

### Configuración archivo hosts:

Por último accederemos al archivo hosts para añadir un par de líneas, para ello haremos **“sudo nano
/etc/hosts”** y añadiremos lo siguiente:
```
IP *url sitio 1*
IP *url sitio 2*
```
## Comprobación final:

Para finalizar nos dirigiremos a nuestro navegador y escribiremos el nombre que le hayamos puesto
a la **IP** en el archivo **hosts,** en nuestro caso **site1.example.com** y **site2.example.com,** como podemos
comprobar nos aparecen los dos **.html** distintos según la **URL** que empleemos y apuntando a la
misma **IP**.

## Importante:

```
-A la hora de agregar los archivos .yml hay que tener especial cuidado a la hora de escribirlo
ya que si escribimos un espacio de más o de menos pues hacer que el archivo no se ejecute
correctamente, también cuidado a la hora de escribir una palabra mal.

-Cuidado con las rutas de los directorios, un fallo a la hora de escribir donde se sitúa nuestro
archivo puede darnos un error el cual luego no pensemos que pueda ser ese el error que nos
esté generando el fallo.
```
