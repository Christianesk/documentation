# Publish an APP using Nginx on centos 8 (Host DigitalOcean)

### The App to be published uses the following technologies:

<br>


**Backend:** Framework NestJS (NodeJs)

**Frontend:** Angular 9
<br>
<br>

# Configuration

**1. Prerequisites**

- Have the backend running and have done the ng build -prod to get the dist folder.
- Have domain and subdomains previously created example:


![Image of config dns](https://firebasestorage.googleapis.com/v0/b/documentation-github.appspot.com/o/img-3.png?alt=media&token=70cf5cfb-4d55-4fd6-b8fb-f9a12a9c47d3)

**2. Frontend folder creation**
#### We create the folder where we will have to place the files of the frontend dist folder change `example.com` for your domain: ####


```shell
[root@server]# sudo mkdir -p /var/www/example.com/html
```

#### We copy all the files from the dist folder (files only) to our previously created address and rename it as html: ####

```shell
[root@server]# cp -R dist /var/www/var/www/example.com/html
```

#### We copy all the files from the dist `files only` folder to our previously created address and rename it as html: ####

```shell
[root@server]# sudo chown -R $USER:$USER /var/www/your_domain/html
```

#### The following command will allow you to present the root of your custom document as HTTP content: ####

```shell
[root@server]# chcon -vR system_u:object_r:httpd_sys_content_t:s0 /var/www/your_domain/
```
<br>

**3. Installation of NGINX on Centos:**
#### Execute the following commands necessary for the correct operation and installation of nginx: ####

```shell
[root@server]# sudo yum install epel-release
```

```shell
[root@server]# sudo yum install nginx
```

#### We permanently enable `http` and `https` connections: ####

```shell
[root@server]# sudo firewall-cmd --permanent --add-service=http
```

```shell
[root@server]# sudo firewall-cmd --permanent --add-service=https
```

#### Restart the Firewall ####

```shell
[root@server]# sudo firewall-cmd –-reload
```

#### Use the editor of your choice for editing files in the Shell in this case vim will be used: ####

- Nos dirigimos a la carpeta `conf.d` en la siguiente ruta

```shell
[root@server]# cd /etc/nginx/conf.d
```

#### Create a file with extension `.conf` example: ####

```shell
[root@server]# vim example.com.conf
```

#### Inside the file we configure the following change the example data for the domain and subdomain of your application. ####

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

#### Save the changes and execute the following commands: ####

```shell
[root@server]# sudo systemctl start nginx
```

```shell
[root@server]# sudo systemctl enable nginx
```

#### We check the status of Nginx ####

```shell
[root@server]# sudo systemctl status nginx
```

#### We activate the following boolean values: ####

```shell 
[root@server]# sudo setsebool -P httpd_can_network_relay on
```

```shell
[root@server]# sudo setsebool -P httpd_can_network_connect on
```

#### Restart Nginx ####

```shell
[root@server]# sudo systemctl restart nginx
```
<br>

**4. 4.	SSL Certificates with LetsEncrypt**

#### To have the ssl certificates in our domains and subdomains we execute the following commands: ####

```shell
[root@server]# sudo dnf install certbot
```

```shell
[root@server]# sudo dnf install certbot python3-certbot-nginx
```

#### We add certificates to domains and subdomains to replace example.com with your domain: ####

```shell
[root@server]# sudo certbot --nginx -d example.com -d www.example.com -d api.example.com -d www.api.example.com
```

#### To automatically renew the certificate, copy and paste the following command ####

```shell
[root@server]# echo "0 0,12 * * * root python3 -c 'import random; import time; time.sleep(random.random() * 3600)' && certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
```

-  It will not request an email we enter it and Enter.
- We accept the terms with A and Enter
- We `don't` add our mail to the Electronic Frontier foundation we put N and Enter.
- We wait for the validation and the SSL would already be configured in your domains and subdomain.