#Práctica 5
En esta práctica se va a tratar la creación de copias de seguridad de las bases de datos en MySQL. Dichas copias la realizaremos con mysqldump y las restauraremos en la segunda máquina. Finalmente estos pasos se automatizarán con una configuración maestro-esclavo lo que nos proporcionará que esta replicación de datos se produzca en un determinado intervalo de tiempo y sin la necesidad de que tengamos que interactuar con el servidor.

##Creación de base de datos
No podemos llevar a cabo la práctica si no disponemos una base de datos y tuplas en su interior. Por eso en la máquina 1 crearemos una simple base de datos (sugerida por el profesor) y meteremos una pequeña cantidad de valores.

Lo primero que debemos hacer es acceder a MySQL. Esto lo hacemos con:

    mysql -uroot -p
En el caso de que indicásemos una contraseña en el momento de la instalación deberemos introducirla.

Crearemos una base de datos de contactos y una tabla datos donde insertaremos varias tuplas. Los parámetros a introducir son estos:

    create database contactos;
    use contactos;
    create table datos(nombre varchar(100),tlf int);
    insert into datos(nombre,tlf) values ("pepe",95834987);
    insert into datos(nombre,tlf) values ("juan",95831127);
    insert into datos(nombre,tlf) values ("antonio",959999999);
    
Comprobamos si se han guardado las tablas y los valores. Debe aparecernos algo similar a lo de la siguiente captura.

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%205/Foto_1.png?raw=true)

##Obtención de backup de la base de datos
Para realizar una réplica de la base de datos usaremos mysqldump. Este nos hará una copia en un fichero de la base de datos deseada. Para evitar que se modifique la base de datos mientras realizamos la copia bloqueamos el acceso a la base de datos. Esto lo hacemos accediendo con nuestro usuario a MySQL y con:

    FLUSH TABLES WITH READ LOCK;
    
Ya podemos hacer el backup. A mysqldump le indicamos el nombre de la base de datos, el usuario de MySQL y la ruta de salida del fichero sql. En nuestro caso será lo siguiente:
    mysqldump contactos -u root -p > /root/contactos.sql
    
Hacemos un cat al fichero sql generado y nos debe aparecer algo parecido a lo de la captura.

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%205/Foto_2.png?raw=true)

Solo nos queda desbloquear la base de datos que se realizará en MySQL con:

    UNLOCK TABLES;
    


##Restauración de la base de datos en la segunda máquina

Accederemos a la segunda máquina y copiaremos en esta la base de datos dumpeada. Esto lo llevamos a cabo con el siguiente comando:

    scp root@192.168.64.128:/root/contactos.sql /root/
    
Ahora en nuesta máquina accedemos a MySQL y creamos una base de datos. Estos son los pasos a seguir:
    mysql -u root –p
    CREATE DATABASE contactos;

En la base de datos creada cargaremos la copia que tenemos almacenada en el fichero.

    mysql -u root -p contactos < /root/contactos.sql
    
Accedemos a MySQL y realizamos un select * from datos. Debe aparecernos lo siguiente:

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%205/Foto_3.png?raw=true)

##Replicación automática de la base de datos

Ya solo nos queda automatizar todo lo anterior. Para ello usaremos una configuración maestro-esclavo.

En la máquina principal editamos el fichero /etc/mysql/my.cnf. Comentamos el parámetro de bind-address. Le indicamos un fichero para la generación de errores si no está creado por defecto, establecemos server-id = 1 y descomentamos log_bin.
Guardamos y reiniciamos el servicio MySQL.

Pasamos a la segunda máquina y volvemos a editar el fichero /etc/mysql/my.cnf. Pondremos lo mismo que en el anterior salvo server-id que será 2.

Pasamos a la primera máquina y entramos en MySQL para crear un usuario que posea permisos de replicación. Debemos introducir lo siguiente:

    CREATE USER esclavo IDENTIFIED BY 'esclavo';
    GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%' IDENTIFIED BY 'esclavo';
    FLUSH PRIVILEGES;
    FLUSH TABLES;
    FLUSH TABLES WITH READ LOCK;
    SHOW MASTER STATUS;
    
Concluido esto accedemos de nuevo al MySQL de la segunda máquina (esclava). Introduciremos la IP del maestro, su puerto, el usuario, la contraseña, el fichero de la base de datos y su posición. En nuestro caso sería así:

    CHANGE MASTER TO MASTER_HOST='192.168.64.128', MASTER_USER='esclavo', MASTER_PASSWORD='esclavo', MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=501, MASTER_PORT=3306;
    
Ahora activamos el esclavo con start slave. Para asegurarnos que está funcionando de forma correcta introduciremos el siguiente comando y comprobaremos que el valor de Second_Behind_Master es distinto de "null".

    SHOW SLAVE STATUS\G 
    
En la siguiente imagen se aprecia que funciona de forma correcta.

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%205/Foto_4.png?raw=true)

Como última comprobación crearemos alguna tabla e insertaremos varias tuplas en la máquina maestro y comprobaremos que se replican en el eaclavo.

![alt tag](https://github.com/jfranguerrero/SWAP/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%205/Foto_5.png?raw=true)
