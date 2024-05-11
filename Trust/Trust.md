Hola a todos y bienvenidos, este va a ser mi primer WriteUp así que tampoco prometo que vaya a ser una maravilla, pero se intentará xD

En este caso voy a estar haciendo una de las máquinas de la plataforma creada por un querido miembro de la comunidad, Mario de El Pingüino de Mario, echad todos un ojo a sus redes sociales, seguro que no os defraudará.

La máquina en cuestión se llama Trust y es una de las más sencillas de la plataforma.

![alt text](image.png)

Siguiendo las instrucciones de instalación de la máquina y de docker, podremos empezar con ello. Al instalar la máquina se nos proporcionará la dirección IP de la misma, sinceramente el proceso de instalación me ha parecido muy sencillo y eso se agradece.

![alt text](image-1.png)

![alt text](image-2.png)

Comprobamos utilizando un ping que la máquina es efectivamente accesible. Esto también nos ayuda a identificar qué sistema operativo está utilizando, si el ttl es de 128 estaremos ante una máquina Windows, si por el contrario, y como sucede en este caso, el ttl es de 64, entonces sabremos que estamos antes una máquina Linux.

![alt text](image-3.png)

Con todo esto hecho, empezamos a hacer un reconocimiento básico de los puertos abiertos para saber cuál va a ser nuestro vector de ataque.

`sudo nmap -p- --open 172.17.0.2 -Pn -n --min-rate 5000 -vvv -oN escaneo`

Vemos que la máquina tiene abiertos los puertos 80 y 22, que hacen referencia a un servicio http y a otro ssh, respectivamente.

![alt text](image-4.png)

Sabiendo esto, lo primero que haremos será acceder al servicio web a través de nuestro navegador para ver qué nos encontramos.

![alt text](image-5.png)

Nos encontramos con una página por defecto de Apache, si usamos Wappalyzer también podemos ver la versión del mismo, siendo en este caso la versión 2.4.57. También podremos verificar esto haciendo un escaneo con nmap tan solo a los puertos abiertos y especificando que queremos que ejecute unos scripts básicos de reconocimiento, además de darnos las versiones con el argumento -sCV.

`sudo nmap -p 22,80 -sCV 172.17.0.2 -Pn -n --min-rate 5000 -vvv -oN escaneoSC`

![alt text](image-6.png)

![alt text](image-7.png)

Buscamos en internet las versiones y vemos si tienen alguna vulnerabilidad conocida, no parece ser el caso así que procederemos a fuzzear la web para buscar posibles directorios y archivos. Aunque antes de esto echaremos un ojo al código fuente para ver si hay algún comentario o algo interesante que nos pueda servir de ayuda.

![alt text](image-8.png)

En el código fuente no hay nada interesante, así que vamos a hacer fuzz. Para esto se pueden usar varias herramientas como dirsearch, wfuzz, o gobuster. En este caso usaré feroxbuster ya que un compañero me la recomendó esta semana. 

`feroxbuster -u http://172.17.0.2/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt`

![alt text](image-9.png)

En principio no vemos que haya encontrado ningún directorio, por lo que usaremos el argumento -x para indicarle extensiones para buscar archivos.

`feroxbuster -u http://172.17.0.2/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,xml,html,js`

![alt text](image-10.png)

Tenemos cositas interesantes por aquí, un index.html y un secret.php, no tan secreto por lo visto, así que vamos a ojear lo que nos encontramos.

![alt text](image-11.png)

En el index.html nos encontramos con la página anterior, por supuesto.

![alt text](image-12.png)

¡Vaya! ¿Qué tenemos por aquí? Nos aparece un mensaje diciendo que esta web no se puede hackear, y posiblemente tenga razón, pero lo más interesante aquí es que nos llama Mario, por lo que podríamos dar por hecho que tenemos un posible usuario. Si nos remontamos al escaneo inicial, recordamos que tenemos un servicio ssh corriendo en esta máquina, por lo que podríamos probar a realizar fuerza bruta a este servicio con el usuario que hemos obtenido.

Para realizar esto utilizaremos hydra, una potente herramienta para hacer ejercicios de fuerza bruta.

`hydra -l mario -P /usr/share/wordlists/rockyou.txt 172.17.0.2 ssh`

Y efectivamente, tenemos la contraseña del usuario y podremos acceder mediante ssh a la máquina.

![alt text](image-13.png)

`ssh mario@172.17.0.2`

![alt text](image-14.png)

Utilizando `sudo -l` y proporcionando la contraseña para el usuario podemos listar lo que podemos utilizar utilizando sudo.

![alt text](image-15.png)

Vemos que podemos utilizar el comando vim como usuario privilegiado, por lo que hacemos una búsqueda para ver cómo podríamos aprovechar esto para elevar nuestros privilegios. Accediendo a la web https://gtfobins.github.io nos encontramos con esto al buscar vim.

![alt text](image-16.png)

Usaremos el siguiente comando y conseguiremos escalar nuestros privilegios y convertirnos en el usuario root, teniendo de esta forma el control total de la máquina.

`sudo vim -c ':!/bin/sh'`

![alt text](image-17.png)

Espero que os haya gustado, y nos veremos en la próxima. Agradecer a Mario por haber hecho esta plataforma y estas máquinas que son muy instructivas y seguiremos haciendo hasta completar todas. Un besito a todos <3






