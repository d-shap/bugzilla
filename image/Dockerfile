FROM        debian:jessie
MAINTAINER  Dmitry Shapovalov

ARG         hostuid=986
ARG         hostgid=986
ARG         guestport=8008
ARG         guestappname=bugzilla
ARG         downloadname=bugzilla-5.0.3

ENV         DEBIAN_FRONTEND noninteractive
ENV         APACHE_RUN_USER bugzilla
ENV         APACHE_RUN_GROUP bugzilla
ENV         GUEST_APP_NAME $guestappname

COPY        . /tmp

# Install packages
RUN         apt-get update && \
            apt-get install --assume-yes \
                apache2 \
                apache2-mpm-prefork \
                build-essential \
                graphviz \
                libapache2-mod-perl2 \
                libapache2-mod-perl2-dev \
                libappconfig-perl \
                libauthen-radius-perl \
                libauthen-sasl-perl \
                libcgi-pm-perl \
                libchart-perl \
                libdate-calc-perl \
                libdatetime-perl \
                libdatetime-timezone-perl \
                libdbd-mysql-perl \
                libdbi-perl \
                libemail-mime-modifier-perl \
                libemail-mime-perl \
                libemail-sender-perl \
                libencode-detect-perl \
                libfile-mimeinfo-perl \
                libfile-slurp-perl \
                libfile-which-perl \
                libgd-dev \
                libgd-graph-perl \
                libhtml-formattext-withlinks-perl \
                libhtml-scrubber-perl \
                libjson-rpc-perl \
                libmath-random-isaac-perl \
                libmath-random-isaac-xs-perl \
                libmime-perl \
                libmodule-build-perl \
                libmysqlclient-dev \
                libnet-ldap-perl \
                libsoap-lite-perl \
                libtemplate-perl \
                libtemplate-perl-doc \
                libtemplate-plugin-gd-perl \
                libtest-taint-perl \
                libtheschwartz-perl \
                libxml-perl \
                libxml-twig-perl \
                lynx-cur \
                mysql-server \
                perlmagick \
                python-sphinx \
                rst2pdf \
                wget && \
            apt-get clean

# Create user and group
RUN         groupadd -g $hostgid bugzilla && \
            useradd -u $hostuid -g $hostgid -M bugzilla

# Create folders for bugzilla
RUN         mkdir /bugzilla && \
            mkdir /bugzilla/backups && \
            mkdir /bugzilla/logs && \
            chown -R bugzilla:bugzilla /bugzilla

# Configure MySQL
RUN         sed -i '\|\[mysqld\]|a ft_min_word_len = 2' /etc/mysql/my.cnf && \
            sed -i 's|user.*|user            = bugzilla|' /etc/mysql/my.cnf && \
            sed -i 's|datadir.*|datadir         = /bugzilla/db|' /etc/mysql/my.cnf && \
            sed -i 's|key_buffer.*|key_buffer_size         = 16M|' /etc/mysql/my.cnf && \
            sed -i 's|max_allowed_packet.*|max_allowed_packet      = 100M|' /etc/mysql/my.cnf && \
            sed -i 's|myisam-recover.*|myisam-recover-options  = BACKUP|' /etc/mysql/my.cnf && \
            sed -i 's|log_error.*|log_error                = /var/log/mysql/dberror.log|' /etc/mysql/my.cnf && \
            sed -i '\|\[mysqld_safe\]|a log_error       = /var/log/mysql/dberror.log' /etc/mysql/my.cnf && \
            sed -i 's|syslog.*|# syslog|' /etc/mysql/conf.d/mysqld_safe_syslog.cnf && \
            rm -rf /var/lib/mysql && \
            rm -rf /var/log/mysql && \
            ln -s /bugzilla/logs /var/log/mysql && \
            chown -R bugzilla:bugzilla /var/log/mysql && \
            chown -R bugzilla:bugzilla /var/run/mysqld

# Configure apache
RUN         sed -i 's|:80>|:'"$guestport"'>|' /etc/apache2/sites-available/000-default.conf && \
            sed -i 's|#ServerName www.example.com|ServerName localhost|' /etc/apache2/sites-available/000-default.conf && \
            mv /tmp/bugzilla.conf /etc/apache2/sites-available && \
            sed -i 's|/guestappname|/'"$guestappname"'|' /etc/apache2/sites-available/bugzilla.conf && \
            sed -i 's|Listen 80|Listen '"$guestport"'|' /etc/apache2/ports.conf && \
            echo "ServerName localhost" > /etc/apache2/conf-available/servername.conf && \
            a2enconf servername && \
            a2enmod cgi expires headers rewrite && \
            a2ensite bugzilla && \
            rm -rf /var/log/apache2 && \
            ln -s /bugzilla/logs /var/log/apache2 && \
            chown -R bugzilla:bugzilla /var/log/apache2 && \
            chown -R bugzilla:bugzilla /var/lock/apache2 && \
            chown -R bugzilla:bugzilla /var/run/apache2

# Configure bugzilla
RUN         cd ~ && \
            wget https://ftp.mozilla.org/pub/mozilla.org/webtools/${downloadname}.tar.gz && \
            tar xzf ${downloadname}.tar.gz -C /var/www/html && \
            rm ${downloadname}.tar.gz && \
            mv /var/www/html/${downloadname} /var/www/html/${guestappname} && \
            rm -rf /var/www/html/${guestappname}/data && \
            ln -s /bugzilla/data /var/www/html/${guestappname}/data && \
            chown -R bugzilla:bugzilla /var/www/html/${guestappname} && \
            cd /var/www/html/${guestappname} && \
            /usr/bin/perl install-module.pl --all && \
            /var/www/html/${guestappname}/checksetup.pl && \
            sed -i "s|\$webservergroup.*|\$webservergroup = 'bugzilla';|" /var/www/html/${guestappname}/localconfig && \
            sed -i "s|\$db_port.*|\$db_port = 3306;|" /var/www/html/${guestappname}/localconfig && \
            sed -i "s|\$db_sock.*|\$db_sock = '/var/run/mysqld/mysqld.sock';|" /var/www/html/${guestappname}/localconfig

# Configure helper scripts
RUN         mv /tmp/setupDatabaseRootPassword /bugzilla && \
            chmod a+x /bugzilla/setupDatabaseRootPassword && \
            mv /tmp/changeDatabaseRootPassword /bugzilla && \
            chmod a+x /bugzilla/changeDatabaseRootPassword && \
            mv /tmp/setupDatabaseUserPassword /bugzilla && \
            chmod a+x /bugzilla/setupDatabaseUserPassword && \
            mv /tmp/changeDatabaseUserPassword /bugzilla && \
            chmod a+x /bugzilla/changeDatabaseUserPassword && \
            mv /tmp/createDatabase /bugzilla && \
            chmod a+x /bugzilla/createDatabase && \
            mv /tmp/backupDatabase /bugzilla && \
            chmod a+x /bugzilla/backupDatabase && \
            mv /tmp/restoreDatabase /bugzilla && \
            chmod a+x /bugzilla/restoreDatabase && \
            mv /tmp/startup /bugzilla && \
            chmod a+x /bugzilla/startup && \
            mv /tmp/shutdown /bugzilla && \
            chmod a+x /bugzilla/shutdown

USER        bugzilla

EXPOSE      $guestport

ENTRYPOINT  ["/bugzilla/startup"]
