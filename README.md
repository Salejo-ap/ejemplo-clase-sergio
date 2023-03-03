# Ejemplo usado en clase

Repositorio usado para el ejercicio visto en clase:
  - Ejecutar un contenedor con una carpeta compartida
  - Crear los scripts para crear y consultar una base de datos

---

## Ejecutar un contenedor con MySQL

Para este ejercicio se usará un contenedor ejecutando MySQL versión 5.7.39.

1. (Opcional) se puede descargar primero la imagen de MySQL para que quede en el caché local. 

    ```
    docker pull mysql:5.7.39
    ```

    **NOTA:** Existen varias etiquetas de la base de datos MySQL. Por ejemplo, existen las etiquetas `latest`, `5.7.39` y `5.7.39-oracle`. 

2. Ejecute un contenedor con la base de datos

    ```
    docker run -d --name mysql-db \
        -p 3306:3306  \
        -p 33060:33060 \
        -e MYSQL_ROOT_PASSWORD=secret \
        mysql:5.7.39
    ```

    | Parámetro                    | Descripción                                   |
    |------------------------------|-----------------------------------------------|
    | --name  mysql-db             | nombre del contenedor                         |
    | -p 3306:3306                 | expone el puerto de la base de datos MySQL    |
    | -e  MYSQL_ROOT_PASSWORD=...  | contraseña para el usuario root               |
    
    

## Ejecutando comandos SQL desde el contenedor del servidor

Es posible ejecutar comandos SQL usando el programa `mysql` incluido en la imagen de contenedor.

1. Ejecute el programa `mysql` en el contenedor `mysql-db`

    ```
    docker exec -it mysql-db mysql -p
    ```

    Al ejecutar el programa, este debe solicitar una contraseña. Use la contraseña `secret` definida en el paso anterior.

    ```
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 3
    Server version: 5.7.39 MySQL Community Server (GPL)

    Copyright (c) 2000, 2022, Oracle and/or its affiliates.

    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    mysql>
    ```    

    En el programa es posible ejecutar comandos MySQL e instrucciones SQL. Por ejemplo, es posible consultar las bases de datos instaladas.

    ```    
    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    +--------------------+
    4 rows in set (0.00 sec)
    ```    

    Por ejemmplo, es posible conocer la fecha actual usando la función `CURDATE()`

    ```
    mysql> select CURDATE();
    +------------+
    | CURDATE()  |
    +------------+
    | 2022-09-26 |
    +------------+
    1 row in set (0.00 sec)        
    ```    

    Para salir de `mysql` se puede ejecutar `\q`

    ```    
    mysql> \q
    Bye
    ```    

3. Es posible detener y eliminar el contenedor usando `docker rm`

    ```    
    # en línea de comandos
    docker rm -f mysql-db
    ```    

    ```
    # elimine todos los volumenes temporales
    docker volume prune
    ```

## Creando una carpeta compartida con el contenedor

Para trabajar más comodamente es posible crear una carpeta compartida entre el entorno de desarrollo y el contenedor. De esta forma, es posible editar el archivo en un entorno y ejecutar los programas dentro del contenedor.

1. Cree una carpeta `sql-scripts` en el directorio del proyecto (en el entorno de desarrollo)

    ```
    # en línea de comandos, por fuera del contenedor
    mkdir sql-scripts
    ```

2. Ejecute el contenedor incluyendo un volumen con la carpeta que acaba de crear.

    ```
    docker run -d --name mysql-db \
        -p 3306:3306 \
        -p 33060:33060 \
        -e MYSQL_ROOT_PASSWORD=secret \
        -v $(pwd)/sql-scripts:/sql-scripts \
        mysql:5.7.39
    ```

    | Parámetro                    | Descripción                                   |
    |------------------------------|-----------------------------------------------|
    | -v local:en-contenedor       | define un volumen compartido con el contenedor.                    |
    | ... $(pwd)                   | nombre completo de la carpeta actual |
    | ... $(pwd)/sql-scripts       | nombre completo de la carpeta `sql-scripts` en la carpeta local    |
    
3. Cree un archivo `ejemplo.sql` en la carpeta `sql-scripts/` con una consulta de SQL válida en MySQL.

    ```
    SELECT "hola mundo";
    ```

4. Ejecute `mysql` en el contenedor `mysql-db` que acaba de iniciar. Recuerde utilizar la contraseña `secret`.

    ```
    docker exec -it mysql-db mysql -p
    ```

