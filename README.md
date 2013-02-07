#### I strongly recommend to try [iphoting/heroku-buildpack-php-tyler](https://github.com/iphoting/heroku-buildpack-php-tyler/) first!

Yet Another Cozy PHP buildpack for Heroku
========================

This is a build pack bundling PHP and Apache for Heroku apps. It contains
* PHP 5.4.11
* Apache 2.2.22
* ICU 50.1.2 (neccesary for intl support for Symfony2 Sonata bundles )

Target usages :
* Symfony2 websites
* PHP sites with/without [Composer](http://getcomposer.org/) support (Composer is an awesome package manager for PHP, try it!)

Yet it's not powerfull, but it's __working__. Also, compiled slugs sizes are about 150MB. (very bad!)

Consider other buildpacks :

* [iphoting/heroku-buildpack-php-tyler/](https://github.com/iphoting/heroku-buildpack-php-tyler) - Nginx+PHP-FPM build pack, now much better than this pack, but doesn't work with Symfony2
* [Swop/heroku-buildpack-symfony2](https://github.com/Swop/heroku-buildpack-symfony2) - In heavy development, written on Python
* [travisj/heroku-buildpack-nginx-php](https://github.com/travisj/heroku-buildpack-nginx-php) - Heroku Buildpack: Nginx + PHP
* [alkhoo/heroku-cedar-php-extension](https://github.com/alkhoo/heroku-cedar-php-extension) - Compiled a few library for Heroku's PHP extension - imagick.so, apc.so, gd.so


Configuration
-------------

The config files are bundled with the LP itself:

* conf/httpd.conf
* conf/php.ini

Usage 
-------------

1. Bundle empty "index.php" in root of your app, heroku needs it! Symfony2 by default have not this one.
2. Do 1 or 2
  1. Create heroku Application :

        $ heroku create myapp --buildpack https://github.com/sumkincpp/heroku-symfony2-buildpack.git

  2. Change buildpack on existing one :

        $ heroku config:add BUILDPACK_URL=https://github.com/sumkincpp/heroku-symfony2-buildpack.git

3. Push changes to Heroku.
4. !?!? 
5. Profit!

(PS. In Symfony2 apps configure your database. )

Pre-compiling binaries
----------------------

ATTENTION! Consider using [heroku/vulcan](https://github.com/heroku/vulcan) to build this binaries for heroku.

Precompilation is not neccesary until you decide to bundle some new modules into apache, etc.

It is recommended to build packages in heroku remote bash(man 'heroku run bash') or on similar systems. 
Currently heroku runs [Ubuntu 10.04 x64](http://releases.ubuntu.com/lucid/), you can check it by running on remote bash 'cat /etc/lsb-release'.
Using non-equal systems to precompile binaries may cause hellish library linking problems, those could not be easy solved. 
It also waste your time.

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
    # apparently, you decide to bundle apc, so add : "--enable-apc --enable-apc-mmap --with-php-config=/usr/bin/php-config"
    make
    make install
    cd ..

    # !! Not neccesary right now !!
    # Heroku doesn't have 1.0.0
    # In app/php/ext dir
    #
    # cd app/php/ext
    # ln -s /usr/lib/libssl.so.0.9.8 libssl.so.1.0.0
    # ln -s /usr/lib/libcrypto.so.0.9.8 libcrypto.so.1.0.0
    # cd ../../..
    
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
    echo '50.1.2' > icu/VERSION
    tar -zcvf icu-50.1.2.tar.gz icu

Hacking
-------

To change this buildpack, fork it on Github. 
Push up changes to your fork, then create a test app with --buildpack <your-github-url> and push to it.


TODO
-------
- [] using vulcan for build
- [] bundle apc
- [] bundle newrelic
- [] bundle AWS support 

Meta
----

Now supported by Fedor Sumkin
Created by Pedro Belo.
Many thanks to Keith Rarick for the help with assorted Unix topics :)
