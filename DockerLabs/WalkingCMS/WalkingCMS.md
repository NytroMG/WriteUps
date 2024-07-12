
Hola otra vez, vamos a resolver otra máquina de [Dockerlabs](https://dockerlabs.es/#/), en este caso la máquina se llama WalkingCMS y está incluida en la categoría fácil de Dockerlabs de [El Pingüino de Mario](https://www.youtube.com/channel/UCGLfzfKRUsV6BzkrF1kJGsg).

![alt text](image.png)

---------------------------------------------------------------------------------------------------------------------------------------------------

Sin más que añadir vamos a ello, como siempre empezaremos por descargar la máquina y realizar su instalación, recordad que funcionan mediante docker por lo que estaremos creando un contenedor en nuestra máquina local en el que se almacenará la máquina víctima.

![alt text](image-1.png)

Empezaremos realizando un ping a la máquina para verificar su correcto funcionamiento, al hacerlo vemos que tiene un TTL de 64, lo que significa que la máquina objetivo usa un sistema operativo Linux.

![alt text](image-2.png)

Como vemos, la máquina funciona correctamente y podemos empezar con el proceso de enumeración de la misma, vamos a ello.

# Enumeración

Lo primero que haremos para enumerar esta máquina será realizar un escaneo básico de puertos para identificar cuáles están abiertos.

```sudo nmap -p- --min-rate 5000 172.17.0.2 -Pn -n -oN escaneo```

![alt text](image-3.png)

Vemos que únicamente tenemos disponible el puerto 80, de cualquier forma vamos a lanzar un escaeno más exhaustivo para tratar de enumerar versiones de servicios así como lanzar unos scripts básicos de reconocimiento.

```sudo nmap -p 80 -sCV 172.17.0.2 -Pn -n -oN escaneoSC```

![alt text](image-4.png)

Nada interesante por el momento, vamos a echar un vistazo al puerto 80 para ver qué podemos encontrarnos y descubrir vulnerabilidades que podamos explotar.

![alt text](image-5.png)

Sólo nos encontramos una página por defecto, parece que aquí no vamos a encontrar nada que nos sirva. Vamos a inspeccionar el código fuente.

![alt text](image-6.png)

Tampoco hay nada por aquí, vamos a realizar un fuzzeo para enumerar directorios y archivos que no estén a la vista del público general y que puedan aumentar nuestra superficie de ataque.

![alt text](image-7.png)

Vaya, parece que encontramos un directorio que se llama wordpress, vamos a intentar acceder al mismo.

![alt text](image-8.png)

![alt text](image-9.png)

Efectivamente, tenemos un Wordpress corriendo en esta máquina, un CMS que no es particularmente seguro. En esta página hay un texto que dice que la web es invulnerable, vamos a ver si esto es cierto. Lo primero que haremos será volver a fuzzear desde este nuevo directorio.

![alt text](image-10.png)

Encontramos el directorio que estábamos buscando, el directorio wp-admin nos redirecciona a wp-login y nos presenta un panel de login que podemos explotar, aunque antes necesitaremos al menos un usuario para realizar un ataque de fuerza bruta. Vamos a escanear toda la web con WPscan para tratar de enumerar usuarios disponibles dentro del CMS.

![alt text](image-11.png)

![alt text](image-12.png)

![alt text](image-13.png)

# Explotación

Bien, encontramos el usuario mario, vamos a realizar un ataque de fuerza bruta al panel de login para lograr un inicio de sesión exitoso.

![alt text](image-14.png)

![alt text](image-15.png)

Parece que en este caso hydra está tardando más de lo previsto por lo que usaré la propia herramienta de WPscan para hacer este ataque de fuerza bruta a ver si los resultados son más fiables.

![alt text](image-17.png)

![alt text](image-16.png)

¡Genial! Ha funcionado correctamente y tenemos unas credenciales válidas, vamos a utilizarlas para iniciar sesión como el usuario.

![alt text](image-18.png)

Una vez dentro sabemos que en WordPress se puede editar el código de las plantillas para introducir código malicioso. Vamos a intentar hacer esto para enviar una shell a nuestra máquina atacante. Introduciremos el código y lo editaremos para indicar nuestra IP y el puerto en el que nos pondremos a la esucha en la máquina atacante.

![alt text](image-19.png)

![alt text](image-20.png)

![alt text](image-21.png)

Estamos dentro y hemos logrado nuestro primer acceso, vamos a buscar la manera de escalar nuestros privilegios.

# Post-Explotación


![alt text](image-22.png)

Parece que no tenemos ningún usuario al que pivotar así que trataremos de convertirnos en root directamente desde este usuario. Vamos a enumerar el sistema para buscar vías potenciales de escalada.

Usaremos el comando ```find / -user root -perm /4000 2>/dev/null``` para localizar los binarios que tienen activados los permisos SUID.

![alt text](image-23.png)

Aquí localizamos que el binario env tiene estos permisos, lo cual es algo extraño. Vamos a investigar en [GTFObins](https://gtfobins.github.io) para ver si hay alguna manera de usar este permiso para elevar nuestros privilegios.

![alt text](image-24.png)

Parece que efectivamente hay una forma de usar esto para convertirnos en root, vamos a aprovecharnos de este permiso para comprometer el sistema por completo.

```/usr/bin/env /bin/bash -p```

![alt text](image-25.png)

¡Genial! Somos root y tenemos el control total del sistema, pudiendo dar por completada la máquina. Espero que os haya gustado mucho y nos vemos en la siguiente. :)

