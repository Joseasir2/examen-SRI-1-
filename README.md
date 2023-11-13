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


8. Como puedo hacer para que la configuración de un contenedor DNS no se borre si creo otro contenedor?

Es necesario utilizar volúmenes en el docker-compose, es decir, los volúmenes son archivos fuera del sistema de archivos de un contenedor que guradan su configuración aunque estos se detengan o se eliminen.

ejemplo:

volumes:

  -./conf:/etc/bind
  -./zonas:/var/lib/bind


 9. Añade una zona tiendaelectronica.int en tu docker DNS que tenga

 -www a la ip 172.16.0.1
Comando utilizado:
--> dig @80.28.5.1 www.tiendaelectronica.int

  ;; ANSWER SECTION:
www.tiendaelectronica.int. 38400 IN     A       172.16.0.1

 -owncloud sea un CNAME de www
Comando utilizado:
--> dig CNAME  @80.28.5.1 owncloud.tiendaelectronica.int

;; ANSWER SECTION:
owncloud.tiendaelectronica.int. 38400 IN CNAME  test.tiendaelectronica.int.

 -Un registro de texto con el contenido "1234ASDF"
Comando utilizado:
--> dig -t TxT @80.28.5.1 texto.tiendaelectronica.int

;; ANSWER SECTION:
texto.tiendaelectronica.int. 38400 IN   TXT     "123456ASD"

 -Comprueba que todo funciona con el comando dig
 -Muestra en los logs que el servicio arranca correctamente

Para ver los logs he usado "view logs" en la extensión de VSCode en este container

 13-Nov-2023 15:59:43.554 all zones loaded
13-Nov-2023 15:59:43.554 running
13-Nov-2023 15:59:43.558 managed-keys-zone: No DNSKEY RRSIGs found for '.': succes
 

10. Realiza el apartado 9 en la máquina virtual con DNS

ZONA DNS EN MV

Tendremos que instalar bind9, iniciarlo y en los archicos de /etc/bind poner la configuracion que encesitamos para que funcione:

named.conf.local:

zone "asircastelao.int" {
	type master;
	file "/var/lib/bind/db.asircastelao.int";
	allow-query {
		any;
		};
	};

named.conf.options:

options {
	directory "/var/cache/bind";

	forwarders {
	 	8.8.8.8;
		8.8.4.4;
		1.1.1.1;
	 };
	 forward only;

	listen-on { any; };
	listen-on-v6 { any; };

	allow-query {
		any;
	};
};

named.conf.default-zones:

// prime the server with knowledge of the root servers
zone "." {
	type hint;
	file "/usr/share/dns/root.hints";
};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

zone "localhost" {
	type master;
	file "/etc/bind/db.local";
};

zone "127.in-addr.arpa" {
	type master;
	file "/etc/bind/db.127";
};

zone "0.in-addr.arpa" {
	type master;
	file "/etc/bind/db.0";
};

zone "255.in-addr.arpa" {
	type master;
	file "/etc/bind/db.255";
};


Desde el host hacemos dig y nos tienes que dar lo siguiente:

;; ANSWER SECTION:
test.asircastelao.int.	38400	IN	A	172.28.5.4

 Los logs los podemos ver si vemos el  archivo /var/log/auth.log

 Nov 13 17:52:10 josete-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:1 (system bus name :1.41 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov 13 17:52:34 josete-VirtualBox dbus-daemon[585]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov 13 17:58:09 josete-VirtualBox sudo:   josete : TTY=pts/1 ; PWD=/var/lib/bind ; USER=root ; COMMAND=/usr/sbin/reboot
Nov 13 17:58:09 josete-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:58:17 josete-VirtualBox systemd-logind[599]: New seat seat0.
Nov 13 17:58:17 josete-VirtualBox systemd-logind[599]: Watching system buttons on /dev/input/event0 (Power Button)
Nov 13 17:58:17 josete-VirtualBox systemd-logind[599]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov 13 17:58:17 josete-VirtualBox systemd-logind[599]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov 13 17:58:19 josete-VirtualBox gdm-autologin]: gkr-pam: no password is available for user
Nov 13 17:58:19 josete-VirtualBox gdm-autologin]: pam_unix(gdm-autologin:session): session opened for user josete(uid=1000) by (uid=0)
Nov 13 17:58:19 josete-VirtualBox systemd-logind[599]: New session 1 of user josete.
Nov 13 17:58:19 josete-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user josete(uid=1000) by (uid=0)
Nov 13 17:58:19 josete-VirtualBox gdm-autologin]: gkr-pam: gnome-keyring-daemon started properly
Nov 13 17:58:19 josete-VirtualBox gnome-keyring-daemon[1190]: The Secret Service was already initialized
Nov 13 17:58:19 josete-VirtualBox gnome-keyring-daemon[1190]: The PKCS#11 component was already initialized
Nov 13 17:58:19 josete-VirtualBox gnome-keyring-daemon[1190]: The SSH agent was already initialized
Nov 13 17:58:20 josete-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:1 (system bus name :1.46 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov 13 17:58:44 josete-VirtualBox dbus-daemon[581]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)








