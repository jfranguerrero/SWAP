#Práctica 6. Discos en RAID

En esta última práctica se va a explicar como se crea un RAID 1 mediante software en una máquina con Ubuntu Server. Posteriormente se automatizará para que tenga un funcionamiento correcto desde que se enciende el servidor. Finalmente simularemos un fallo en un disco del RAID para demostrar su eficacia. Veremos que la máquina puede seguir funcionando con normalidad sin perder información y que podremos añadir un disco nuevo el cual funcionará en el mencionado RAID.

##Configuración de RAID 1 en Ubuntu Server mediante software
Desde las opciones de la máquina virtual creamos un nuevo disco duro virtual del mismo tamaño que el existente. Concluido esto arrancamos la máquina.

Instalamos mdadm que es el software encargado de la configuración de discos duros RAID.

    sudo apt-get install mdadm
    
Usamos fdisk para saber la información de ambos discos.

    sudo fdisk -l 

Nos aparecerá una pantalla similar a esta:

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%206/IMG_1.png?raw=true)

Vemos que nuestros dos discos son sdb y sdc. El RAID 1 lo llevamos a cabo mediante el siguiente comando:

    sudo mdadm -C /dev/md0 --level=raid1 --raid-devices=2 /dev/sdb /dev/sdc
    
![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%206/IMG_2.png?raw=true)

Le aplicamos formato al sistema RAID 1 creado mediante:

    sudo mkfs /dev/md0
    
Como resultado obtendremos lo siguiente:

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%206/IMG_3.png?raw=true)

Creamos el directorio donde se montará el sistema RAID y lo montamos.

    sudo mkdir /datos
    sudo mount /dev/md0 /datos
    
Como método de comprobación para saber si el RAID se realizó correctamente ejecutaremos lo siguiente:

    sudo mdadm --detail /dev/md0

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%206/IMG_4.png?raw=true)


##Automatización del RAID 1

La automatización del montaje del RAID al inicio del sistema es muy sencillo. Editamos el fichero /etc/fstab y añadimos la siguiente línea para que se monte de forma automática.

    /dev/md0 /datos ext2 noatime,rw 0 0
    
Reiniciamos la máquina y ya debería montarse el RAID de forma automática.


##Simulación de un fallo en un disco RAID, comprobar su funcionamiento sin él y repararlo

Vamos a generar un fallo en el RAID. Lo produciremos en el disco sdb. (Se ha cambiado el nombre del RAID de md0 a md127 para realizar previamente una prueba por si era irreparable).

    mdadm --manage --set-faulty /dev/md127 /dev/sdb

Por tanto este disco lo eliminamos con el siguiente comando para que el RAID desaparezca y el disco duro sdc sea funcional.

    mdadm --remove /dev/md127 /dev/sdb
    
En los detalles del RAID apreciamos que solo hay un disco activo actualmente.

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%206/IMG_5.png?raw=true)

Para volver a añadir el disco le indicamos al RAID la unidad de disco que debe sincronizar con el RAID.

    mdadm --add /dev/md127 /dev/sdb
    
  ![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%206/IMG_6.png?raw=true) 
  
 El porcentaje de rebuild irá ascendiendo hasta el 100% que es cuando funcionará correctamente.
    