.. _howitworks/ssl:

Alignak inter-daemon SSL communication
======================================

Introduction
------------

SSL keys:
::

    # Generate private key (without pass phrase protection!)
    # For pass phrase protection, add -aes256 parameter
    $ openssl genrsa -out alignak-private.key 2048
        Generating RSA private key, 2048 bit long modulus
        .........................................................................................+++
        ...............................................................................................................+++
        unable to write 'random state'
        e is 65537 (0x010001)

    # Generate public key
    $ openssl rsa -in private.key -out public.key
        Enter pass phrase for private.key: (alignak)
        writing RSA key


    # Create a self-signed certificate
    $ openssl req -new -x509 -days 365 -key alignak-private.key -out alignak.crt
    $ openssl req -new -x509 -nodes -sha1 -key alignak-private.key -out alignak.crt -days 365 -config /etc/ssl/openssl.cnf
        You are about to be asked to enter information that will be incorporated
        into your certificate request.
        What you are about to enter is what is called a Distinguished Name or a DN.
        There are quite a few fields but you can leave some blank
        For some fields there will be a default value,
        If you enter '.', the field will be left blank.
        -----
        Country Name (2 letter code) [AU]:FR
        State or Province Name (full name) [Some-State]:
        Locality Name (eg, city) []:
        Organization Name (eg, company) [Internet Widgits Pty Ltd]:
        Organizational Unit Name (eg, section) []:
        Common Name (e.g. server FQDN or YOUR name) []:alignak
        Email Address []:



Configuration
=============

Introduction
------------

We will introduce the configuration part with an example.

We set the *Scheduler* daemon in load balancing mode.

We will distribute the following daemons on three servers:

* *Arbiter* daemon is on *ServerA* with IP 192.168.0.1
* *Scheduler master* is on *ServerB* with IP 192.168.0.2
* *Scheduler spare* is on *ServerC* with IP 192.168.0.3

**Note**: it is possible to have more than 2 *Scheduler* daemons, this example is for 2.

First step
----------

We need to install Alignak on the 3 servers.

Second step
-----------

We configure the 2 *Scheduler* daemons in the *Arbiter* configuration on *ServerA*.

In the folder *etc/alignak/arbiter/daemons*:

* rename the file *scheduler-master.cfg* in *scheduler-mars.cfg*. In this file, define the IP of *ServerB* like::

    scheduler_name      scheduler-mars
    address             192.168.0.2
    spare               0

* copy this file *scheduler-mars.cfg* to *scheduler-venus.cfg* (the name is not really important). In this file, define the scheduler_name, the IP of *ServerC* like::

    scheduler_name      scheduler-venus
    address             192.168.0.3
    spare               0


Third step
----------

On *ServerB* and *ServerC*, start the *Scheduler* daemons, and only those daemons.

On *ServerA*, start the *Arbiter* daemon.

Conclusion
----------

You run now Alignak with load balancing for Scheduler.
You can do the same for other daemons.
