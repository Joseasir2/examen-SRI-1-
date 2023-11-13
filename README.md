1. Explcia métodos para abrir una consola a un contenedor que se está ejecutando.

Si ya está corriendo hay dos formas para poder hacerlo, la primera es en una terminal ejecutando el siguiente comando:
--> docker exec -it [nombre del container][command] 
--> docker exec -it servidor bash

La otra forma sería abriendo VSCode, posicionádonos sobre el container que está corriendo en la extensión de Docker de VSCode, haciendo click derecho y dándole a "attach shell"

2. EN el contenedor anterior con qué opciones tiene que haber sido arrancado para poder interecatuar con las entrdas y salidas?

Tiene que haber sido arrancado con:
 --> docker start [nombre del contenedor]
 --> docker start servidor

 Si no se arrancan los contenedores, no dejará entrar a la terminal.

3. Como sería un fichero docker-compose para que dos contenedores se comuniquen entre sí en una red solo de ellos?

Se debe crear una red nueva y en el archivo docker-compose asignarle esa red nueva a los contenedores