#Ejercicios Tema 7

##Ejercicio T7.1:
###¿Qué tamaño de unidad de unidad RAID se obtendrá al configurar un RAID 0 a partir de dos discos de 100 GB y 100 GB?
El RAID 0 mejora el rendimiento y aumenta la capacidad del disco final. Por tanto la capacidad final de la unidad RAID con dos discos de 100GB será de 200GB.
###¿Qué tamaño de unidad de unidad RAID se obtendrá al configurar un RAID 0 a partir de tres discos de 200 GB cada uno?
Como es un RAID como el anterior se suman la capacidad de los discos y obtendremos que la unidad RAID será de 600GB.

##Ejercicio T7.2:
###¿Qué tamaño de unidad de unidad RAID se obtendrá al configurar un RAID 1 a partir de dos discos de 100 GB y 100 GB?
La principal característica del RAID 1 es la réplica de datos en ambos discos. Por tanto la unidad RAID tendrá solo el valor de un disco, es decir, 100GB.

###¿Qué tamaño de unidad de unidad RAID se obtendrá al configurar un RAID 1 a partir de tres discos de 200 GB cada uno?
Como en el caso anterior se replican los datos por lo que la unidad RAID será de 200GB.

##Ejercicio T7.3: 
###¿Qué tamaño de unidad de unidad RAID se obtendrá al configurar un RAID 5 a partir de tres discos de 120 GB cada uno?
La unidad RAID sería de 240GB quedando un disco de 120GB como disco de paridad.

##Ejercicio T7.4:
###Buscar información sobre los sistemas de ficheros en red más utilizados en la actualidad y comparar sus características. Hacer una lista de ventajas e inconvenientes de todos ellos, así como grandes sistemas en los que se utilicen.
Un Sistema de archivos de red (NFS) permite a los hosts remotos montar sistemas de archivos sobre la red e interactuar con esos sistemas de archivos como si estuvieran montados localmente. Esto permite a los administradores de sistemas consolidar los recursos en servidores centralizados en la red. 
Existen 3 versiones: NFSv2, NFSv3 y NFSv4.

El sistema NFS está dividido al menos en dos partes principales: un servidor y uno o más clientes. Los clientes acceden de forma remota a los datos que se encuentran almacenados en el servidor.

Las estaciones de trabajo locales utilizan menos espacio de disco debido a que los datos se encuentran centralizados en un único lugar pero pueden ser accedidos y modificados por varios usuarios, de tal forma que no es necesario replicar la información.

Los usuarios no necesitan disponer de un directorio “home” en cada una de las máquinas de la organización. Los directorios “home” pueden crearse en el servidor de NFS para posteriormente poder acceder a ellos desde cualquier máquina a través de la infraestructura de red.

También se pueden compartir a través de la red dispositivos de almacenamiento como disqueteras, CD-ROM y unidades ZIP. Esto puede reducir la inversión en dichos dispositivos y mejorar el aprovechamiento del hardware existente en la organización.

###Configurar en una máquina virtual un servidor NFS. Montar desde otra máquina virtual en la misma subred la carpeta exportada y comprobar que ambas pueden acceder a la misma para lectura y escritura.

Lo primero que hacemos es descargar e instalar el servidor NFS. Lo realizamos mediante:

    apt-get install nfs-kernel-server portmap
    
Creamos un directorio, por ejemplo  el siguiente:

    mkdir /var/nfs/
    
Cambiamos los permisos de acceso al directorio.

    chown nobody:nogroup /var/nfs
    
Exportamos los directorios desde el fichero /etc/exports

    nano /etc/exports
    
Añadimos lo siguiente:

    /var/nfs        192.168.64.128(rw,sync,no_subtree_check)
    
Y aplicamos la configuración con exportfs -a.

Pasamos a la máquina que será el cliente e instalamos el cliente NFS.

    apt-get install nfs-common portmap

Montamos el directorio:

    mkdir -p /mnt/nfs/var/nfs
    mount 192.168.64.128:/var/nfs /mnt/nfs/var/nfs
    
Para comprobar su funcionamiento tal simple como crear un fichero en el servidor y en el directorio montado en el NFS cliente debe aparecernos si realizamos un ls en el directorio.
