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

# Install packages
RUN         apt-get update && \
            apt-get install --assume-yes \
                apache2 \
                mysql-server \
                libappconfig-perl \
                libdate-calc-perl \
                libtemplate-perl \
                libmime-perl \
                build-essential \
                libdatetime-timezone-perl \
                libdatetime-perl \
                libemail-sender-perl \
                libemail-mime-perl \
                libemail-mime-modifier-perl \
                libdbi-perl \
                libdbd-mysql-perl \
                libcgi-pm-perl \
                libmath-random-isaac-perl \
                libmath-random-isaac-xs-perl \
                apache2-mpm-prefork \
                libapache2-mod-perl2 \
                libapache2-mod-perl2-dev \
                libchart-perl \
                libxml-perl \
                libxml-twig-perl \
                perlmagick \
                libgd-graph-perl \
                libtemplate-plugin-gd-perl \
                libsoap-lite-perl \
                libhtml-scrubber-perl \
                libjson-rpc-perl \
                libtheschwartz-perl \
                libtest-taint-perl \
                libauthen-radius-perl \
                libfile-slurp-perl \
                libencode-detect-perl \
                libmodule-build-perl \
                libnet-ldap-perl \
                libauthen-sasl-perl \
                libtemplate-perl-doc \
                libfile-mimeinfo-perl \
                libhtml-formattext-withlinks-perl \
                libfile-which-perl \
                libgd-dev \
                libmysqlclient-dev \
                lynx-cur \
                graphviz \
                python-sphinx \
                rst2pdf \
                wget \
                vim

# Create user and group
RUN         groupadd -g $hostgid bugzilla && \
            useradd -u $hostuid -g $hostgid -M bugzilla

# Create folder for bugzilla
RUN         mkdir /bugzilla && \
            chown bugzilla:bugzilla /bugzilla

COPY        init /bugzilla
COPY        startup /bugzilla
COPY        bugzilla.conf /etc/apache2/sites-available

# Configure MySQL
RUN         sed -i 's|user.*|user            = bugzilla|' /etc/mysql/my.cnf && \
            sed -i 's|datadir.*|datadir         = /bugzilla/database|' /etc/mysql/my.cnf && \
            sed -i 's|max_allowed_packet.*|max_allowed_packet      = 100M|' /etc/mysql/my.cnf && \
            sed -i 's|key_buffer.*|key_buffer_size         = 16M|' /etc/mysql/my.cnf && \
            sed -i 's|myisam-recover.*|myisam-recover-options  = BACKUP|' /etc/mysql/my.cnf && \
            sed -i '\|\[mysqld\]|a ft_min_word_len         = 2' /etc/mysql/my.cnf && \
            chown -R bugzilla:bugzilla /var/run/mysqld && \
            chown -R bugzilla:bugzilla /var/log/mysql

# Configure bugzilla
RUN         cd ~ && \
            wget https://ftp.mozilla.org/pub/mozilla.org/webtools/${downloadname}.tar.gz && \
            tar xzf ${downloadname}.tar.gz -C /var/www/html && \
            rm ${downloadname}.tar.gz && \
            mv /var/www/html/${downloadname} /var/www/html/${guestappname} && \
            cd /var/www/html/${guestappname} && \
            /usr/bin/perl install-module.pl --all && \
            /var/www/html/${guestappname}/checksetup.pl && \
            sed -i "s|\$webservergroup.*|\$webservergroup = 'bugzilla';|" /var/www/html/${guestappname}/localconfig && \
            sed -i "s|\$db_pass.*|\$db_pass = 'bugs';|" /var/www/html/${guestappname}/localconfig

# Configure apache
RUN         sed -i 's|:80>|:'"$guestport"'>|' /etc/apache2/sites-available/000-default.conf && \
            sed -i 's|#ServerName www.example.com|ServerName localhost|' /etc/apache2/sites-available/000-default.conf && \
            sed -i 's|/guestappname|/'"$guestappname"'|' /etc/apache2/sites-available/bugzilla.conf && \
            sed -i 's|Listen 80|Listen '"$guestport"'|' /etc/apache2/ports.conf && \
            echo "ServerName localhost" > /etc/apache2/conf-available/servername.conf && \
            a2enconf servername && \
            a2enmod cgi expires headers rewrite && \
            a2ensite bugzilla && \
            chown -R bugzilla:bugzilla /var/run/apache2 && \
            chown -R bugzilla:bugzilla /var/log/apache2

# Grant permits for helper scripts
RUN         chmod a+x /bugzilla/init && \
            chmod a+x /bugzilla/startup

USER        bugzilla

EXPOSE      $guestport

ENTRYPOINT  ["/bugzilla/startup"]
