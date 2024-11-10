FROM        debian:buster
MAINTAINER  Dmitry Shapovalov

ARG         hostuid=967
ARG         hostgid=967

ENV         DEBIAN_FRONTEND=noninteractive \
            RUN_USER="bugzilla" \
            RUN_GROUP="bugzilla" \
            RUN_PORT=8007 \
            APACHE_RUN_USER="bugzilla" \
            APACHE_RUN_GROUP="bugzilla" \
            DB_NAME="bugs" \
            DB_USER_NAME="bugs" \
            BUGZILLA_FOLDER_NAME="bugzilla-5.2"

COPY        . /tmp

# Install common packages and configure environment
RUN         apt-get update && \
            apt-get upgrade --assume-yes && \
            apt-get install --assume-yes \
                apt-utils \
                build-essential \
                cron \
                curl \
                net-tools \
                procps \
                software-properties-common \
                vim \
                wget && \
            apt-get update && \
            apt-get install --assume-yes \
                graphviz \
                libgd-dev \
                lynx \
                patchutils \
                python-sphinx \
                rst2pdf \
                sendmail \
                sendmail-base \
                sendmail-bin && \
            apt-get clean

# Create user and group
RUN         groupadd -g $hostgid $RUN_GROUP && \
            useradd -u $hostuid -g $hostgid -M $RUN_USER

# Configure application folders
RUN         mkdir /bugzilla && \
            mkdir /bugzilla/db && \
            mkdir /bugzilla/data && \
            mkdir /bugzilla/scripts && \
            mkdir /bugzilla/logs && \
            mkdir /bugzilla/backups && \
            chown -R $RUN_USER:$RUN_GROUP /bugzilla

# Install Perl
RUN         apt-get update && \
            apt-get install --assume-yes \
                libappconfig-perl \
                libauthen-radius-perl \
                libauthen-sasl-perl \
                libcgi-pm-perl \
                libcache-memcached-perl \
                libchart-perl \
                libconfig-yaml-perl \
                libdaemon-generic-perl \
                libdate-calc-perl \
                libdatetime-perl \
                libdatetime-timezone-perl \
                libdbd-mysql-perl \
                libdbd-pg-perl \
                libdbi-perl \
                libemail-address-perl \
                libemail-mime-modifier-perl \
                libemail-mime-perl \
                libemail-reply-perl \
                libemail-sender-perl \
                libencode-detect-perl \
                libfile-copy-recursive-perl \
                libfile-mimeinfo-perl \
                libfile-slurp-perl \
                libfile-tail-perl \
                libfile-which-perl \
                libgd-graph-perl \
                libhtml-formattext-withlinks-perl \
                libhtml-scrubber-perl \
                libjson-rpc-perl \
                libmath-random-isaac-perl \
                libmath-random-isaac-xs-perl \
                libmime-tools-perl \
                libmodule-build-perl \
                libmodule-reader-perl \
                libnet-ldap-perl \
                libsoap-lite-perl \
                libtemplate-perl \
                libtemplate-plugin-gd-perl \
                libtest-taint-perl \
                libtext-patch-perl \
                libtheschwartz-perl \
                libxml-perl \
                libxml-twig-perl \
                perlmagick && \
            apt-get clean

# Install MySQL
RUN         apt-get update && \
            apt-get install --assume-yes \
                default-libmysqlclient-dev \
                default-mysql-client \
                default-mysql-server && \
            apt-get clean && \
            sed -i '\|\[mysqld\]|a ft_min_word_len = 2' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i 's|user.*|user            = '"$RUN_USER"'|' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i 's|datadir.*|datadir         = /bugzilla/db|' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i 's|key_buffer.*|key_buffer_size         = 16M|' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i 's|max_allowed_packet.*|max_allowed_packet      = 8M|' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i 's|myisam-recover.*|myisam-recover-options  = BACKUP|' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i 's|log_error.*|log_error                = /var/log/mysql/dberror.log|' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i 's|syslog.*|# syslog|' /etc/mysql/mariadb.conf.d/50-mysqld_safe.cnf && \
            rm -rf /var/lib/mysql && \
            rm -rf /var/log/mysql && \
            ln -s /bugzilla/logs /var/log/mysql && \
            chown -h $RUN_USER:$RUN_GROUP /var/log/mysql && \
            mkdir -p /var/run/mysqld && \
            chown -R $RUN_USER:$RUN_GROUP /var/run/mysqld

