.. Copyright 2016 tsuru authors. All rights reserved.
   Use of this source code is governed by a BSD-style
   license that can be found in the LICENSE file.

++++++++++++++++++++++++++
Deploying PHP applications
++++++++++++++++++++++++++

Overview
========

This document is a hands-on guide to deploying a simple PHP application in
tsuru. The example application will be a very simple Wordpress project
associated to a MySQL service. It's applicable to any php over apache
application.

Creating the app
================

To create an app, you use the command `app-create`:

.. highlight:: bash

::

    $ tsuru app-create <app-name> <app-platform>

For PHP, the app platform is, guess what, ``php``! Let's be over creative
and develop a never-developed tutorial-app: a blog, and its name will also be
very creative, let's call it "blog":

.. highlight:: bash

::

    $ tsuru app-create blog php

To list all available platforms, use the command `platform-list`.

You can see all your applications using the command `app-list`:

.. highlight:: bash

::

    $ tsuru app-list
    +-------------+-------------------------+--------------------------+
    | Application | Units State Summary     | Address                  |
    +-------------+-------------------------+--------------------------+
    | blog        | 0 of 0 units in-service | blog.192.168.50.4.nip.io |
    +-------------+-------------------------+--------------------------+

Application code
================

This document will not focus on how to write a php blog, you can download the
entire source direct from wordpress: http://wordpress.org/latest.zip. Here is
all you need to do with your project:

.. highlight:: bash

::

    # Download and unpack wordpress
    $ wget http://wordpress.org/latest.zip
    $ unzip latest.zip
    # Preparing wordpress for tsuru
    $ cd wordpress
    # Notify tsuru about the necessary packages
    $ echo php5-mysql > requirements.apt
    # Preparing the application to receive the tsuru environment related to the mysql service
    $ sed "s/'database_name_here'/getenv('MYSQL_DATABASE_NAME')/; \
                s/'username_here'/getenv('MYSQL_USER')/; \
                s/'localhost'/getenv('MYSQL_HOST')/; \
                s/'password_here'/getenv('MYSQL_PASSWORD')/" \
                wp-config-sample.php  > wp-config.php
    # Creating a local Git repository
    $ git init
    $ git add .
    $ git commit -m 'initial project version'


Git deployment
==============

When you create a new app, tsuru will display the Git remote that you should
use. You can always get it using the command `app-info`:

.. highlight:: bash

::

    $ tsuru app-info --app blog
    Application: blog
    Repository: git@192.168.50.4.nip.io:blog.git
    Platform: php
    Teams: admin
    Address: blog.192.168.50.4.nip.io
    Owner: admin@example.com
    Team owner: admin
    Deploys: 0
    Pool: theonepool

    App Plan:
    +---------------+--------+------+-----------+--------+---------+
    | Name          | Memory | Swap | Cpu Share | Router | Default |
    +---------------+--------+------+-----------+--------+---------+
    | autogenerated | 0 MB   | 0 MB | 100       |        | false   |
    +---------------+--------+------+-----------+--------+---------+

The Git remote will be used to deploy your application using Git. You can just
push to tsuru remote and your project will be deployed:

.. highlight:: console

