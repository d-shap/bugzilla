FROM        debian:jessie
MAINTAINER  Dmitry Shapovalov

ARG         hostuid=967
ARG         hostgid=967

ENV         DEBIAN_FRONTEND noninteractive
ENV         RUN_USER bugzilla
ENV         RUN_GROUP bugzilla
ENV         RUN_PORT 8007
ENV         DB_NAME bugs
ENV         DB_USER_NAME bugs
ENV         APACHE_RUN_USER $RUN_USER
ENV         APACHE_RUN_GROUP $RUN_GROUP
ENV         BUGZILLA_DOWNLOAD_VERSION 5.0.4
ENV         BUGZILLA_DOWNLOAD_NAME bugzilla-$BUGZILLA_DOWNLOAD_VERSION
ENV         BUGZILLA_DOWNLOAD_URL https://ftp.mozilla.org/pub/mozilla.org/webtools/$BUGZILLA_DOWNLOAD_NAME.tar.gz

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
                sendmail \
                vim \
                wget && \
            apt-get clean

# Create user and group
RUN         groupadd -g $hostgid $RUN_GROUP && \
            useradd -u $hostuid -g $hostgid -M $RUN_USER

# Create folders for bugzilla
RUN         mkdir /bugzilla && \
            mkdir /bugzilla/db && \
            mkdir /bugzilla/data && \
            mkdir /bugzilla/scripts && \
            mkdir /bugzilla/logs && \
            mkdir /bugzilla/backups && \
            chown -R $RUN_USER:$RUN_GROUP /bugzilla

# Configure MySQL
RUN         sed -i '\|\[mysqld\]|a ft_min_word_len = 2' /etc/mysql/my.cnf && \
            sed -i 's|user.*|user            = '"$RUN_USER"'|' /etc/mysql/my.cnf && \
            sed -i 's|datadir.*|datadir         = /bugzilla/db|' /etc/mysql/my.cnf && \
            sed -i 's|key_buffer.*|key_buffer_size         = 16M|' /etc/mysql/my.cnf && \
            sed -i 's|max_allowed_packet.*|max_allowed_packet      = 8M|' /etc/mysql/my.cnf && \
            sed -i 's|myisam-recover.*|myisam-recover-options  = BACKUP|' /etc/mysql/my.cnf && \
            sed -i 's|log_error.*|log_error                = /var/log/mysql/dberror.log|' /etc/mysql/my.cnf && \
            sed -i '\|\[mysqld_safe\]|a log_error       = /var/log/mysql/dberror.log' /etc/mysql/my.cnf && \
            sed -i 's|syslog.*|# syslog|' /etc/mysql/conf.d/mysqld_safe_syslog.cnf && \
            rm -rf /var/lib/mysql && \
            rm -rf /var/log/mysql && \
            ln -s /bugzilla/logs /var/log/mysql && \
            chown -h $RUN_USER:$RUN_GROUP /var/log/mysql && \
            mkdir -p /var/run/mysqld && \
            chown -R $RUN_USER:$RUN_GROUP /var/run/mysqld

# Configure apache
RUN         sed -i 's|:80>|:'"$RUN_PORT"'>|' /etc/apache2/sites-available/000-default.conf && \
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
RUN         cd /tmp && \
            wget -O $BUGZILLA_DOWNLOAD_NAME.tar.gz $BUGZILLA_DOWNLOAD_URL && \
            tar xzf $BUGZILLA_DOWNLOAD_NAME.tar.gz -C /var/www/html && \
            rm $BUGZILLA_DOWNLOAD_NAME.tar.gz && \
            mv /var/www/html/$BUGZILLA_DOWNLOAD_NAME /var/www/html/bugzilla && \
            rm -rf /var/www/html/bugzilla/data && \
            ln -s /bugzilla/data /var/www/html/bugzilla/data && \
            cd /var/www/html/bugzilla && \
            /usr/bin/perl install-module.pl --all && \
            /var/www/html/bugzilla/checksetup.pl && \
            chown -R -h $RUN_USER:$RUN_GROUP /var/www/html/bugzilla && \
            sed -i "s|\$webservergroup.*|\$webservergroup = '"$RUN_GROUP"';|" /var/www/html/bugzilla/localconfig && \
            sed -i "s|\$db_name.*|\$db_name = '"$DB_NAME"';|" /var/www/html/bugzilla/localconfig && \
            sed -i "s|\$db_user.*|\$db_user = '"$DB_USER_NAME"';|" /var/www/html/bugzilla/localconfig && \
            sed -i "s|\$db_port.*|\$db_port = 3306;|" /var/www/html/bugzilla/localconfig && \
            sed -i "s|\$db_sock.*|\$db_sock = '/var/run/mysqld/mysqld.sock';|" /var/www/html/bugzilla/localconfig

# Configure helper scripts
RUN         mv /tmp/initialize /bugzilla/scripts && \
            chmod a+x /bugzilla/scripts/initialize && \
            mv /tmp/changeRootPassword /bugzilla/scripts && \
            chmod a+x /bugzilla/scripts/changeRootPassword && \
            mv /tmp/changeUserPassword /bugzilla/scripts && \
            chmod a+x /bugzilla/scripts/changeUserPassword && \
            mv /tmp/backup /bugzilla/scripts && \
            chmod a+x /bugzilla/scripts/backup && \
            mv /tmp/restore /bugzilla/scripts && \
            chmod a+x /bugzilla/scripts/restore && \
            mv /tmp/startup /bugzilla/scripts && \
            chmod a+x /bugzilla/scripts/startup && \
            mv /tmp/shutdown /bugzilla/scripts && \
            chmod a+x /bugzilla/scripts/shutdown

USER        $RUN_USER

EXPOSE      $RUN_PORT

ENTRYPOINT  ["/bugzilla/scripts/startup"]
