.. _Installation/sources:

========================
Installation from source
========================

Requirements
============

Some requirements are needed to install from source:

* python 2.6 or 2.7 (recommended)
* python-dev
* pip
* setuptools
* git (only for installing from the git repository)

**Note** that Python 2.6 will very soon be deprecated and Alignak is targeted to run with Python 2.7.

**Note** that those required libraries will not be installed with the Alignak pip package.

Installation from the git repository
====================================

This procedure is useful only if you want to be able to hack the code base. For only installing, use the next chapter.

Follow these steps:

* Create user *alignak* member of group *alignak*
* Add this user to the sudoers
* Login with this user account::

   adduser alignak
   adduser alignak sudo
   su - alignak

* Clone the Alignak repository::

    $ git clone https://github.com/Alignak-monitoring/alignak

* cd to alignak folder::

    $ cd alignak

* Install with python pip::

    $ sudo pip install .
        ...
        ...
        Successfully installed CherryPy-13.1.0 alignak certifi-2018.1.18 chardet-3.0.4 cheroot-6.0.0 docopt-0.6.2 idna-2.6 importlib-1.0.4 more-itertools-4.1.0 portend-2.2 psutil-5.4.3 requests-2.18.4 setproctitle-1.1.10 six-1.11.0 tempora-1.10 termcolor-1.1.0 ujson-1.35 urllib3-1.22

* Check installation::

    $ ll /usr/local/bin/alignak*
        -rwxr-xr-x 1 root root 166 janv. 30 09:30 /usr/local/bin/alignak-arbiter*
        -rwxr-xr-x 1 root root 165 janv. 30 09:30 /usr/local/bin/alignak-broker*
        -rwxr-xr-x 1 root root 170 janv. 30 09:30 /usr/local/bin/alignak-environment*
        -rwxr-xr-x 1 root root 165 janv. 30 09:30 /usr/local/bin/alignak-poller*
        -rwxr-xr-x 1 root root 170 janv. 30 09:30 /usr/local/bin/alignak-reactionner*
        -rwxr-xr-x 1 root root 167 janv. 30 09:30 /usr/local/bin/alignak-receiver*
        -rwxr-xr-x 1 root root 168 janv. 30 09:30 /usr/local/bin/alignak-scheduler*


**Note:** that installing with root/sudo credentials is needed because some scripts are created in the */usr/local/bin* directory.


Installation from the source
============================

Follow these steps:

* Create user *alignak* member of group *alignak*
* Add this user to the sudoers
* Login with this user account::

   adduser alignak
   adduser alignak sudo
   su - alignak

* Get source archive on this page: https://github.com/Alignak-monitoring/alignak/releases ::

   $ wget https://github.com/Alignak-monitoring/alignak/archive/1.0.0.tar.gz

* Uncompress the archive ::

    $ tar -xvf 1.0.0.tar.gz

* cd to alignak folder ::

    $ cd alignak-1.0.0

* Install with python pip (**sudo needed**)::

    $ sudo pip install .
        ...
        ...
        Successfully installed CherryPy-13.1.0 alignak certifi-2018.1.18 chardet-3.0.4 cheroot-6.0.0 docopt-0.6.2 idna-2.6 importlib-1.0.4 more-itertools-4.1.0 portend-2.2 psutil-5.4.3 requests-2.18.4 setproctitle-1.1.10 six-1.11.0 tempora-1.10 termcolor-1.1.0 ujson-1.35 urllib3-1.22


* Check installation::

    $ ll /usr/local/bin/alignak*
        -rwxr-xr-x 1 root root 166 janv. 30 09:30 /usr/local/bin/alignak-arbiter*
        -rwxr-xr-x 1 root root 165 janv. 30 09:30 /usr/local/bin/alignak-broker*
        -rwxr-xr-x 1 root root 170 janv. 30 09:30 /usr/local/bin/alignak-environment*
        -rwxr-xr-x 1 root root 165 janv. 30 09:30 /usr/local/bin/alignak-poller*
        -rwxr-xr-x 1 root root 170 janv. 30 09:30 /usr/local/bin/alignak-reactionner*
        -rwxr-xr-x 1 root root 167 janv. 30 09:30 /usr/local/bin/alignak-receiver*
        -rwxr-xr-x 1 root root 168 janv. 30 09:30 /usr/local/bin/alignak-scheduler*

    $ ll /usr/local/etc/alignak/
        total 60
        drwxr-xr-x 5 root root  4096 janv. 30 10:19 ./
        drwxr-xr-x 3 root root  4096 janv. 30 10:19 ../
        -rw-rw-r-- 1 root root  9237 janv. 30 10:19 alignak.cfg
        -rw-rw-r-- 1 root root 23092 janv. 30 10:19 alignak.ini
        -rw-rw-r-- 1 root root  2055 janv. 30 10:18 alignak-logger.json
        drwxr-xr-x 7 root root  4096 janv. 30 10:19 arbiter/
        drwxr-xr-x 2 root root  4096 janv. 30 10:19 certs/
        drwxr-xr-x 3 root root  4096 janv. 30 10:19 sample/

    $ ll /usr/local/var/log/alignak/
        total 8
        drwxr-xr-x 2 root root 4096 janv. 30 10:19 ./
        drwxr-xr-x 3 root root 4096 janv. 30 10:19 ../



**Note:** that the created directories are owned by the root user!

**Note:** that installing with root/sudo credentials is needed because some scripts are created in the */usr/local/bin* directory.

**Important note:** because of some pip specific behavior, installing Alignak requires to be connected as a user (and not as root) to run the pip command. If you really need to install from a root account, use ``pip install . -v --install-option='--prefix=/usr/local'``

Install as a python lib
=======================

In a virtualenv ::

  virtualenv env
  source env/bin/activate
  python setup.py install_lib
  python -c 'from alignak.bin import VERSION; print(VERSION)'

Or directly on your system ::

  sudo python setup.py install_lib
  python -c 'from alignak.bin import VERSION; print(VERSION)'

