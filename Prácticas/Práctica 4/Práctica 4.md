#Práctica 4
En la cuarta práctica se realizarán pruebas de rendimiento a las opciones de balanceo de carga que se propusieron en la práctica anterior. Para ello se utilizarán algunos de las aplicaciones de benchmark más usadas en la actualidad.

###Preparación

Para la correcta realización de los benchmarks usaremos un script en lenguaje PHP. Este es un bucle que realiza diversos cálculos con la finalidad de que se requiera una mayor cantidad de tiempo a la hora de ofrecer una respuesta a la petición.

El código PHP es el siguiente:
```sh
<?php
$tiempo_inicio = microtime(true);
for ($i=0; $i<250000; $i++){
 $a = $i * $i;
 $b = $a - $i;
 $c = $a / $b;
 $d = 1 / $c;
}
$tiempo_fin = microtime(true);
echo "Tiempo empleado: " . round($tiempo_fin - $tiempo_inicio, 4) ; 
?>
```

##Apache Benchmark

El primer benchmark que vamos a usar es el desarrollado por Apache. Su funcionamiento es muy simple. Cuando está instalado o lo tenemos como ejecutable tan solo lo llamamos con el comando ab y posteriormente le indicamos el número de peticiones mediante -n, y el número de peticiones de forma concurrente con -c. Finalmente le indicamos la URL de la web donde realizar las peticiones.

El primer benchmark lo realizaremos en una de las máquinas donde tenemos instalado Apache server. Lanzaremos el AB de la siguiente forma:

    ab -n 1000 -c 10 http://192.168.64.128/bench.php
    
Donde como se explicó previamente se realizan 1000 solicitudes en concurrencias de 10. La URL es la ubicación de nuestro script en el servidor.
Este test lo realizamos 5 veces para calcular de forma más precisa la media y la desviación.

De los resultados obtenidos haremos el estudio sobre 4 de ellos: Time taken for tests, Failed requests, Requests per second y Time per request (ms).

Realizados los benchmarks en el servidor Apache, el siguiente será en la tercera máquina con el balanceador Nginx activado. Por tanto debemos parar el servicio HAProxy y asegurarnos que Nginx está en funcionamiento.
El comando a ejecutar es como el anterior salvo por la IP de la tercera máquina.

    ab -n 1000 -c 10 http://192.168.64.130/bench.php
    
Anotamos los resultados de los 5 tests y pasamos a la última configuración. Paramos el servicio Nginx y activamos HAProxy. realizamos los benchmarks con la misma orden y como en el caso anterior anotamos los resultados.

Para calcular la media y la desviación deberemos generar una tabla como la de la siguiente imagen:



![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%204/Tabla_AB.png?raw=true)

Los resultados almacenados en las tablas representados en gráficas en función de los parámetros estudiados son los siguientes:

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%204/Grafico1_AB.png?raw=true)

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%204/Grafico2_AB.png?raw=true)

##HTTPerf

El siguiente programa para realizar benchmarks de rendimiento es HTTPerf. Este software desgraciadamente no posee una versión para ejecutarse en Windows. Como solución se ha creado una máquina virtual con acceso a las otras.
Su instalación es muy simple, solamente introducimos

     sudo apt-get install httperf
     
En teoría debería funcionar correctamente pero al menos en mi caso me aparecía un error en el momento de la ejecución. Este error se debía al tamaño del descriptor FD_SETSIZE. Para arreglarlo de forma local y temporal introducimos ulimit -n 50000 y funcionará correctamente.

El comando a ejecutar para la máquina con el servidor Apache será el siguiente:

    httperf --server 192.168.64.128 --port 80 --uri /bench.php --rate 150 --num-conn 27000 --num-call 1 --timeout 5
    
Así indicamos el número de llamadas, el de conexiones, el ratio, la IP, el puerto y la ruta. Este benchmark lo realizamos 5 veces y anotamos en una tabla los parámetros obtenidos en Total connections, Total replies, Requests rate y Errors total.

Para Nginx y HAProxy el comando a ejecutar es similar salvo la IP que debemos introducir la de la tercera máquina. Como siempre paramos uno de los servicios cuando hagamos el benchmark al otro.

Realizadas las pruebas de rendimiento de las tres formas e introducidas en una tabla para calcular la media y desviación nos aparecen los siguientes resultados:

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%204/Tabla_HTTPerf.png?raw=true)

Estos resultados los representamos de forma gráfica en función de cada parámetro medido los cuales quedan de esta forma:

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%204/Grafico1_HTTPerf.png?raw=true)

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%204/Grafico2_HTTPerf.png?raw=true)

##OpenWebLoad
Finalmente usaremos OpenWebLoad para concluir los tests de rendimiento. En este caso si existe un ejecutable funcional en Windows. Lo descargamos y lo llamamos desde terminal. Posteriormente introducimos la url y el número de conexiones simultáneas. En nuestro caso serán 20. Realizaremos estos tests durante 30 segundos.

    openload.exe 192.168.64.128/bench.php 20
    
El comando mostrado en la parte superior sería para el servidor Apache. Para Nginx y HAProxy se indicaría la IP (192.168.64.130). Como siempre solo puede existir uno de los dos servicios en ejecución para realizar los tests.
Con los resultados obtenidos realizaremos las tablas y gráficos con 3 parámetros: Total TPS, Avg. Response time y Max Response time.

Los resultados obtenidos son los siguientes:

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%204/Tabla_OWL.png?raw=true)

La representación gráfica de los resultados de la tabla quedarían así:

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%204/Grafico1_OWL.png?raw=true)

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%204/Grafico2_OWL.png?raw=true)


Analizando todos los resultados podemos decir que la mejor opción es con un balanceador Nginx. En segundo lugar podríamos instalar HAProxy y como peor opción se podría tener un único servidor con Apache instalado.
