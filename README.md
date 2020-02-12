# Bugzilla web server
Docker image for bugzilla web server.

Container runs as non-root user.
This user owns bugzilla process and owns bugzilla files.

To run container next volumes should be mapped:
* folder for SQL database
* folder for bugzilla files
* logs folder
* backups folder

## Installation
### Installation from docker image
1. Pull docker image.
2. Create user and group to own bugzilla files and to run docker container:
    ```
    sudo groupadd -g 967 bugzilla
    ```
    ```
    useradd -u 967 -g 967 -M bugzilla
    ```
3. Proceed to configuration.

### Installation from source
1. Pull project sources from version control system.
2. Create user and group to own bugzilla files and to run docker container:
    ```
    sudo useradd -r bugzilla
    ```
3. Make **build** executable:
    ```
    sudo chmod u+x ./build
    ```
4. Execute **build**:
    ```
    sudo ./build bugzilla
    ```
5. Proceed to configuration.

### Configuration
1. Create folders for bugzilla files:
    ```
    sudo mkdir /bugzilla
    ```
    ```
    sudo mkdir /bugzilla/db
    ```
    ```
    sudo mkdir /bugzilla/data
    ```
2. Create folder for logs:
    ```
    sudo mkdir /var/log/bugzilla
    ```
3. Create folder for backups:
    ```
    sudo mkdir /var/backups/bugzilla
    ```
4. Grant permit to all folders:
    ```
    sudo chown -R bugzilla:bugzilla /bugzilla
    ```
    ```
    sudo chown bugzilla:bugzilla /var/log/bugzilla
    ```
    ```
    sudo chown bugzilla:bugzilla /var/backups/bugzilla
    ```
5. Copy **etc/init.d/bugzilla** to **/etc/init.d** folder:
    ```
    sudo cp ./etc/init.d/bugzilla /etc/init.d
    ```
6. Copy **usr/sbin/bugzilla** to **/usr/sbin** folder:
    ```
    sudo cp ./usr/sbin/bugzilla /usr/sbin
    ```
7. Copy **usr/bin/bgutil** to **/usr/bin** folder:
    ```
    sudo cp ./usr/bin/bgutil /usr/bin
    ```
8. Make all files executable:
    ```
    sudo chmod a+x /etc/init.d/bugzilla
    ```
    ```
    sudo chmod a+x /usr/sbin/bugzilla
    ```
    ```
    sudo chmod a+x /usr/bin/bgutil
    ```
9. Register service:
    ```
    sudo update-rc.d bugzilla defaults
    ```
10. Specify database root password in **/usr/sbin/bugzilla** file:
    ```
    docker run ... -e DB_ROOT_PASSWORD="<some_password>" ...
    ```
11. Specify bugzilla database user password in **/usr/sbin/bugzilla** file:
    ```
    docker run ... -e DB_USER_PASSWORD="<some_password>" ...  
    ```
12. Start bugzilla service:
    ```
    sudo service bugzilla start
    ```
13. Initialize bugzilla database:
    ```
    sudo bgutil initialize
    ```
    Specify the following information:
    * administrator e-mail
    * administrator name
    * administrator password
    * administrator password confirmation
14. Restart bugzilla service:
    ```
    sudo service bugzilla restart
    ```

## Management
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

## Apache mod_proxy configuration
Bugzilla web server can be located with another web applications.
For example, mercurial, bugzilla, wiki etc can be run as docker containers on the same host.
In this case apache server can be used to redirect requests to different docker containers.

1. Enable apache mod_proxy:
    ```
    sudo a2enmod deflate headers proxy proxy_ajp proxy_balancer proxy_connect proxy_html proxy_http rewrite
    ```
2. Configure proxy:
    ```
    <VirtualHost *:80>
        ...
        ProxyPreserveHost On
        <Proxy *>
            Order allow,deny
            Allow from all
        </Proxy>
        ...
    </VirtualHost>
    ```
3. Copy **./etc/apache2/sites-available/bugzilla.conf** to **/etc/apache2/sites-available** folder:
    ```
    sudo cp ./etc/apache2/sites-available/bugzilla.conf /etc/apache2/sites-available
    ```
4. Enable apache sites:
    ```
    sudo a2ensite bugzilla
    ```
5. Restart apache service:
    ```
    sudo service apache2 restart
    ```

## HOW TO
### How to change database root password
1. Stop bugzilla service:
    ```
    sudo service bugzilla stop
    ```
2. Specify new database root password in **/usr/sbin/bugzilla** file:
    ```
    docker run ... -e DB_ROOT_PASSWORD="<new_password>" ...
    ```
3. Start bugzilla service:
    ```
    sudo service bugzilla start
    ```
4. Run the following command:
    ```
    sudo bgutil changeRootPassword "<old_password>"
    ```

### How to change bugzilla database user password
1. Stop bugzilla service:
    ```
    sudo service bugzilla stop
    ```
2. Specify new bugzilla database user password in **/usr/sbin/bugzilla** file:
    ```
    docker run ... -e DB_USER_PASSWORD="<new_password>" ...  
    ```
3. Start bugzilla service:
    ```
    sudo service bugzilla start
    ```
4. Run the following command
    ```
    sudo bgutil changeUserPassword
    ```

### How to specify special characters in password
Special characters should be escaped:
```
docker run ... -e DB_ROOT_PASSWORD="pa\$\$word" ...
```
```
docker run ... -e DB_USER_PASSWORD="pas\$11" ...  
```

### How to create cron job for backups
```
sudo crontab -l | { cat; echo "minute hour * * * /usr/bin/bgutil backup <filename>"; echo ""; } | sudo crontab -
```

# Donation
If you find my code useful, you can [bye me a coffee](https://www.paypal.me/dshapovalov)
