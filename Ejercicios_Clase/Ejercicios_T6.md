#Ejercicios Tema 6

##Ejercicio T6.1: Aplicar con iptables una política de denegar todo el tráfico en una de las máquinas de prácticas. Comprobar el funcionamiento. 

Introducimos en el terminal el siguiente comando para bloquear todo el tráfico entrante.

    iptables -A INPUT -j DROP
    
Mediante iptables -L podemos mostrar el tráfico bloqueado y el permitido. Como comprobación el bloqueo lo estaba realizando mediante Putty y en el momento de ejecutar la orden me apareció el error de que la conexión se había abortado.

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Ejercicios_Clase/Ejercicio_6_1a.png?raw=true)

##Ejercicio T6.1b: Aplicar con iptables una política de permitir todo el tráfico en una de las máquinas de prácticas. Comprobar el funcionamiento.

Es muy similar al caso anterior salvo que DROP se sustituye por ACCEPT. Por tanto la orden sería la siguiente:

    iptables -A INPUT -j ACCEPT
    
No podemos olvidar que debemos eliminar la tabla con drop creada previamente. Se hace con:

    
    iptables -D INPUT-j DROP
    
Ya Putty nos funciona correctamente.

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Ejercicios_Clase/Ejercicio_6_1b.png?raw=true)


##Ejercicio T6.2: Comprobar qué puertos tienen abiertos nuestras máquinas, su estado, y qué programa o demonio lo ocupa.

Para comprobarlo usaremos netstat. Concretamente en terminal introduciremos:

    netstat -tulpn
    
Nos mostrará algo similar a lo siguiente:

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Ejercicios_Clase/Ejercicio_6_2.png?raw=true)


##Ejercicio T6.3: Buscar información acerca de los tipos de ataques más comunes en servidores web, en qué consisten, y cómo se pueden evitar.

Existen diversos tipos de ataques pero los más comunes son los siguientes:

Por inyección. Pretende modificar una cadena de consulta de base de datos mediante la inyección de código en la consulta. Es de los ataques más fáciles de llevar a cabo. Esto permite acceder a toda la información de una empresa la cual se almacena en su base de datos.

DDoS. Lo que hace la denegación de servicio es imposibilitar el acceso a una página web. Se generan multitud de peticiones hacia el servidor con la finalidad de desbordar el ancho de banda o consumir todos los recursos de red. De esta forma la web queda inaccesible para cualquier usuario normal que intente acceder al contenido.

Fuerza bruta. Se basa en obtener el usuario y contraseña probando todas las posibilidades posibles de claves alfanuméricas. Generalmente se consiguen obtener contraseñas sencillas debido a que se necesita un menor tiempo al ser normalmente fechas o palabras comunes.

Cross Site Scripting. Es la inyección de scripts maliciosos en webs de confianzas para que los visitantes lo ejecuten creyendo que es de confianza y así los hackers tengan acceso a las cookies o tokens generados por la web presuntamente confiable.