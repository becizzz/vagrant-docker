FROM ubuntu:trusty

MAINTAINER Mārtiņš Bērtiņš, martins.bertins@wunderkraut.com

RUN apt-get update

# Set up SSH
RUN apt-get -qqy install openssh-server
RUN mkdir -p /var/run/sshd
# No idea what these do :)
RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config
RUN sed -ri 's/#UsePAM no/UsePAM no/g' /etc/ssh/sshd_config
RUN sed -ri 's/PermitRootLogin without-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
# Change root password
RUN echo 'root:root' |chpasswd
# Generate a host key
RUN service ssh start; service ssh stop

# Set up Apache
RUN apt-get -qqy install apache2-mpm-prefork apache2-utils
RUN a2enmod rewrite
RUN mkdir -p /etc/apache2/conf.d/
RUN echo "ServerName localhost" | tee /etc/apache2/conf.d/fqdn
ADD conf/apache-default /etc/apache2/sites-enabled/000-default

# Set up PHP
RUN apt-get -qqy install libapache2-mod-php5 php5 php5-cli php5-mysql php5-curl php5-dev php5-gd php-pear php-apc php5-xdebug
ADD conf/php-default /etc/php5/apache2/php.ini

# Set up Mysql
RUN apt-get -qqy install mysql-server mysql-common mysql-client
ADD conf/mysql-default /etc/mysql/my.cnf
RUN mysql_install_db && chown -R mysql:mysql /var/lib/mysql && (/usr/bin/mysqld_safe &) && sleep 5 && \
    mysql -uroot -e "GRANT ALL PRIVILEGES ON *.* TO root@'%' WITH GRANT OPTION; FLUSH PRIVILEGES;"

# Set up Drush
RUN pear channel-discover pear.drush.org
RUN pear install drush/drush

# Set up SASS
RUN apt-get -qqy install ruby
RUN gem install sass

# Set up Supervisor
RUN apt-get -qqy install supervisor
ADD conf/ubuntu-lamp.conf /etc/supervisor/conf.d/ubuntu-lamp.conf

RUN mkdir -p /db && mkdir -p /var/www

VOLUME ["/db", "/var/www"]

EXPOSE 80 3306 22

CMD ["/usr/bin/supervisord", "-n"]
