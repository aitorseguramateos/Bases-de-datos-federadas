Primero de todo, debemos instalar el mysql en las máquinas para poder tener conexión a una base de datos.
```
sudo apt install mysql-server
```
![Alt text](image.png)

Una vez tenemos instalado el mysql en la máquina servidor y en la máquina cliente, vamos a configurar el archivo mysqld.cnf de cada máquina.
```
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
Dentro de este debemos comentar lo siguiente:
```
#bind-address           = 127.0.0.1
#mysqlx-bind-address    = 127.0.0.1
```
![Alt text](image-1.png)

Finalmente escribiremos lo siguiente al final del documento:

```
federated
```
![Alt text](image-2.png)

Una vez lo hayamos modificado todo, guardaremos los cambios y saldremos. Seguidamente reiniciaremos el servicio de mysql.

```
sudo service mysql restart
```
![Alt text](image-3.png)

Seguidamente miraremos el estado del servicio de este.

```
sudo service mysql status
```
![Alt text](image-4.png)

Una vez veamos que está corriendo y está todo correindo, iniciaremos el mysql como root para tener todos los permisios.

```
sudo mysql -u root
```
![Alt text](image-5.png)

Una vez hayamos logeado como administrador miraremos si tiene algún engine de bases de datos federadas.

```
show engines\s;
```
![Alt text](image-6.png)

Al comprobar que que tenemos el engine FEDERATED, crearemos un shema para la base de datos.
```
crate schema pineapple;
```
![Alt text](image-7.png)

Una vez creado, crearemos un usuario (Comercial) en la máquina servidor con la ip del cliente, y le atribuiremos los permisos de lectura.
```
create user 'comercial'@'192.168.1.66' identified by 'comercial';
grant select on pineapple.*to 'comercial'@'192.168.1.66';
```
![Alt text](image-8.png)

Una vez creado el usuario y le hemos dado sus permisos correspondientes, crearemos la tabla clientes.
```
use pineapple;
create table clientes(DNI int(8), nombre varchar(50), apellido1 varchar(50), apellido2 varchar(50), direccion varchar(50), mail varchar(100), telefono int, movil int);
```
![Alt text](image-9.png)

Una vez tenemos creada la tabla, iremos a la máquina cliente, y crearemos el usuario.
```
create user 'comercial'@'localhost' identified by 'comercial';
grant select on pineapple.*to 'comercial'@'localhost';
```
![Alt text](image-10.png)

Una vez tenemos creado el usuario, y le hemos dado los permisos de lectura, realizaremos la conenxión.
```
use pineapple;
create table clientes(dni int, nombre varchar(50), apellido1 varchar(50), apellido2 varchar(50), direccion varchar(50), mail varchar(100), telefono int, movil int) egine=federated connection='mysql://comercial:comercial@192.168.1.33:3306/pineapple/clientes';
```
![Alt text](image-11.png)
![Alt text](image-12.png)

Para comprobar que se ha establecido conexion correctamente, realizaremos los siguientes comandos desde la máquina cliente:

```
show tables;
select * from clientes;
```
![Alt text](image-13.png)

Para comprobar el permio de edición, realizarmos los siguientes comados:
```
insert into clientes(nombre) values("manolo");
```
![Alt text](image-14.png)

Hemos podido ver que no nos deja añadir reguistros, por lo tanto hemos aplicado bien los permisos.