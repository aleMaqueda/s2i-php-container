FROM centos/s2i-base-centos7

# This image provides an Apache+PHP environment for running PHP
# applications.

EXPOSE 8080
EXPOSE 8443

# Description
# This image provides an Apache 2.4 + PHP 7.0 environment for running PHP applications.
# Exposed ports:
# * 8080 - alternative port for http

ENV PHP_VERSION=7.0 \
    PHP_VER_SHORT=70 \
    NAME=php \
    PATH=$PATH:/opt/rh/rh-php70/root/usr/bin

ENV SUMMARY="Platform for building and running PHP $PHP_VERSION applications" \
    DESCRIPTION="PHP $PHP_VERSION available as container is a base platform for \
building and running various PHP $PHP_VERSION applications and frameworks. \
PHP is an HTML-embedded scripting language. PHP attempts to make it easy for developers \
to write dynamically generated web pages. PHP also offers built-in database integration \
for several commercial and non-commercial database management systems, so writing \
a database-enabled webpage with PHP is fairly simple. The most common use of PHP coding \
is probably as a replacement for CGI scripts."

LABEL summary="${SUMMARY}" \
      description="${DESCRIPTION}" \
      io.k8s.description="${DESCRIPTION}" \
      io.k8s.display-name="Apache 2.4 with PHP ${PHP_VERSION}" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,${NAME},${NAME}${PHP_VER_SHORT},rh-${NAME}${PHP_VER_SHORT}" \
      io.openshift.s2i.scripts-url="image:///usr/libexec/s2i" \
      io.s2i.scripts-url="image:///usr/libexec/s2i" \
      name="centos/${NAME}-${PHP_VER_SHORT}-centos7" \
      com.redhat.component="rh-${NAME}${PHP_VER_SHORT}-container" \
      version="${PHP_VERSION}" \
      help="For more information visit https://github.com/sclorg/s2i-${NAME}-container" \
      usage="s2i build https://github.com/sclorg/s2i-php-container.git --context-dir=${PHP_VERSION}/test/test-app centos/${NAME}-${PHP_VER_SHORT}-centos7 sample-server" \
      maintainer="SoftwareCollections.org <sclorg@redhat.com>"

# Install Apache httpd and PHP
RUN yum install -y centos-release-scl && \
    INSTALL_PKGS="rh-php70 rh-php70-php rh-php70-php-mysqlnd rh-php70-php-pgsql rh-php70-php-bcmath \
                  rh-php70-php-gd rh-php70-php-intl rh-php70-php-ldap rh-php70-php-mbstring rh-php70-php-pdo \
                  rh-php70-php-process rh-php70-php-soap rh-php70-php-opcache rh-php70-php-xml \
                  rh-php70-php-gmp httpd24-mod_ssl" && \
    yum install -y --setopt=tsflags=nodocs $INSTALL_PKGS --nogpgcheck && \
    rpm -V $INSTALL_PKGS && \
    yum -y clean all --enablerepo='*'
RUN yum install -y epel-release
RUN yum update -y update
RUN yum install -y sendmail sendmail-cf telnet wget libmcrypt-devel net-tools
RUN wget http://mirror.centos.org/centos/7/os/x86_64/Packages/libtool-ltdl-2.4.2-22.el7_3.x86_64.rpm
RUN wget https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libmcrypt-2.5.8-13.el7.x86_64.rpm
RUN wget https://dl.iuscommunity.org/pub/ius/archive/CentOS/7/x86_64/php70u-mcrypt-7.0.33-1.ius.centos7.x86_64.rpm
RUN wget https://dl.iuscommunity.org/pub/ius/archive/CentOS/7/x86_64/php70u-common-7.0.33-1.ius.centos7.x86_64.rpm

RUN rpm -Uvh libtool-ltdl-2.4.2-22.el7_3.x86_64.rpm
#RUN rpm -Uvh libmcrypt-2.5.8-13.el7.x86_64.rpm
RUN rpm -Uvh php70u-common-7.0.33-1.ius.centos7.x86_64.rpm
RUN rpm -Uvh php70u-mcrypt-7.0.33-1.ius.centos7.x86_64.rpm
RUN ln -s /usr/lib64/php/modules/mcrypt.so /opt/rh/rh-php70/root/usr/lib64/php/modules/mcrypt.so


ENV PHP_CONTAINER_SCRIPTS_PATH=/usr/share/container-scripts/php/ \
    APP_DATA=${APP_ROOT}/src \
    PHP_DEFAULT_INCLUDE_PATH=/opt/rh/rh-php70/root/usr/share/pear \
    PHP_SYSCONF_PATH=/etc/opt/rh/rh-php70 \
    PHP_HTTPD_CONF_FILE=rh-php70-php.conf \
    HTTPD_CONFIGURATION_PATH=${APP_ROOT}/etc/conf.d \
    HTTPD_MAIN_CONF_PATH=/etc/httpd/conf \
    HTTPD_MAIN_CONF_D_PATH=/etc/httpd/conf.d \
    HTTPD_VAR_RUN=/var/run/httpd \
    HTTPD_DATA_PATH=/var/www \
    HTTPD_DATA_ORIG_PATH=/opt/rh/httpd24/root/var/www \
    HTTPD_VAR_PATH=/opt/rh/httpd24/root/var \
    SCL_ENABLED=rh-php70

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Copy extra files to the image.
COPY ./root/ /

# Reset permissions of filesystem to default values
RUN /usr/libexec/container-setup && rpm-file-permissions

RUN sed -i "s/^\(.*\)define\(.*\)SMART_HOST\(.*\)$/define\2SMART_HOST\3/" /etc/mail/sendmail.mc
RUN sed -i "s/smtp.your.provider/relay.imss.gob.mx/" /etc/mail/sendmail.mc
RUN sed -i "s/Port=smtp,Addr=127.0.0.1/Port=smtp/" /etc/mail/sendmail.mc
RUN echo "relay.imss.gob.mx" >> /etc/mail/local-host-names
RUN echo "Connect:172.16.23.18    RELAY" >> /etc/mail/access
RUN echo "Connect:172.16.23.17    RELAY" >> /etc/mail/access
RUN echo "Connect:10.100.1.173    RELAY" >> /etc/mail/access
RUN echo "172.16.23.18 relay.imss.gob.mx"
RUN m4 /etc/mail/sendmail.mc > /etc/mail/sendmail.cf 
RUN makemap hash /etc/mail/access.db < /etc/mail/access
RUN make -C /etc/mail 
RUN systemctl enable sendmail
USER 1001

# Set the default CMD to print the usage of the language image
CMD $STI_SCRIPTS_PATH/usage
