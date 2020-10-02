FROM openshift/base-centos7

# This image provides an Apache+PHP environment for running PHP
# applications.

MAINTAINER SoftwareCollections.org <sclorg@redhat.com>

EXPOSE 8080

ENV PHP_VERSION=5.5 \
    PATH=$PATH:/opt/rh/php55/root/usr/bin

LABEL io.k8s.description="Platform for building and running PHP 5.5 applications" \
      io.k8s.display-name="Apache 2.4 with PHP 5.5" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,php,php55"

# Install Apache httpd and PHP
RUN yum install -y epel-release wget
RUN sudo yum-config-manager --enable rhel-server-rhscl-7-rpms
RUN wget http://www6.atomicorp.com/channels/atomic/centos/7/x86_64/RPMS/atomic-php55-php-mcrypt-5.5.31-31.el7.art.x86_64.rpm
RUN wget http://www6.atomicorp.com/channels/atomic/centos/7/x86_64/RPMS/atomic-php55-php-imap-5.5.31-31.el7.art.x86_64.rpm
RUN wget http://www6.atomicorp.com/channels/atomic/centos/7/x86_64/RPMS/atomic-php55-php-common-5.5.31-31.el7.art.x86_64.rpm
RUN wget https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libc-client-2007f-16.el7.x86_64.rpm
RUN wget http://mirror.centos.org/centos/7/os/x86_64/Packages/libtool-ltdl-2.4.2-22.el7_3.x86_64.rpm
RUN wget https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libmcrypt-2.5.8-13.el7.x86_64.rpm
RUN rpm -Uvh libmcrypt-2.5.8-13.el7.x86_64.rpm
RUN rpm -Uvh libtool-ltdl-2.4.2-22.el7_3.x86_64.rpm
RUN rpm -Uvh libc-client-2007f-16.el7.x86_64.rpm
RUN rpm -Uvh atomic-php55-php-common-5.5.31-31.el7.art.x86_64.rpm
RUN rpm -Uvh atomic-php55-php-imap-5.5.31-31.el7.art.x86_64.rpm
RUN rpm -Uvh atomic-php55-php-mcrypt-5.5.31-31.el7.art.x86_64.rpm
RUN yum install atomic-php55-php-mcrypt
RUN yum install -y centos-release-scl 
ENV INSTALL_PKGS="httpd24 php55 php55-php php55-php-mysqlnd php55-php-pgsql php55-php-bcmath php55-php-devel \
                  php55-php-fpm php55-php-gd php55-php-intl php55-php-ldap php55-php-mbstring php55-php-pdo \
                  php55-php-pecl-memcache php55-php-process php55-php-soap php55-php-opcache php55-php-xml \
                  php55-php-gmp" 
RUN yum install -y --setopt=tsflags=nodocs --enablerepo=centosplus $INSTALL_PKGS
RUN rpm -V $INSTALL_PKGS 
RUN yum -y clean all --enablerepo='*'

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Copy extra files to the image.
COPY ./root/ /

# In order to drop the root user, we have to make some directories world
# writeable as OpenShift default security model is to run the container under
# random UID.
RUN sed -i -f /opt/app-root/etc/httpdconf.sed /opt/rh/httpd24/root/etc/httpd/conf/httpd.conf && \
    sed -i '/php_value session.save_path/d' /opt/rh/httpd24/root/etc/httpd/conf.d/php55-php.conf && \
    echo "IncludeOptional /opt/app-root/etc/conf.d/*.conf" >> /opt/rh/httpd24/root/etc/httpd/conf/httpd.conf && \
    head -n151 /opt/rh/httpd24/root/etc/httpd/conf/httpd.conf | tail -n1 | grep "AllowOverride All" || exit && \
    mkdir /tmp/sessions && \
    chown -R 1001:0 /opt/app-root /tmp/sessions && \
    chmod -R a+rwx /tmp/sessions && \
    chmod -R ug+rwx /opt/app-root && \
    chmod -R a+rwx /opt/rh/php55/root/etc && \
    chmod -R a+rwx /opt/rh/httpd24/root/var/run/httpd

USER 1001

# Set the default CMD to print the usage of the language image
CMD $STI_SCRIPTS_PATH/usage
