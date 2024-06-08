
Hola de nuevo y gracias por leerme. Hoy seguiremos completando el Path de Offensive Pentesting de TryHackMe con una máquina llamada HackPark así que no vamos a perder el tiempo y nos ponemos manos a la obra, espero que os guste y que todos aprendamos mucho juntos. Recordad que todo lo que veremos en el día de hoy se está utilizando en un entorno de pruebas controlado y que su uso fuera de este ámbito no es ético y podría conllevar problemas penales.


Como siempre empezaremos conectando nuestra máquina atacante a la VPN de TryHackMe e iniciando la máquina, lo cual no debería llevarnos más de 2 minutos, una vez hecho esto estamos listos para comenzar nuestra aventura.


![alt text](image.png)


![alt text](image-1.png)


Con nuestra máquina activa y como solemos hacer, empezaremos mandando un ping a la máquina para verificar que se encuentra en correcto funcionamiento. 


![alt text](image-2.png)


# Enumeración


Al igual que en la máquina Alfred este objetivo no responde a las conexiones por el protocolo ICMP, pero como ya vimos anteriormente tenemos en nmap un parámetro perfecto para estos casos, el parámetro -Pn. 


``` sudo nmap -p- --min-rate 5000 10.10.249.6 -Pn -n -oN escaneo ```


![alt text](image-3.png)


Vemos dos puertos abiertos, el puerto 80 haciendo referencia a un servicio HTTP y el puerto 3389 que está relacionado con RDP lo cual nos indica que esta máquina víctima utiliza el sistema operativo Windows. Pasamos a realizar un escaneo más exhaustivo a estos dos puertos antes de visitar la web.


``` sudo nmap -p 80,3389 -sCV 10.10.249.6 -Pn -n -oN escaneoSC ```


![alt text](image-4.png)


Podemos ver alguna información sobre el nombre de la máquina y alguna versión, pero nada que nos llame demasiado la atención, así que vamos a ir al puerto 80 para ver qué podemos encontrarnos allí. 


![alt text](image-5.png)


Vemos un blog en el que aparece la imagen de un payaso conocido por la antigua serie de It, basada en los relatos del gran Stephen King. En una de las preguntas que TryHackMe nos hace nos pide el nombre del payaso que aparece en la web. Para conseguirlo simplemente tendremos que hacer una investigación en internet o utilizar la búsqueda inversa de imágenes.


![alt text](image-6.png)


![alt text](image-7.png)


Contestamos correctamente a la pregunta y seguimos enumerando esta web. Utilizaremos cualquier herramienta de fuzz para descubrir directorios ocultos o archivos interesantes.


![alt text](image-8.png)


Encontramos el directorio admin, el cual nos da un código de error 300, lo cual significa que hace una redirección. Si vamos a este directorio nos lleva a un panel de login en el que podremos probar credenciales por defecto o ataques de fuerza bruta. Inspeccionemos un poco su funcionamiento.


![alt text](image-9.png)


Intentamos manualmente varias combinaciones pero todas nos dan error en el login. Si nos fijamos vemos que debajo del panel de login hay una función de recuperación de contraseña, vamos a echarle un vistazo.


![alt text](image-10.png)


El hecho de que nos indique en texto claro si un usuario existe o no nos da la posibilidad de enumerar usuarios desde aquí, por lo que prepararemos nuestro BurpSuite para interceptar la petición y hacer un pequeño ataque de fuerza bruta para lograr enumerar usuarios válidos.


![alt text](image-11.png)


![alt text](image-12.png)


![alt text](image-13.png)


Con la petición interceptada la enviaremos al Intruder de nuestro Burp para realizar un ataque de tipo Sniper. 


![alt text](image-14.png)


Con esto realizado iremos a la parte de payloads y crearemos un alista simple con usuarios posibles.


![alt text](image-15.png)


Bien, estamos listos para lanzar el ataque así que vamos allá.


![alt text](image-16.png)


Vemos que con el usuario admin el output cambia gracias a la longitud de la respuesta y nos dice que el servidor ha intentado enviar el email de recuperación de la contraseña aunque no ha sido posible. De cualquier forma esto nos indica que ese usuario en concreto existe dentro de la base de datos, por lo que sabiendo esto podemos realizar otro ataque de fuerza bruta al panel de login en sí tratando de encontrar la contraseña de este usuario, para esto usaremos Hydra ya que es lo que no recomienda TryHackMe en este caso, aunque podríamos seguir usando el intruder de BurpSuite.


Para utilizar Hydra antes interceptaremos la petición POST del panel de login.


![alt text](image-17.png)


Con esto interceptado podemos ver los datos que envía la petición POST, los cuales necesitaremos para utilizar Hydra.


![alt text](image-18.png)


Con nuestro ataque listo simplemente lo lanzaremos y esperaremos hasta que encuentre una coincidencia entre al usuario y la contraseña.


![alt text](image-19.png)


¡Eso es! Tenemos la contraseña del usuario admin y podemos entrar en el panel de admin de este sitio web.


![alt text](image-20.png)


![alt text](image-21.png)


En la pestaña About podemos identificar la versión de este servicio web así que haremos una investigación sobre el mismo. Al buscar nos damos cuenta de que cuenta con una vulnerabilidad de ejecución remota de código CVE-2019-6714.


![alt text](image-22.png)











