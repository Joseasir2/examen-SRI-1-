1. Explica métodos para abrir una consola a un contenedor que se está ejecutando.

Si ya está corriendo hay dos formas para poder hacerlo, la primera es en una terminal ejecutando el siguiente comando:
--> docker exec -it [nombre del container][command] 
--> docker exec -it servidor bash

La otra forma sería abriendo VSCode, posicionádonos sobre el container que está corriendo en la extensión de Docker de VSCode, haciendo click derecho y dándole a "attach shell"





2. En el contenedor anterior con qué opciones tiene que haber sido arrancado para poder interecatuar con las entrdas y salidas?

Tiene que haber sido arrancado con:
 --> docker start [nombre del contenedor]
 --> docker start servidor

 Si no se arrancan los contenedores, no dejará entrar a la terminal.





3. Como sería un fichero docker-compose para que dos contenedores se comuniquen entre sí en una red solo de ellos?

Se debe crear una red nueva y en el archivo docker-compose asignarle esa red nueva a los contenedores

Para crear la red usaríamos:

docker network create \
  --driver=bridge \
  --subnet=33.28.0.5/16 \
  --ip-range=33.28.5.0/24 \
  --gateway=33.28.5.254 \
  examen_subnet

Y el fichero compose se vería así:

services:
  bind9:
    image: ubuntu/bind9
    container_name: servidor

    ports:
      - "53:53/tcp"
      - "53:53/udp"
    networks:
      repaso_subnet:
        ipv4_address: 33.28.5.1
    volumes:
      - ./conf:/etc/bind 
      - ./zonas:/var/lib/bind
    environment:
      - TZ=Europe/Paris

  cliente:
    container_name: cliente
    image: alpine
    tty: true
    stdin_open: true
    dns:
      - 33.28.5.1
    networks:
      repaso_subnet:
        ipv4_address: 33.28.5.2

networks:
  repaso_subnet:
    external: true






4. Qué hay que añadir al fichero anterior para que un contenedor tenga la IP fija?

Para ello tendremos que usar IPV4







5. Qué comando de consola puedo usar para saber las ips de los contenedores anteriores?

El comando que se usa para ello es:

--> docker network inspect [nombre de la red]
--> docker network inspect examen_subnet

Esto dará la información de esta red y sus equipos




6. Cual es la funcionalidad del apartado "ports"?

Esto sirve para asignar puertos a un container. Por ejemplo, si ponemos en el docker-compose lo siguiente en apartado de ports:



 ports:
      - "80:8000/udp"

 Esto hará que el puerto 80 se mapee con el 8000 



7. Para que sirve el registro CNAME?

Esto redirecciona subdominios o directamente asocia dominios a otro existente. Por ejemplo:

Al tener una base de datos tal que esta base de datos:


ns		IN A		33.28.5.1

test	IN A		33.28.5.4

www     IN A        33.28.5.7

cosas   IN A        33.28.5.9

alias	IN CNAME	test

texto	IN TXT		mensaje


El dominio de "test" está asociado al "alias" en su CNAME, si nosotros lanzamos un dig con la siguiente sintáxis.

--> dig CNAME @33.28.5.1 alias.tiendaelectronica.int

En la respuesta nos saldrá el nombre "test.tiendaelectronica.int
