
<style>
.red{
    color:red;
}
</style>
# Instalación de Mongo DB en Server de Producción

#### <b class="red">Nota:</b> La configuración se ha realizado utilizando Droplets, el proveedor de servidores Digital Ocean con la versión de MongoDB 4.4 y una versión de Centos RHEL 8.

<br>
<br>

# Instalación y configuración

**1. Creamos un archivo en el siguiente directorio para instalar MongoDB directamente usando yum.**

```shell
[root@server]# vim /etc/yum.repos.d/mongodb-org-4.4.repo
```

**2. Dentro del archivo colocamos lo siguiente:**

```shell
[mongodb-org-4.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.4.asc
```
Guardamos los cambios del archivo.

**3. Actualizamos YUM**

```shell
[root@server]# yum update
```

**4. Instalamos MongoDB**
```shell
[root@server]# sudo yum install -y mongodb-org
```

**5. Etiquetamos el puerto que vayamos a utilizar para mongo, `se recomienda no usar` el puerto por defecto por seguridad para este ejemplo usaremos el puerto 27064**

```shell
[root@server]# sudo semanage port -a -t mongod_port_t -p tcp 27064
```

**6. Agregamos permisos al firewall sobre el puerto a utilizarse:**
```shell
[root@server]# sudo firewall-cmd --add-port=27064/tcp --permanent 
```
Recargamos el servicio de firewall

```shell
[root@server]# sudo firewall-cmd --reload
```

**7. También se puede limitar el acceso según la dirección ip de origen:**

La `ip_address` es la direccion ip desde donde se desea acceder a la base de datos.
```shell
[root@server]# sudo firewall-cmd --permanent --add-rich-rule "rule family="ipv4" \
source address="ip_address" port protocol="tcp" port="27064" accept"
```

**8.	Habilitamos acceso fuera del server:**

Editamos el siguiente archivo:

```shell
[root@server]# vim /etc/mongod.conf  
```

Dentro del archivo agregamos la direccion ip del server donde estamos instalando Mongo.

Remplazamos `IP_SERVER` por nuestra direccion ip del servidor

El archivo debe quedar de la siguiente manera:

```shell
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# Where and how to store data.
storage:
  dbPath: /var/lib/mongo
  journal:
    enabled: true
#  engine:
#  wiredTiger:

# how the process runs
processManagement:
  fork: true  # fork and run in background
  pidFilePath: /var/run/mongodb/mongod.pid  # location of pidfile
  timeZoneInfo: /usr/share/zoneinfo

# network interfaces
net:

  port: 27064
  bindIp: 127.0.0.1,IP_SERVER

```

**9. Recargamos deamon**

```shell
[root@server]# systemctl daemon-reload
```

**10. Inicializamos y habilitamos mongo database:**

```shell
[root@server]# start mongod
```

```shell
[root@server]# enable  mongod
```
<br>
<br>

# Creación de usuarios en MongoDB

**1. Creación de `Usuario Global` con restricciones de autenticación por ip**

Ingresamos a Mongo con el puerto que lo asignamos anteriormente

```shell
[root@server]# mongo --port 27064 
```

**2. Ingresamos a la base admin para crear el usuario:**

```shell
> use admin
```

<b class="red">Nota:</b> La siguiente operación crea un usuario. Este usuario solo puede autenticarse si se conecta de una dirección IP `XXX.X.X.X` a otra `ZZZ.ZZ.ZZZ.Z`

```shell
> db.createUser({user: "your_global_user", pwd: "your_password", roles: [{role: "root", db: "admin"}],authenticationRestrictions:[{clientSource:[ "XXX.X.X.X","add_other_ip" ],serverAddress:[ "ZZZ.ZZ.ZZZ.Z" ]}]})
```

**3. Crearemos un `usuario` para una base de datos en específico por seguridad**

```shell
> use nueva_base
```

```shell
> db.createUser({user: "your_user", pwd: "your_password", roles: [{role: "readWrite", db: "nueva_base"}],authenticationRestrictions:[{clientSource:[ " XXX.X.X.X ","add_other_ip" ],serverAddress:[ " ZZZ.ZZ.ZZZ.Z" ]}]})
```

**4. Editamos nuevamente el archivo `mongod.conf`**

```shell
[root@server]# vim /etc/mongod.conf 
```

**5. Agregamos lo siguiente al final del archivo**

```shell
security:
  authorization: enabled
```

**6. El archivo `mongod.conf` deberia quedar de la siguiente manera:**

```shell
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# Where and how to store data.
storage:
  dbPath: /var/lib/mongo
  journal:
    enabled: true
#  engine:
#  wiredTiger:

# how the process runs
processManagement:
  fork: true  # fork and run in background
  pidFilePath: /var/run/mongodb/mongod.pid  # location of pidfile
  timeZoneInfo: /usr/share/zoneinfo

# network interfaces
net:

  port: 27064
  bindIp: 127.0.0.1,IP_SERVER

security:
  authorization: enabled

```

**7. Reiniciamos Mongo**

```shell
[root@server]# systemctl restart mongod
```
<br>
<br>

# Agregamos Seguridad en el Droplet de Digital Ocean

**1. Creamos unas reglas del firewall de la siguiente manera:**

Ingresamos al Droplet y damos clic en Networking:

![Image of config firewall in droplet](https://firebasestorage.googleapis.com/v0/b/documentation-github.appspot.com/o/img-1.png?alt=media&token=1539000a-25b3-4b39-a8c0-288ea1939f8d)

Tenemos un apartado de Firewalls le damos a crear reglas firewall e ingresamos las reglas que necesites para tu servidor, en este caso se agregaron las básicas y en los espacios en blanco de la imagen deberían ir las IP permitidas para poder acceder a tu server y como tal a la base de datos por el puerto anteriormente configurado:

![Image of config firewall in droplet](https://firebasestorage.googleapis.com/v0/b/documentation-github.appspot.com/o/img-2.png?alt=media&token=1e95d2e4-afc8-42d4-bea8-a16480c3183f)