5. En el MySQL, es posible ejecutar un script SQL usando  el comando `SOURCE` con el nombre del archivo. Ejecute `SOURCE` con el nombre de archivo `sql-scripts`.

    ```
    mysql> source /sql-scripts/ejemplo.sql
    +------------+
    | hola mundo |
    +------------+
    | hola mundo |
    +------------+
    1 row in set (0.00 sec)
    ```

3. En el entorno de desarrollo, modifique el `sql-scripts/ejemplo.sql` con una consulta SQL diferente en MySQL. Cambie "hola mundo" por "hola contenedor", para revisar que el cambio se puede ejecutar en el contenedor.

    ```
    SELECT "hola contenedor";
    ```

4. En el contenedor, ejecute de nuevo el script. Note que el resultado cambia debido a que se modificó el archivo `ejemplo.sql`

    ```
    mysql> source /sql-scripts/ejemplo.sql

    +-----------------+
    | hola contenedor |
    +-----------------+
    | hola contenedor |
    +-----------------+
    1 row in set (0.00 sec)    
    ```

5. Termine la sesión de MySQL y detenga el contenedor.

    ```
    mysql> \q
    Bye    
    ```

    ```
    # por fuera del contenedor
    docker rm -f mysql-db
    ```

## Creando scripts para crear una base de datos y unos datos iniciales

Es posible crear uno o más scripts en SQL para crear una base de datos y agregar unos datos iniciales al momento de iniciar el contenedor con MySQL. Para hacerlo, es recomendable tener una carpeta independiente con los archivos de incialización.

1. Cree una carpeta diferente `sql-init` en el entorno de desarrollo

    ```
    # en línea de comandos, por fuera del contenedor
    mkdir sql-init
    ```

2. Cree un archivo `1.crear-tablas.sql` en la carpeta `sql-init` donde se creen una base de datos y unas tablas en su interior

    ```
    CREATE TABLE empleados (
        primer_nombre   varchar(25),
        segundo_nombre  varchar(25),
        departamento    varchar(15),
        email           varchar(50)
    );
    ```
3. Cree un archivo `2.insertar-datos.sql` en la carpeta `sql-init` donde se inserten algunos datos en la tablas.

    ```
    INSERT INTO empleados (primer_nombre, segundo_nombre, departamento, email)
        VALUES ('Bill', 'Gates', 'IT', 'billg@ejemplo.com');

    INSERT INTO empleados (primer_nombre, segundo_nombre, departamento, email)
        VALUES ('Steve', 'Jobs', 'Ventas', 'steve.jobs@ejemplo.com');
    ```

4.  Ejecute el contenedor definiendo la carpeta con los scripts iniciales y con el nombre de la base de datos.

    ```
    docker run -d --name mysql-db \
        -p 3306:3306 \
        -p 33060:33060 \
        -e MYSQL_ROOT_PASSWORD=secret \
        -e MYSQL_DATABASE=empleados \
        -v $(pwd)/sql-scripts:/sql-scripts \
        -v $(pwd)/sql-init:/docker-entrypoint-initdb.d \
        mysql:5.7.39
    ```

    | Parámetro                    | Descripción                                   |
    |------------------------------|-----------------------------------------------|
    | -e MYSQL_DATABASE=...        | define el nombre de la base de datos a crear dentro de MySQL          |
    | -v ...:/docker-entrypoint-initdb.d       | carga una carpeta compartida en la carpeta `/docker-entrypoint-initdb.d`, donde se tienen los scripts que se ejecutan al iniciar la base de datos.   |
    
5. Ejecute MySQL en el contenedor. Use la contraseña `secret`.

    ```
    docker exec -it mysql-db mysql -p
    ```

6. Revise las bases de datos y los datos en la base de datos `empleados`

    ```
    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | empleados          |
    | mysql              |
    | performance_schema |
    | sys                |
    +--------------------+
    5 rows in set (0.01 sec)

    mysql> use empleados
    Database changed

    mysql> show tables;
    +---------------------+
    | Tables_in_empleados |
    +---------------------+
    | empleados           |
    +---------------------+
    1 row in set (0.00 sec)

    mysql> select * from empleados;
    +---------------+----------------+--------------+------------------------+
    | primer_nombre | segundo_nombre | departamento | email                  |
    +---------------+----------------+--------------+------------------------+
    | Bill          | Gates          | IT           | billg@ejemplo.com      |
    | Steve         | Jobs           | Ventas       | steve.jobs@ejemplo.com |
    +---------------+----------------+--------------+------------------------+
    2 rows in set (0.00 sec)
    ```
