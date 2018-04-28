Bugzilla web server
===================
Docker image for bugzilla web server.

Container runs as non-root user.
This user owns bugzilla process and owns bugzilla database.

To run container next volumes should be mapped
* folder for SQL database
* folder for bugzilla data, such as categories and local stored attachments (Big Files)
* log folder
* backup folder

Installation
------------
Create user and group to own bugzilla files and to run docker container:
```
sudo useradd -r bugzilla
```

Make **build** executable:
```
sudo chmod u+x ./build
```

Execute **build**:
```
sudo ./build bugzilla
```

Create folders for bugzilla database:
```
sudo mkdir /bugzilla
```
```
sudo mkdir /bugzilla/db
```
```
sudo mkdir /bugzilla/data
```

Create folder for logs:
```
sudo mkdir /var/log/bugzilla
```

Create folder for backups:
```
sudo mkdir /var/backups/bugzilla
```

Grant permit to all folders:
```
sudo chown -R bugzilla:bugzilla /bugzilla
```
```
sudo chown bugzilla:bugzilla /var/log/bugzilla
```
```
sudo chown bugzilla:bugzilla /var/backups/bugzilla
```

Copy **etc/init.d/bugzilla** to **/etc/init.d** folder:
```
sudo cp ./etc/init.d/bugzilla /etc/init.d
```

Copy **usr/sbin/bugzilla** to **/usr/sbin** folder:
```
sudo cp ./usr/sbin/bugzilla /usr/sbin
```

Copy **usr/bin/bgutil** to **/usr/bin** folder:
```
sudo cp ./usr/bin/bgutil /usr/bin
```

Make all files executable:
```
sudo chmod a+x /etc/init.d/bugzilla
```
```
sudo chmod a+x /usr/sbin/bugzilla
```
```
sudo chmod a+x /usr/bin/bgutil
```

Register service:
```
sudo update-rc.d bugzilla defaults
```

Specify database root password in **/usr/sbin/bugzilla** file:
```
docker run ... -e DB_ROOT_PASSWORD="some_password" ...
```

Specify bugzilla database user password in **/usr/sbin/bugzilla** file:
```
docker run ... -e DB_USER_PASSWORD="some_password" ...  
```

Start bugzilla service:
```
sudo service bugzilla start
```

Initialize bugzilla database:
```
sudo bgutil initialize
```

Specify the following information:
* administrator e-mail
* administrator name
* administrator password

Management
----------
### Service management
```
sudo service bugzilla (start|stop|status|restart)
```

### Create backup
```
sudo bgutil backup <filename>
```

Backup file **/var/backups/bugzilla/&lt;filename&gt;.tar.gz** will be created.

### Restore backup
```
sudo bgutil restore <filename>
```

### Command line (bash)
```
sudo bgutil bash
```

Apache mod_proxy configuration
------------------------------
Bugzilla web server can be located with another web applications.
For example, mercurial, bugzilla, wiki etc can be run as docker containers on the same host.
In this case apache server can be used to redirect requests to different docker containers.

First, mod_proxy should be enabled:
```
sudo a2enmod proxy proxy_ajp proxy_http rewrite deflate headers proxy_balancer proxy_connect proxy_html
```

Then configure proxy:
```
<VirtualHost *:80>

...

ProxyPreserveHost On
<Proxy *>
    Order allow,deny
    Allow from all
</Proxy>

...

ProxyPass /bugzilla http://localhost:8008/bugzilla
ProxyPassReverse /bugzilla http://localhost:8008/bugzilla

...

</VirtualHost>
```

Finally, restart apache service:
```
sudo service apache2 restart
```

HOW TO
------
### How to change database root password
Stop bugzilla service:
```
sudo service bugzilla stop
```

Specify new database root password in **/usr/sbin/bugzilla** file:
```
docker run ... -e DB_ROOT_PASSWORD="new_password" ...
```

Start bugzilla service:
```
sudo service bugzilla start
```

Run the following command:
```
sudo bgutil changeRootPassword "<old_password>"
```

### How to change bugzilla database user password
Stop bugzilla service:
```
sudo service bugzilla stop
```

Specify new bugzilla database user password in **/usr/sbin/bugzilla** file:
```
docker run ... -e DB_USER_PASSWORD="new_password" ...  
```

Start bugzilla service:
```
sudo service bugzilla start
```

Run the following command
```
sudo bgutil changeUserPassword
```

### How to create cron job for backups
```
sudo crontab -l | { cat; echo "minute hour * * * /usr/bin/bgutil backup <filename>"; echo ""; } | sudo crontab -
```