::

    $ git push git@192.168.50.4.nip.io:blog.git master
    Counting objects: 1295, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (1271/1271), done.
    Writing objects: 100% (1295/1295), 6.09 MiB | 5.65 MiB/s, done.
    Total 1295 (delta 102), reused 0 (delta 0)
    remote: text
    remote: Deploying the PHP application...
    remote: tar: Removing leading `/' from member names
    #########################################
    #  OMIT DEPENDENCIES STEPS (see below)  #
    #########################################
    remote:
    remote: ---- Building application image ----
    remote:  ---> Sending image to repository (51.40MB)
    remote:  ---> Cleaning up
    remote:
    remote: ---- Starting 1 new unit ----
    remote:  ---> Started unit 027c2a31a0...
    remote:
    remote: ---- Binding and checking 1 new units ----
    remote:  ---> Bound and checked unit 027c2a31a0
    remote:
    remote: ---- Adding routes to 1 new units ----
    remote:  ---> Added route to unit 027c2a31a0
    remote:
    remote: OK
    To git@192.168.50.4.nip.io:blog.git
     * [new branch]      master -> master

If you get a "Permission denied (publickey).", make sure you're member of a
team and have a public key added to tsuru. To add a key, use the command
`key-add`:

.. highlight:: bash

::

    $ tsuru key-add mykey ~/.ssh/id_dsa.pub

You can use ``git remote add`` to avoid typing the entire remote url every time
you want to push:

.. highlight:: bash

::

    $ git remote add tsuru git@192.168.50.4.nip.io:blog.git

Then you can run:

.. highlight:: bash

::

    $ git push tsuru master
    Everything up-to-date

And you will be also able to omit the ``--app`` flag from now on:

.. highlight:: bash

::

    $ tsuru app-info
    Application: blog
    Repository: git@192.168.50.4.nip.io:blog.git
    Platform: php
    Teams: admin
    Address: blog.192.168.50.4.nip.io
    Owner: admin@example.com
    Team owner: admin
    Deploys: 1
    Pool: theonepool
    Units: 1
    +------------+---------+
    | Unit       | State   |
    +------------+---------+
    | 027c2a31a0 | started |
    +------------+---------+

    App Plan:
    +---------------+--------+------+-----------+--------+---------+
    | Name          | Memory | Swap | Cpu Share | Router | Default |
    +---------------+--------+------+-----------+--------+---------+
    | autogenerated | 0 MB   | 0 MB | 100       |        | false   |
    +---------------+--------+------+-----------+--------+---------+

Listing dependencies
====================

In the last section we omitted the dependencies step of deploy. In tsuru, an
application can have two kinds of dependencies:

* **Operating system dependencies**, represented by packages in the package manager
  of the underlying operating system (e.g.: ``yum`` and ``apt-get``);
* **Platform dependencies**, represented by packages in the package manager of the
  platform/language (e.g. in Python, ``pip``).

All ``apt-get`` dependencies must be specified in a ``requirements.apt`` file,
located in the root of your application, and pip dependencies must be located
in a file called ``requirements.txt``, also in the root of the application.
Since we will use MySQL with PHP, we need to install the package depends on just
one ``apt-get`` package:
``php5-mysql``, so here is how ``requirements.apt``
looks like:

::

    php5-mysql


You can see the complete output of installing these dependencies below:

.. highlight:: bash

::

    % git push tsuru master
    #####################################
    #                OMIT               #
    #####################################
    Counting objects: 1155, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (1124/1124), done.
    Writing objects: 100% (1155/1155), 4.01 MiB | 327 KiB/s, done.
    Total 1155 (delta 65), reused 0 (delta 0)
    remote: Cloning into '/home/application/current'...
    remote: Reading package lists...
    remote: Building dependency tree...
    remote: Reading state information...
    remote: The following extra packages will be installed:
    remote:   libmysqlclient18 mysql-common
    remote: The following NEW packages will be installed:
    remote:   libmysqlclient18 mysql-common php5-mysql
    remote: 0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
    remote: Need to get 1042 kB of archives.
    remote: After this operation, 3928 kB of additional disk space will be used.
    remote: Get:1 http://archive.ubuntu.com/ubuntu/ quantal/main mysql-common all 5.5.27-0ubuntu2 [13.7 kB]
    remote: Get:2 http://archive.ubuntu.com/ubuntu/ quantal/main libmysqlclient18 amd64 5.5.27-0ubuntu2 [949 kB]
    remote: Get:3 http://archive.ubuntu.com/ubuntu/ quantal/main php5-mysql amd64 5.4.6-1ubuntu1 [79.0 kB]
    remote: Fetched 1042 kB in 1s (739 kB/s)
    remote: Selecting previously unselected package mysql-common.
    remote: (Reading database ... 23874 files and directories currently installed.)
    remote: Unpacking mysql-common (from .../mysql-common_5.5.27-0ubuntu2_all.deb) ...
    remote: Selecting previously unselected package libmysqlclient18:amd64.
    remote: Unpacking libmysqlclient18:amd64 (from .../libmysqlclient18_5.5.27-0ubuntu2_amd64.deb) ...
    remote: Selecting previously unselected package php5-mysql.
    remote: Unpacking php5-mysql (from .../php5-mysql_5.4.6-1ubuntu1_amd64.deb) ...
    remote: Processing triggers for libapache2-mod-php5 ...
    remote:  * Reloading web server config
    remote:    ...done.
    remote: Setting up mysql-common (5.5.27-0ubuntu2) ...
    remote: Setting up libmysqlclient18:amd64 (5.5.27-0ubuntu2) ...
    remote: Setting up php5-mysql (5.4.6-1ubuntu1) ...
    remote: Processing triggers for libc-bin ...
    remote: ldconfig deferred processing now taking place
    remote: Processing triggers for libapache2-mod-php5 ...
    remote:  * Reloading web server config
    remote:    ...done.
    remote: sudo: unable to resolve host 8cf20f4da877
    remote: sudo: unable to resolve host 8cf20f4da877
    remote: debconf: unable to initialize frontend: Dialog
    remote: debconf: (Dialog frontend will not work on a dumb terminal, an emacs shell buffer, or without a controlling terminal.)
    remote: debconf: falling back to frontend: Readline
    remote: debconf: unable to initialize frontend: Dialog
    remote: debconf: (Dialog frontend will not work on a dumb terminal, an emacs shell buffer, or without a controlling terminal.)
    remote: debconf: falling back to frontend: Readline
    remote:
    remote: Creating config file /etc/php5/mods-available/mysql.ini with new version
    remote: debconf: unable to initialize frontend: Dialog
    remote: debconf: (Dialog frontend will not work on a dumb terminal, an emacs shell buffer, or without a controlling terminal.)
    remote: debconf: falling back to frontend: Readline
    remote:
    remote: Creating config file /etc/php5/mods-available/mysqli.ini with new version
    remote: debconf: unable to initialize frontend: Dialog
    remote: debconf: (Dialog frontend will not work on a dumb terminal, an emacs shell buffer, or without a controlling terminal.)
    remote: debconf: falling back to frontend: Readline
    remote:
    remote: Creating config file /etc/php5/mods-available/pdo_mysql.ini with new version
    remote:
    remote:  ---> App will be restarted, please check its log for more details...
    remote:
    To git@192.168.50.4.nip.io:blog.git
     * [new branch]      master -> master


Running the application
=======================

As you can see, in the deploy output there is a step described as "App will be
restarted". In this step, tsuru will restart your app if it's running, or start
it if it's not.
Now that the app is deployed, you can access it from your browser, getting the
IP or host listed in ``app-list`` and opening it. For example,
in the list below:

::

    $ tsuru app-list
    +-------------+-------------------------+---------------------+
    | Application | Units State Summary     | Address             |
    +-------------+-------------------------+---------------------+
    | blog        | 1 of 1 units in-service | blog.cloud.tsuru.io |
    +-------------+-------------------------+---------------------+


Customizing the platform
========================

The PHP platform supports customizations in the frontend and the interpreter,
for more details, check the `README of the platform
<https://github.com/tsuru/basebuilder/blob/master/php/README.md>`_.

Going further
=============

For more information, you can dig into `tsuru docs <http://docs.tsuru.io>`_, or
read `complete instructions of use for the tsuru command
<https://tsuru.readthedocs.org>`_.
