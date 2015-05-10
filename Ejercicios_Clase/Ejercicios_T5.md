#Ejercicios Tema 5


##Ejercicio 5.1: Buscar información sobre cómo calcular el número de conexiones por segundo.

Una primera forma sería con Apache Benchmark. Subiríamos de forma progresiva el número de peticiones por segundo hasta nuestro servidor mientras no se produjese ningun error en ninguna de las peticiones. Dicha forma sería válida para la mayoría de los benchmarks.

Si lo que queremos es saber las actuales en Apache podemos activar mod-status el cual su código en httpd.conf es algo similar a esto:

    <Location /server-status>
    SetHandler server-status

    Order Deny,Allow
    Deny from all
    Allow from .website.com
    </Location>

Se accede mediante http://your.server.name/server-status.

Otras aplicaciones de utilidad son iptstate, apache2ctl y netstat.

##Ejercicio 5.2: Instalar wireshark y observar cómo fluye el tráfico de red en uno de los servidores web mientras se le hacen peticiones HTTP.
Para llevarlo a cabo he usado una máquina con CentOS y otra con Ubuntu que disponía en VirtualBox.

Se instala en CentOS wireshark mediante yum install wireshark-gnome. Con el instalado selecionamos la interfaz de red que usamos para la red donde se realizarán las peticiones y le indicamos que empiece a capturar paquetes.

Desde la otra máquina accedemos a la IP de CentOS. En wireshark se verá algo parecido a esto:

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Ejercicios_Clase/Ejercicio_5_2.png?raw=true)

Apreciamos los envios de ACK para establecer una conexión y sus SYN como respuesta. Se intenta pedir la página principal al servidor pero al no existir ninguna página en el servidor este le responde con un error de no found. Esto se repite por cada petición realizada al servidor.

##Ejercicio 5.3: Buscar información sobre características, disponibilidad para diversos SO, etc de herramientas para monitorizar las prestaciones de un servidor.
Nagios. Monitor creado para Linux capaz de monitorizar los servicios de red, los recursos hardware y posee diversas características de personalización como el acceso remoto, el servicio de notificaciones, el chequeo de servicios paralelizados o la visualización en tiempo real del estado de la red.


Munin. Monitor de servidores para Linux cuya principal virtud es su extensibilidad mediante plugins. También posee una estructura basada en nodos que nos permite recopilar información de varios servidores de forma simultánea.

AWStats. Válido tanto para Windows como para Linux nos permite generar informes html y log sobre todo lo que ocurre en los servicios que tenemos en funcionamiento en nuestro servidor. Toda la información recogida puede reprensentarse en gráficas y tablas.

Ganglia. Sistema de monitoreo de computadores de alto rendimiento como clusters. Gracias a sus algoritmos consume bastante poco por nodo y utiliza una alta concurrencia a la hora de obtener toda la información tanto a nivel de servicios como hardware de nuestros servidores.