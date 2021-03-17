# Publicar una APP usando Nginx en centos 8  (Host DigitalOcean)

### La App a publicar usa las siguientes tecnologías:

<br>


**Backend:** Framework NestJS (NodeJs)

**Frontend:** Angular 9
<br>
<br>

# Configuración

**1. Prerequisitos**

- Tener ejecutado el backend y haber hecho el ng build –prod para obtener la carpeta dist
- Tener dominio y subdominios creados previamente ejemplo:


![Image of config dns](https://firebasestorage.googleapis.com/v0/b/documentation-github.appspot.com/o/img-3.png?alt=media&token=70cf5cfb-4d55-4fd6-b8fb-f9a12a9c47d3)

**2. Creación de carpeta para Frontend**
#### Creamos la carpeta donde tendremos que colocar los archivos de la carpeta dist del frontend cambiar `example.com` por tu dominio: ####


```shell
[root@server]# sudo mkdir -p /var/www/example.com/html
```

#### Copiamos todos los archivos de la carpeta dist `solo archivos` a nuestra dirección  creada previamente y la renombramos como html: ####

```shell
[root@server]# cp -R dist /var/www/var/www/example.com/html
```

#### Otorgamos permisos `en $USER colocar el nombre del usuario actual ejemplo root:root en $USER colocar el nombre del usuario actual ejemplo root:root`: ####

```shell
[root@server]# sudo chown -R $USER:$USER /var/www/your_domain/html
```

#### El siguiente comando permitirá presentar el root de su documento personalizado como contenido HTTP: ####

```shell
[root@server]# chcon -vR system_u:object_r:httpd_sys_content_t:s0 /var/www/your_domain/
```
<br>

**3. Instalación de NGINX en Centos:**
#### Ejecutar los siguientes comandos necesarios para el funcionamiento e instalación correcta de nginx: ####

```shell
[root@server]# sudo yum install epel-release
```

```shell
[root@server]# sudo yum install nginx
```

#### Habilitamos permanentemente las conexiones `http` y `https`: ####

```shell
[root@server]# sudo firewall-cmd --permanent --add-service=http
```

```shell
[root@server]# sudo firewall-cmd --permanent --add-service=https
```

#### Reiniciamos el Firewall ####

```shell
[root@server]# sudo firewall-cmd –-reload
```

#### Usar el editor de su preferencia para edición de archivos en el Shell en este caso se usará vim: ####

- Nos dirigimos a la carpeta `conf.d` en la siguiente ruta

```shell
[root@server]# cd /etc/nginx/conf.d
```

#### Crear un archivo con extencion `.conf` ejemplo: ####

```shell
[root@server]# vim example.com.conf
```

#### Dentro del archivo configuramos lo siguiente cambiar los datos de example por el dominio y subdominio de su aplicación. ####

```shell
# Frontend
server{
	listen 80;
       server_name example.com  www.example.com;
       location / {
                root /var/www/example.com/html;
                index index.html index.htm index.nginx-debian.html;
                try_files $uri$args $uri$args/ /index.html;
       }
}

# Backend
server {
	 listen: 80;
       server_name api.example.com  www.api.example.com;

       location / {
                proxy_pass http://localhost:3001;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
       }
}

```

#### Guardamos los cambios y ejecutamos los siguientes comandos: ####

```shell
[root@server]# sudo systemctl start nginx
```

```shell
[root@server]# sudo systemctl enable nginx
```

#### Comprobamos el estado de Nginx ####

```shell
[root@server]# sudo systemctl status nginx
```

#### Activamos los siguientes valores booleanos: ####

```shell 
[root@server]# sudo setsebool -P httpd_can_network_relay on
```

```shell
[root@server]# sudo setsebool -P httpd_can_network_connect on
```

#### Reiniciamos Nginx ####

```shell
[root@server]# sudo systemctl restart nginx
```
<br>

**4. Instalación de NGINX en Centos:**

#### Para tener los certificados ssl en nuestros dominios y subdominios ejecutamos los siguientes comandos: ####

```shell
[root@server]# sudo dnf install certbot
```

```shell
[root@server]# sudo dnf install certbot python3-certbot-nginx
```

#### Añadimos los certificados a los dominios y subdominios remplazar `example.com` por su dominio: ####

```shell
[root@server]# sudo certbot --nginx -d example.com -d www.example.com -d api.example.com -d www.api.example.com
```

#### Para renovar automáticamente el certificado copiar y pegar el siguiente comando: ####

```shell
[root@server]# echo "0 0,12 * * * root python3 -c 'import random; import time; time.sleep(random.random() * 3600)' && certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
```

-  No solicitara un correo lo ingresamos  y Enter.
- Aceptamos los términos con A y Enter.
- `No` agregamos nuestro correo a la fundación Electronic Frontier colocamos N y Enter.
- Esperamos la validación y ya estaría configurados los SSL en tus dominios y subdominio.