# Install Apache
RUN         apt-get update && \
            apt-get install --assume-yes \
                apache2 \
                libapache2-mod-perl2 \
                libapache2-mod-perl2-dev && \
            apt-get clean && \
            sed -i 's|:80>|:'"$RUN_PORT"'>|' /etc/apache2/sites-available/000-default.conf && \
            sed -i 's|#ServerName www.example.com|ServerName localhost|' /etc/apache2/sites-available/000-default.conf && \
            mv /tmp/bugzilla.conf /etc/apache2/sites-available && \
            sed -i 's|Listen 80|Listen '"$RUN_PORT"'|' /etc/apache2/ports.conf && \
            echo "ServerName localhost" > /etc/apache2/conf-available/servername.conf && \
            a2enconf servername && \
            a2enmod cgi expires headers rewrite && \
            a2ensite bugzilla && \
            rm -rf /var/log/apache2 && \
            ln -s /bugzilla/logs /var/log/apache2 && \
            chown -h $RUN_USER:$RUN_GROUP /var/log/apache2 && \
            mkdir -p /var/lock/apache2 && \
            chown -R $RUN_USER:$RUN_GROUP /var/lock/apache2 && \
            mkdir -p /var/run/apache2 && \
            chown -R $RUN_USER:$RUN_GROUP /var/run/apache2

# Configure bugzilla
RUN         tar xzf /tmp/bin/$BUGZILLA_FOLDER_NAME.tar.gz -C /var/www/html && \
            rm /tmp/bin/$BUGZILLA_FOLDER_NAME.tar.gz && \
            mv /var/www/html/$BUGZILLA_FOLDER_NAME /var/www/html/bugzilla && \
            rm -rf /var/www/html/bugzilla/data && \
            ln -s /bugzilla/data /var/www/html/bugzilla/data && \
            cd /var/www/html/bugzilla && \
            /usr/bin/perl install-module.pl --all && \
            /usr/bin/perl install-module.pl PatchReader && \
            /usr/bin/perl checksetup.pl && \
            sed -i "s|\$webservergroup.*|\$webservergroup = '"$RUN_GROUP"';|" /var/www/html/bugzilla/localconfig && \
            sed -i "s|\$db_name.*|\$db_name = '"$DB_NAME"';|" /var/www/html/bugzilla/localconfig && \
            sed -i "s|\$db_user.*|\$db_user = '"$DB_USER_NAME"';|" /var/www/html/bugzilla/localconfig && \
            sed -i "s|\$db_driver.*|\$db_driver = 'mariadb';|" /var/www/html/bugzilla/localconfig && \
            sed -i "s|\$db_port.*|\$db_port = 3306;|" /var/www/html/bugzilla/localconfig && \
            sed -i "s|\$db_sock.*|\$db_sock = '/var/run/mysqld/mysqld.sock';|" /var/www/html/bugzilla/localconfig

# Final configurations and clean-ups
RUN         rm -rf /tmp/bin && \
            chown -R -h $RUN_USER:$RUN_GROUP /var/www/html/bugzilla

# Configure helper scripts
RUN         mv /tmp/containerConfig /bugzilla/scripts && \
            mv /tmp/containerDummy /bugzilla/scripts && \
            mv /tmp/containerStartup /bugzilla/scripts && \
            mv /tmp/containerShutdown /bugzilla/scripts && \
            mv /tmp/webStartup /bugzilla/scripts && \
            mv /tmp/webShutdown /bugzilla/scripts && \
            mv /tmp/utilInitialize /bugzilla/scripts && \
            mv /tmp/utilChangeRootPassword /bugzilla/scripts && \
            mv /tmp/utilChangeUserPassword /bugzilla/scripts && \
            mv /tmp/utilBackup /bugzilla/scripts && \
            mv /tmp/utilRestore /bugzilla/scripts && \
            chmod 755 /bugzilla/scripts/*

USER        $RUN_USER

EXPOSE      $RUN_PORT

ENTRYPOINT  ["/bugzilla/scripts/containerStartup"]
