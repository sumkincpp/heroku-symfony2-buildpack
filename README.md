Apache+PHP build pack
========================

This is a build pack bundling PHP and Apache for Heroku apps.

Configuration
-------------

The config files are bundled with the LP itself:

* conf/httpd.conf
* conf/php.ini


Pre-compiling binaries
----------------------
    
    # make some temporary dir, go to it

    # apache
    mkdir app
    wget http://www.apache.org/dist/httpd/httpd-2.2.22.tar.gz
    tar xvzf httpd-2.2.22.tar.gz
    cd httpd-2.2.22
    ./configure --prefix=`pwd`/../app/apache --enable-rewrite
    make
    make install
    cd ..
    
    # icu for 'intl'
    mkdir app
    wget http://download.icu-project.org/files/icu4c/50.1.2/icu4c-50_1_2-src.tgz
    tar xvzf icu4c-50_1_2-src.tar.gz
    cd icu4c-50_1_2-src/source
    ./configure --prefix=`pwd`/../app/icu
    make
    make install
    cd ../..

    # php
    wget http://www.php.net/get/php-5.4.11.tar.gz/from/us1.php.net/mirror
    mv mirror php.tar.gz
    tar xzvf php.tar.gz
    cd php-5.4.11/
    ./configure --prefix=`pwd`/../app/php --with-apxs2=`pwd`/../app/apache/bin/apxs --with-mysql --with-pdo-mysql --with-pgsql --with-pdo-pgsql --with-iconv --with-gd --with-curl=/usr/lib --with-config-file-path=/app/php --enable-soap=shared --with-openssl --enable-intl --with-icu-dir=`pwd`/../app/icu
    make
    make install
    cd ..

    # Heroku doesn't have 1.0.0
    # In app/php/ext dir
    #
    cd app/php/ext
    ln -s /usr/lib/libssl.so.0.9.8 libssl.so.1.0.0
    ln -s /usr/lib/libcrypto.so.0.9.8 libcrypto.so.1.0.0
    cd ../../..
    
    # php extensions    
    mkdir app/php/ext
    #sudo apt-get install libmysqlclient-dev
    #cp /usr/lib/libmysqlclient.so.15 app/php/ext/
    
    # pear
    apt-get install php5-dev php-pear
    pear config-set php_dir app/php
    pecl install apc
    mkdir app/php/include/php/ext/apc
    cp /usr/lib/php5/20100525/apc.so app/php/ext/
    cp /usr/include/php5/ext/apc/apc_serializer.h app/php/include/php/ext/apc/
    
    
    # package
    cd app
    echo '2.2.22' > apache/VERSION
    tar -zcvf apache-2.2.22.tar.gz apache
    echo '5.4.11' > php/VERSION
    tar -zcvf php-5.4.11.tar.gz php


Hacking
-------

To change this buildpack, fork it on Github. Push up changes to your fork, then create a test app with --buildpack <your-github-url> and push to it.


Meta
----

Created by Pedro Belo.
Many thanks to Keith Rarick for the help with assorted Unix topics :)