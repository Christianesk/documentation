
# Installing MongoDB on Production Server

#### <b style="color:red">Note:</b> The configuration was performed using Droplets, the Digital Ocean server provider with MongoDB version 4.4 and a Centos RHEL 8 version.

<br>
<br>

# Installation and configuration

**1. Create a file in the following directory to install MongoDB directly using yum.**

```shell
[root@server]# vim /etc/yum.repos.d/mongodb-org-4.4.repo
```

**2. Inside the file we place the following:**

```shell
[mongodb-org-4.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.4.asc
```
`Save the changes to the file.`

**3. Update YUM**

```shell
[root@server]# yum update
```

**4. Install MongoDB**
```shell
[root@server]# sudo yum install -y mongodb-org
```

**5. We label the port we are going to use for mongo, `it is recommended not to use` the default port for security reasons for this example we will use port 27064.**

```shell
[root@server]# sudo semanage port -a -t mongod_port_t -p tcp 27064
```

**6. Add permissions to the firewall on the port to be used:**
```shell
[root@server]# sudo firewall-cmd --add-port=27064/tcp --permanent 
```
Reload the firewall service

```shell
[root@server]# sudo firewall-cmd --reload
```

**7. Access can also be limited according to the source ip address:**

The `ip_address` is the ip address from where you want to access the database.
```shell
[root@server]# sudo firewall-cmd --permanent --add-rich-rule "rule family="ipv4" \
source address="ip_address" port protocol="tcp" port="27064" accept"
```

**8. Enable access outside the server:**

Edit the following file:

```shell
[root@server]# vim /etc/mongod.conf  
```

Inside the file we add the ip address of the server where we are installing Mongo.

Replace `IP_SERVER` with our server ip address.

The file should look as follows:

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

**9. Reload deamon**

```shell
[root@server]# systemctl daemon-reload
```

**10. Initialize and enable mongo database:**

```shell
[root@server]# start mongod
```

```shell
[root@server]# enable  mongod
```
<br>
<br>

# Creating users in MongoDB

**1. Creating `Global User` with ip authentication restrictions**

Log in to Mongo with the port we assigned previously

```shell
[root@server]# mongo --port 27064 
```

**2. Log in to the admin database to create the user:**

```shell
> use admin
```

<b style="color:red">Note:</b> The following operation creates a user. This user can only be authenticated by connecting from an IP address `XXX.X.X.X.X` to another `ZZZ.ZZ.ZZ.ZZZ.Z`.

```shell
> db.createUser({user: "your_global_user", pwd: "your_password", roles: [{role: "root", db: "admin"}],authenticationRestrictions:[{clientSource:[ "XXX.X.X.X","add_other_ip" ],serverAddress:[ "ZZZ.ZZ.ZZZ.Z" ]}]})
```

**3. Create a `user` for a specific database for security purposes.**

```shell
> use nueva_base
```

```shell
> db.createUser({user: "your_user", pwd: "your_password", roles: [{role: "readWrite", db: "nueva_base"}],authenticationRestrictions:[{clientSource:[ " XXX.X.X.X ","add_other_ip" ],serverAddress:[ " ZZZ.ZZ.ZZZ.Z" ]}]})
```

**4. Edit again the `mongod.conf` file**

```shell
[root@server]# vim /etc/mongod.conf 
```

**5. Add to the end of the file**

```shell
security:
  authorization: enabled
```

**6. The `mongod.conf` file should look like this:**

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

**7. Restart Mongo**

```shell
[root@server]# systemctl restart mongod
```
<br>
<br>

# Added Security in Digital Ocean Droplet

**1. Create firewall rules as follows:**

Enter the Droplet and click on Networking:

![Image of config firewall in droplet](https://firebasestorage.googleapis.com/v0/b/documentation-github.appspot.com/o/img-1.png?alt=media&token=1539000a-25b3-4b39-a8c0-288ea1939f8d)

We have a Firewalls section, click on create firewall rules and enter the rules you need for your server, in this case the basic ones were added and in the blanks of the image should be the IP allowed to access your server and as such to the database through the previously configured port:

![Image of config firewall in droplet](https://firebasestorage.googleapis.com/v0/b/documentation-github.appspot.com/o/img-2.png?alt=media&token=1e95d2e4-afc8-42d4-bea8-a16480c3183f)




