
Hola otra vez, vamos a resolver otra máquina de [Dockerlabs](https://dockerlabs.es/#/), en este caso la máquina se llama Pinguinazo y está incluida en la categoría fácil de Dockerlabs de [El Pingüino de Mario](https://www.youtube.com/channel/UCGLfzfKRUsV6BzkrF1kJGsg).

![alt text](images/image.png)

---------------------------------------------------------------------------------------------------------------------------------------------------

Sin más que añadir vamos a ello, como siempre empezaremos por descargar la máquina y realizar su instalación, recordad que funcionan mediante docker por lo que estaremos creando un contenedor en nuestra máquina local en el que se almacenará la máquina víctima.

![alt text](images/image-1.png)

Empezaremos realizando un ping a la máquina para verificar su correcto funcionamiento, al hacerlo vemos que tiene un TTL de 64, lo que significa que la máquina objetivo usa un sistema operativo Linux.

![alt text](images/image-2.png)

Como vemos, la máquina funciona correctamente y podemos empezar con el proceso de enumeración de la misma, vamos a ello.

# Enumeración

Lo primero que haremos para enumerar esta máquina será realizar un escaneo básico de puertos para identificar cuáles están abiertos.

```sudo nmap -p- --min-rate 5000 172.17.0.2 -Pn -n -oN escaneo```

![alt text](images/image-3.png)

Encontramos abiertos los puertos 22 y 80, vamos a realizar un escaneo más exhaustivo para tratar de enumerar las versiones de los servicios así como para lanzar unos scripts básicos de reconocimiento que nos proporciona la propia herramienta de Nmap.

``sudo nmap -p 22,80 --min-rate 5000 -sCV 172.17.0.2 -Pn -n -oN escaneoSC``

![alt text](images/image-4.png)

Vemos el título de la página web que se encuentra en el puerto 80 pero no encontramos nada más interesante, vamos a inspeccionar manualmente la web para tratar de localizar nuestro primer vector de entrada y obtener nuestro primer acceso.

![alt text](images/image-5.png)

En el código fuente encontramos un comentario interesante en el que se menciona un directorio al que podemos acceder, vamos a ver qué nos encontramos en el mismo.

![alt text](images/image-6.png)

![alt text](images/image-7.png)

Podemos ver un directorio con una lista sobre los profesores de esta universidad ficticia, pero uno de ellos llama especialmente la atención ya que pone claramente que es el admin del wordpress, vamos a fuzzear este puerto para ver si localizamos la ruta en la que se encuentre este CMS en caso de estar presente.

![alt text](images/image-8.png)

![alt text](images/image-9.png)

Encontramos la ruta que estábamos buscando, vamos a usar WPscan para enumerar esta versión concreta de Wordpress y enumerar los usuarios para confirmar lo que encontramos previamente.

![alt text](images/image-10.png)

![alt text](images/image-11.png)

Efectivamente, existe el usuario luisillo dentro del Wordpress, vamos a usar el propio WPscan para lanzar un ataque de fuerza bruta a este usuario. 

# Explotación

Antes de realizar este ataque crearemos un diccionario personalizado con los datos que encontramos acerca de este usuario en concreto. Usaré Cupp para generar esta lista de posibles contraseñas ya que es una herramienta bastante intuitiva.

![alt text](images/image-12.png)

Con esta lista creada estamos listos para lanzar el ataque de fuerza bruta, vamos a intentar obtener unas credenciales válidas.

![alt text](images/image-13.png)

![alt text](images/image-14.png)

¡Bien! Tenemos unas credenciales válidas para el usuario luisillo, vamos a iniciar sesión.

![alt text](images/image-15.png)

![alt text](images/image-16.png)

Para poder ver el contenido correctamente tendremos que editar nuestro archivo /etc/hosts para añadir las resolución del nombre que vemos en la URL.

![alt text](images/image-17.png)

Y estamos dentro, vamos a identificar la forma en la que podríamos introducir en el servidor web una reverse shell que nos otorgue una conexión en nuestra máquina atacante.

![alt text](images/image-18.png)

Vemos que podemos subir plugins creados por nosotros mismos, lo cual es crítico ya que podremos usar esto a nuestro favor para introducir en el servidor web una reverse shell.

![alt text](images/image-19.png)

![alt text](images/image-20.png)

Con nuestro archivo zip creado estamos listos para subirlo al servidor como si fuese un plugin, nos pondremos en escucha por el puerto indicado antes de realizar esto.

![alt text](images/image-21.png)

![alt text](images/image-22.png)

![alt text](images/image-23.png)

Esto no ha funcionado como esperaba, vamos a investigar de qué forma podemos hacer que el Wordpress interprete nuestro archivo como un plugin legítimo.

![alt text](images/image-24.png)

Vamos a tratar de hacerlo con esta información a ver si tenemos más suerte.

![alt text](images/image-25.png)

![alt text](images/image-26.png)

![alt text](images/image-27.png)

![alt text](images/image-28.png)

Lo tenemos, hemos conseguido subir nuestra shell correctamente, ahora activaremos nuestro plugin malicioso para conseguir recibir la conexión que nos otorgue nuestro primer acceso al sistema.

![alt text](images/image-29.png)

Activamos el plugin y recibimos la conexión en el puerto en el que nos encontrábamos a la escucha.

# Post-Explotación

Vamos a listar los usuarios disponibles en el sistema, aunque al hacerlo nos vamos a encontrar una sorpresita.

![alt text](images/image-30.png)

Encontramos un archivo "secreto", vamos a tratar de listar su contenido.

![alt text](images/image-31.png)

Vemos que lo que hay en este archivo es la contraseña del usuario luisillo, vamos a pivotar hacia el mismo.

![alt text](images/image-32.png)

Una vez pivotamos con éxito a este usaurio vamos a listar los permisos del mismo para identificar nuestro vector de escalada de privilegios.

![alt text](images/image-33.png)

Este usuario puede ejecutar sudo junto al binario awk, vamos a investigar cómo podemos usar esto para convertirnos en el usuario root.

![alt text](images/image-34.png)

[GTFOBins](https://gtfobins.github.io) como siempre es nuestro gran aliado, y nos dice el comando exacto que podemos usar para elevar nuestros privilegios, vamos a usarlo.

![alt text](images/image-35.png)

Somos el usuario root y hemos comprometido el sistema por completo pudiendo dar por concluida la máquina. Espero que os haya gustado mucho y nos vemos en la siguiente. :)

