#Ejercicios Tema 4

##Ejercicio T4.1: Buscar información sobre cuánto costaría en la actualidad un mainframe. Comparar precio y potencia entre esa máquina y una granja web de unas prestaciones similares. 

Se adjunta la noticia de la presentación del nuevo modelo de IBM donde el precio de la versión más baja rondará los 200.000$
http://www.ticbeat.com/tecnologias/ibm-reinventa-el-mainframe-en-la-era-movil-z13/

Es muy complicado realizar comparaciones de especificiones y precios debido a que los fabricantes no los exponen de forma pública si no que dichos precios se negocian de forma personal con las grandes empresas interesadas.

##Ejercicio T4.2: Buscar información sobre precio y características de balanceadores hardware específicos. Compara las prestaciones que ofrecen unos y otros. 

    BBF240A Barracuda Load Balancer 240 Appliance
    Manufacturer	Barracuda Networks
    Video	No
    Maximum Throughput	95 Mbps
    Servers Support	10
    SSL Offloading/Acceleration	No
    Features	No
    Precio $1,424.05

    BBF643A Barracuda Load Balancer 640 ADC w/ 8 1GbE NICs
    Manufacturer	Barracuda Networks
    Video	No
    Maximum Throughput	4 Gbps
    Servers Support	250
    SSL Offloading/Acceleration	15,000 TPS
    Features	No
    Precio $12,000


##Ejercicio T4.3: Buscar información sobre los métodos de balanceo que implementan los dispositivos recogidos en el ejercicio 4.2.

Según la documentación de Barracuda:Barracuda Load Balancer can distribute traffic based on several scheduling algorithms, including round robin, server weighting and least connections. 

Esto está indicado en el siguiente pdf: https://www.barracuda.com/assets/docs/datasheets/barracuda_load_balancer_ds_us.pdf
##Ejercicio T4.4: Instala y configura en una máquina virtual el balanceador ZenLoadBalancer. 

Creamos una nueva máquina virtual en VMPlayer. Descargamos la iso de ZenLoadBalancer y la montamos en la máquina virtual. Nos mostrará la siguiente pantalla:

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Ejercicios_Clase/ZenLoadBalancer/zen_1.png?raw=true)

Seguimos los típicos pasos que en todas las versiones de Linux donde inicialmente seleccionamos el idioma de la distribución.

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Ejercicios_Clase/ZenLoadBalancer/zen_2.png?raw=true)

A continuación le indicamos la dirección IP de la máquina.

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Ejercicios_Clase/ZenLoadBalancer/zen_3.png?raw=true)

Seguimos con los pasos siguientes donde el servidor de nombres en nuestro caso lo dejamos vacío.

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Ejercicios_Clase/ZenLoadBalancer/zen_4.png?raw=true)

Creamos las particiones y puntos de montaje del sistema y continuamos.

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Ejercicios_Clase/ZenLoadBalancer/zen_5.png?raw=true)

Para configurarlo entramos en https://192.168.64.134:444/ e introducimos como usuario admin y como contraseña admin. Nos aparecerá la siguiente ventana:

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Ejercicios_Clase/ZenLoadBalancer/zen_6.png?raw=true)

Vamos a Manage- Farm e introducimos un nombre para la granja. Le asignamos la IP virtual y guardamos.

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Ejercicios_Clase/ZenLoadBalancer/zen_7.png?raw=true)

Editamos la granja y añadimos Real Servers donde introducimos las IPS, puertos y el peso que tiene en el balanceo. De esta forma tendremos el balanceo de carga hacia esos dos servidores. Arriba en Load Balance Algorithm podremos elegir el tipo de algoritmo de balanceo que queramos, desde round robin hasta priorizar determinados servidores. 

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Ejercicios_Clase/ZenLoadBalancer/zen_8.png?raw=true)

##Ejercicio T4.5: Probar las diferentes maneras de redirección HTTP. ¿Cuál es adecuada y cuál no lo es para hacer balanceo de carga global? ¿Por qué? 
Redirección 301. Es permanente. Esto le indica a los buscadores que ignoren la dirección original e indexen la nueva dirección. Este tipo de redirección se utiliza cuando estamos cambiando de dominio a una web, esto haría que los buscadores indexen la nueva dirección sin perder los enlaces entrantes que tenía el dominio y sin perder el pagerank.

Redirección 302. Es temporal. Esto indica que la dirección original no ha cambiado y se seguirá utilizando pero temporalmente cambiaremos de dirección.

En mi opinión la adecuada es la 302 ya que sería posible redirigir el tráfico a difernetes direcciones temporales y no una permanente la cual también la actualizan navegadores.

##Ejercicio T4.6: Buscar información sobre los bloques de IP para los distintos países o continentes. Implementar en JavaScript o PHP la detección de la zona desde donde se conecta un usuario.
Las direcciones IP se reparten a países por bloques. Existen diversas bases de datos que identifican que que país es cada bloque. La mayoría son de pago pero GeoLite2 si es gratuita. Su web es http://dev.maxmind.com/geoip/geoip2/geolite2/

En esto es en lo que se basan la mayoría de webs donde consultas tu IP y te dicen tu ubicación.

La web http://chir.ag/projects/geoiploc tiene desarrollado un script en php que nos muestra nuestra IP y su país. El código es el siguiente:

```ssh
<?php
error_reporting(E_ALL & ~E_NOTICE);
include("geoiploc.php"); 
  if (empty($_POST['checkip']))
  {
        $ip = $_SERVER["REMOTE_ADDR"]; 
  }
  else
  {
        $ip = $_POST['checkip']; 
  }
?> 
Tu dirección IP es: <?php echo($ip); ?> <br>
Tu País es : <?php echo(getCountryFromIP($ip, " NamE"));?>
 (<?php echo(getCountryFromIP($ip, "code"));?>)
```


##Ejercicio T4.7: Buscar información sobre métodos y herramientas para implementar GSLB. 

GSLB se basa en técnicas como la redirección de DNS, redirección HTTP, RHI ,etc para conseguir su objetivo. Normalmente GSLB se configura en la configuración del centro de datos activo-activo o configuración activo-standby. En configuración activo-activo las solicitudes de los clientes se equilibra su carga a través de los centros de datos activos. En configuración activo-standby, sólo en el fracaso del centro de datos activo, el centro de datos de espera entra en funcionamiento. Controladores de entrega de aplicaciones (ADC) que proporcionan SLB ( equilibrio de carga del servidor) también suelen proporcionar funcionalidades al GSLB.