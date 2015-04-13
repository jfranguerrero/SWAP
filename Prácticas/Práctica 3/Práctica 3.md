#Práctica 3
En esta tercera práctica instalaremos dos tipos de balanceadores de carga (nginx y haproxy) en una nueva máquina virtual con características similares a las dos que tenemos creadas previamente exceptuando la existencia de Apache para si tener disponible el puerto 80.

Creada ya la mencionada máquina comenzaremos instalando el servidor web nginx para posteriormente usarlo como balanceador de carga.

##Nginx

###Instalación
Como dice el guión de prácticas añadimos la clave del repositorio donde esta nginx. Esto lo realizamos introduciendo los siguientes comandos:
```sh
cd /tmp/
wget http://nginx.org/keys/nginx_signing.key
apt-key add /tmp/nginx_signing.key
rm -f /tmp/nginx_signing.key
```

Acto seguido añadimos añadimos el repositorio de nginx a nuestra lista de repositorios. Para ello modificamos el fichero /etc/apt/sources.list.
```sh
echo "deb http://nginx.org/packages/ubuntu/ lucid nginx" >> /etc/apt/sources.list
echo "deb-src http://nginx.org/packages/ubuntu/ lucid nginx" >> /etc/apt/sources.list
```

El siguiente paso es instalar nginx. Simplemente lo instalamos como cualquier otra aplicación con apt-get install nginx.

En la imagen podemos apreciar que se instaló correctamente.
![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/nginx_install.png?raw=true)


###Configuración Round-Robin
Pasamos a configurar nginx. Con un editor cualquiera abrimos el fichero de configuración de nginx situado en la ruta /etc/nginx/conf.d/default.conf

Añadimos las direcciones IPs de los servidores que van a funcionar como cluster, es decir, en los que habíamos instalado Apache en la primera práctica. Se añadiría lo siguiente:
```sh
upstream apaches {
 server 192.168.64.128;
 server 192.168.64.129;
}
```
Indicamos que los backends sean HTTP 1.1 y se borra la cabecera Connection. Nos quedaría algo así en nuestro fichero.
```sh
server{
 listen 80;
 server_name localhost;
 access_log /var/log/nginx/host.access.log;
 error_log /var/log/nginx/host.error.log;
 root /var/www/;
 location /
 {
 proxy_pass http://apaches;
 proxy_set_header Host $host;
 proxy_set_header X-Real-IP $remote_addr;
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 proxy_http_version 1.1;
 proxy_set_header Connection "";
 }
}
```
Sería una configuración round-robin con prioridad similar entre ambas máquinas. Cuando comprobemos que funciona correctamente se realizará una configuración con ponderación.

###Comprobación
Reiniciamos el servicio nginx y accedemos a una de las máquinas de nuestro cluster. Modificamos un solo index.html para poder distinguir entre ambas máquinas. En mi caso en el index de la segunda máquina indiqué que es dicha máquina. Esto lo guardamos y accedemos rápidamente a la terminal de nuestra máquina antitriona para comprobar que funciona correctamente antes de que se clonen los directorios de ambas máquinas.

Realizamos curl http://192.168.64.130 dos veces y debería realizar en cada solicitud una petición a una máquina distinta.

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/nginx_roundrobin.png?raw=true)

Comprobando el contenido de las dos peticiones podemos aprenciar que los html son distintos lo que nos indica que el tráfico se ha repartido entre las dos máquinas.

###Configuración Ponderación

Ahora se realizará una nueva versión en la que se usará un algoritmo de ponderación y en la que supondremos que la máquina 1 posee el doble de capacidad que la máquina 2.

Partiendo de la configuración Round-Robin la modificación es muy sencilla. Abrimos con un editor el fichero /etc/nginx/conf.d/default.conf y buscamos las IPs de nuestros servidores. Con weight indicaremos junto a la IP del servidor, que dicho servidor tiene una mayor capacidad de carga. Para el doble simplemente poniendo el valor de weight a 2. Quedaría de la siguiente manera:

```sh
upstream apaches {
 server 192.168.64.128 weight=2;
 server 192.168.64.129 weight=1;
}
```
Guardamos los cambios y reiniciamos de nuevo el servicio nginx.

###Comprobación

Desde nuestra máquina anfitriona usamos 3 veces de forma consecutiva curl http://192.168.64.130

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/nginx_ponderacion.png?raw=true)

Vemos en la imagen que la primera petición se realizó a la máquina 2 mientras que las dos siguientes fueron a la máquina 1. Esto corrobora que nuestra configuración funciona correctamente ya que nginx le asigna un mayor tráfico a la primera máquina ya que posee una capacidad superior.

##HaProxy

###Instalación

Lo primero que hacemos es parar el servicio nginx en nuestra tercera máquina para dejar libre el puerto 80. Lo realizamos con service nginx stop.
A continuación instalamos HaProxy con apt-get introduciendo lo siguiente: sudo apt-get install haproxy joe.
En mi caso el editor joe no estaba en ninguno de mis repositorios por lo que tuve que añadir el repositorio universe. Esto lo llevé a cabo mediante echo "deb http://archive.ubuntu.com/ubuntu $(lsb_release -sc) universe" >> /etc/apt/sources.list.

En la siguiente imagen ya nos aparece que se ha instalado.

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/haproxy_install.png?raw=true)
###Configuración Round-Robin

Abrimos con el editor joe el fichero de configuración de HaProxy llamado haproxy.cfg.
La configuración simple para un balanceador Round-Robin sería similar al siguiente código:

```sh

global
	daemon
	maxconn 256

defaults
	mode http
	contimeout 4000
	clitimeout 42000
	srvtimeout 43000

frontend http-in
	bind *:80
	default_backend servers

backend servers
	server m1 192.168.64.128:80 maxconn 32
	server m2 192.168.64.129:80 maxconn 32
```

Donde hemos indicado las IPs y los puertos de los servidores donde está funcionando Apache server.
Ahora lanzamos HaProxy mediante el siguiente comando y pasamos a la comprobación.
``
sudo /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg
``

Aplicamos el fichero de configuración a HaProxy.

/usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg
###Comprobación

Reiniciamos el servico HaProxy y vamos a la máquina anfitriona para realizar varios curl http://192.168.64.130
![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/haproxy_roundrobin.png?raw=true)
Se aprecia una alternancia de peticiones entre ambas máquinas.
###Configuración Ponderación
En este caso suponemos que nuestra máquina 1 posee el doble de capacidad que la máquina 2. Por tanto configuraremos HaProxy para que la máquina 1 acepte el doble de peticiones que la máquina 2.

```sh

global
	daemon
	maxconn 256

defaults
	mode http
	contimeout 4000
	clitimeout 42000
	srvtimeout 43000

frontend http-in
	bind *:80
	default_backend servers

backend servers
	server m1 192.168.64.128:80 weight 2
	server m2 192.168.64.129:80 weight 1
```

Hemos asignado el doble de capacidad a la máquina 1 mediante weight 2. Aplicamos de nuevo el fichero de configuración y reiniciamos el servicio.

###Comprobación
Vamos a la máquina anfitriona y realizamos varios curl http://192.168.64.130 de forma consecutiva.
![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/haproxy_ponderacion.png?raw=true)

Apreciamos que la máquina 2 solo se cagó una vez mientras que la 1 se mostró dos veces.