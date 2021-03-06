.. _howitworks/run_alignak:

===============
Running Alignak
===============

With packaging
==============

If you install with packaging (DEB, RPM...), see the package documentation for the best solution to
configure the daemons and to start/stop Alignak.


With sources and pip
====================

Individual start / stop
-----------------------
All the Alignak daemons have a script that can be launched with command line parameters.

All the command line parameters are optional because default values are used by the daemon when it starts but it is recommended to use a daemon configuration file with the `-c` option.

Without a configuration file, the daemon will use some default values:

    - use a generic prefix for its files except if a daemon name is provided on the command line. The prefix is the daemon name if it is provided, else it is the daemon type with an ending `d` character (eg. brokerd for a broker)
    - the default daemon name is the daemon type (eg. *broker* for a broker daemon)
    - create its pid (*prefix.pid*) and log (*prefix.log*) file in the current working directory.
    - It will also use a default port to listen to the other daemons (arbiter: 7770, scheduler: 7768, broker: 7772, poller: 7771, reactionner: 7769, receiver: 7773).

Other command line parameters are available, but they are really rarely used ;)

For all the daemons (broker, poller, receiver, reactionner, scheduler)::

   $ alignak-broker -h
   usage: alignak-broker [-h] [-v] [-n DAEMON_NAME] [-c CONFIG_FILE] [-d] [-r]
                         [-f DEBUG_FILE] [-p PORT] [-l LOCAL_LOG]

   optional arguments:
     -h, --help            show this help message and exit
     -v, --version         show program's version number and exit
     -n DAEMON_NAME, --name DAEMON_NAME
                           Daemon unique name. Must be unique for the same daemon
                           type.
     -c CONFIG_FILE, --config CONFIG_FILE
                           Daemon configuration file
     -d, --daemon          Run as a daemon
     -r, --replace         Replace previous running daemon
     -f DEBUG_FILE, --debugfile DEBUG_FILE
                           File to dump debug logs
     -p PORT, --port PORT  Port used by the daemon
     -l LOCAL_LOG, --local_log LOCAL_LOG
                           File to use for daemon log


The arbiter is slightly different because it needs to receive the monitoring configuration that is to be loaded::

   $ alignak-arbiter -h
   usage: alignak-arbiter [-h] [-v] -a MONITORING_FILES [-V] [-k ALIGNAK_NAME]
                          [-n DAEMON_NAME] [-c CONFIG_FILE] [-d] [-r]
                          [-f DEBUG_FILE] [-p PORT] [-l LOCAL_LOG]

   optional arguments:
     -h, --help            show this help message and exit
     -v, --version         show program's version number and exit
     -a MONITORING_FILES, --arbiter MONITORING_FILES
                           Monitored configuration file(s), (multiple -a can be
                           used, and they will be concatenated to make a global
                           configuration file)
     -V, --verify-config   Verify configuration file(s) and exit
     -k ALIGNAK_NAME, --alignak-name ALIGNAK_NAME
                           Set the name of the arbiter to pick in the
                           configuration files For a spare arbiter, this
                           parameter must contain its name!
     -n DAEMON_NAME, --name DAEMON_NAME
                           Daemon unique name. Must be unique for the same daemon
                           type.
     -c CONFIG_FILE, --config CONFIG_FILE
                           Daemon configuration file
     -d, --daemon          Run as a daemon
     -r, --replace         Replace previous running daemon
     -f DEBUG_FILE, --debugfile DEBUG_FILE
                           File to dump debug logs
     -p PORT, --port PORT  Port used by the daemon
     -l LOCAL_LOG, --local_log LOCAL_LOG
                           File to use for daemon log

With the default installed configuration::

    $ alignak-broker -c /usr/local/etc/alignak/daemons/brokerd.ini
    $ alignak-scheduler -c /usr/local/etc/alignak/daemons/schedulerd.ini
    $ alignak-poller -c /usr/local/etc/alignak/daemons/pollerd.ini
    $ alignak-reactionner -c /usr/local/etc/alignak/daemons/reactionnerd.ini
    $ alignak-receiver -c /usr/local/etc/alignak/daemons/receiverd.ini
    $ alignak-arbiter -c /usr/local/etc/alignak/daemons/arbiterd.ini -a /usr/local/etc/alignak/alignak.cfg


Start / stop example scripts
----------------------------

Some scripts to start/stop Alignak daemons are provided with the Alignak source archive.

Those scripts are located in the *dev* directory of the alignak repository. They are delivered as examples to build your own start/stop scripts if you do not use the one provided with an OS installation package.

They are using the *etc/alignak.ini* file to get the Alignak daemons parameters and start the daemons.

The main `_launch_daemon.sh` script is called to launch a daemon which you provide the name. The `launch_arbiter.sh`, `launch_scheduler.sh`, ... scripts are used to start one instance of the each type of daemon. You can start all the daemons at once with the `launch_all.sh` script.

Stopping the launched daemons is possible thanks to the `stop_*.sh` scripts.

You can then start all daemons (as alignak user) like this::

    $launch_all.sh

And stop all daemons::

    $stop_all.sh


Restart to load a new configuration::

    $restart_all.sh


As default:

    - each daemon starts in daemonize mode to be detached from the current shell;
    - the working directory of each daemon is the current working directory. As such, each daemon will create its pid file in the current directory

Specifying the `-d` option will start the daemons in debug mode. Then you will get a log file for each daemon in the current working directory.

Specifying the `-c` option will start the daemons with its own configuration file as defined in *alignak.ini*. In this mode, the daemon will change its working directory according to the values defined in its configuration file. Take care about the defined parameters ;)


.. note :: By default, the arbiter starting script uses the monitoring configuration file defined in the *alignak.ini* file. You can use another configuration file if you set the ``ALIGNAKCFG`` shell environment variable.


.. note :: It is also possible to define a second monitoring configuration file that will be used by the Alignak arbiter. If your configuration is defined in two separated files, you can define the second configuration file if you set the ``ALIGNAKSPECIFICCFG`` shell environment variable.


The `_launch_daemon.sh` script has several command line parameters that may be interesting for more specific usage. When calling one of the `launch*.sh` script you can also use those parameters because they will be forwarded to the `launch_daemon.sh` script.

::

    Usage: ./_launch_daemon.sh [-h|--help] [-v|--version] [-d|--debug] [-a|--arbiter] [-n|--no-daemon] [-V|--verify] daemon_name

        -h (--help)        display this message
        -v (--version)     display alignak version
        -d (--debug)       start requested daemon in debug mode
        -c (--config)      start requested daemon without its configuration file
                           Default is to start with the daemon configuration file
                           This option allow to use the default daemon parameters and the pid and
                           log files are stored in the current working directory
        -r (--replace)     do not replace an existing daemon (if valid pid file exists)
        -n (--no-daemon)   start requested daemon in console mode (do not daemonize)
        -a (--arbiter)     start requested daemon in arbiter mode
                           This option adds the monitoring configuration file(s) on the command line
                           This option will raise an error if the the daemon is not an arbiter.
        -V (--verify)      start requested daemon in verify mode (only for the arbiter)
                           This option will raise an error if the the daemon is not an arbiter.



Alignak.ini configuration file
------------------------------

.. note: This part will be moved to the configuration part of this documentation but, as of now, is stays here for a better understanding of the previously described scripts.

The *etc/alignak.ini* configuration aims to define the main information about how Alignak is installed on the current system.

This file will be located by an OS installation package in the Alignak *etc* directory (eg. */etc/alignak/alignak.ini* or */usr/local/etc/alignak/alignak.ini*). This to allow a third party application or alignak extension to locate it easily. Once parsed this file will contain the necessary information about:

    - the alignak installation directories
    - the alignak daemons and their configuration
    - the alignak monitoring configuration file

This file is structured as an Ini file:

::

    #
    # Copyright (C) 2015-2016: Alignak team, see AUTHORS.txt file for contributors
    #
    # This file is part of Alignak.
    #
    # Alignak is free software: you can redistribute it and/or modify
    # it under the terms of the GNU Affero General Public License as published by
    # the Free Software Foundation, either version 3 of the License, or
    # (at your option) any later version.
    #
    # Alignak is distributed in the hope that it will be useful,
    # but WITHOUT ANY WARRANTY; without even the implied warranty of
    # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    # GNU Affero General Public License for more details.
    #
    # You should have received a copy of the GNU Affero General Public License
    # along with Alignak.  If not, see <http://www.gnu.org/licenses/>.
    #

    #
    # This configuration file is the main Alignak configuration entry point. Each Alignak installer
    # will adapt the content of this file according to the installation process. This will allow
    # any Alignak extension or third party application to find where the Alignak components and
    # files are located on the system.
    #
    # ---
    # This version of the file contains variable that are suitable to run a single node Alignak
    # with all its daemon using the default configuration existing in the repository.
    #

    # Main alignak variables:
    # - BIN is where the launch scripts are located
    #   (Debian sets to /usr/bin)
    # - ETC is where we store the configuration files
    #   (Debian sets to /etc/alignak)
    # - VAR is where the libraries and plugins files are installed
    #   (Debian sets to /var/lib/alignak)
    # - RUN is the daemons working directory and where pid files are stored
    #   (Debian sets to /var/run/alignak)
    # - LOG is where we put log files
    #   (Debian sets to /var/log/alignak)
    #
    [DEFAULT]
    BIN=../alignak/bin
    ETC=../etc
    VAR=/tmp
    RUN=/tmp
    LOG=/tmp


    # We define the name of the 2 main Alignak configuration files.
    # There may be 2 configuration files because tools like Centreon generate those...
    [alignak-configuration]
    # Alignak main configuration file
    CFG=%(ETC)s/alignak.cfg
    # Alignak secondary configuration file (none as a default)
    SPECIFICCFG=


    # For each Alignak daemon, this file contains a section with the daemon name. The section
    # identifier is the corresponding daemon name. This daemon name is built with the daemon
    # type (eg. arbiter, poller,...) and the daemon name separated with a dash.
    # This rule ensure that alignak will be able to find all the daemons configuration in this
    # whatever the number of daemons existing in the configuration
    #
    # Each section defines:
    # - the location of the daemon configuration file
    # - the daemon launching script
    # - the location of the daemon pid file
    # - the location of the daemon debug log file (if any is to be used)

    [arbiter-master]
    ### ARBITER PART ###
    PROCESS=alignak-arbiter
    DAEMON=%(BIN)s/alignak_arbiter.py
    CFG=%(ETC)s/daemons/arbiterd.ini
    DEBUGFILE=%(LOG)s/arbiter-debug.log


    [scheduler-master]
    ### SCHEDULER PART ###
    PROCESS=alignak-scheduler
    DAEMON=%(BIN)s/alignak_scheduler.py
    CFG=%(ETC)s/daemons/schedulerd.ini
    DEBUGFILE=%(LOG)s/scheduler-debug.log

    [poller-master]
    ### POLLER PART ###
    PROCESS=alignak-poller
    DAEMON=%(BIN)s/alignak_poller.py
    CFG=%(ETC)s/daemons/pollerd.ini
    DEBUGFILE=%(LOG)s/poller-debug.log

    [reactionner-master]
    ### REACTIONNER PART ###
    PROCESS=alignak-reactionner
    DAEMON=%(BIN)s/alignak_reactionner.py
    CFG=%(ETC)s/daemons/reactionnerd.ini
    DEBUGFILE=%(LOG)s/reactionner-debug.log

    [broker-master]
    ### BROKER PART ###
    PROCESS=alignak-broker
    DAEMON=%(BIN)s/alignak_broker.py
    CFG=%(ETC)s/daemons/brokerd.ini
    DEBUGFILE=%(LOG)s/broker-debug.log

    [receiver-master]
    ### RECEIVER PART ###
    PROCESS=alignak-receiver
    DAEMON=%(BIN)s/alignak_receiver.py
    CFG=%(ETC)s/daemons/receiverd.ini
    DEBUGFILE=%(LOG)s/receiver-debug.log



.. note :: in future version, the role of this file will be extended to contain all the daemons configuration in place of each file used for each daemon.

Environment variables
=====================

Alignak uses some environment variables


Alignak internal metrics
------------------------

If some environment variables exist the Alignak internal metrics will be logged to a file in append mode:
    ``ALIGNAK_STATS_FILE``
        the file name

    ``ALIGNAK_STATS_FILE_LINE_FMT``
        defaults to [#date#] #counter# #value# #uom#\n'

    ``ALIGNAK_STATS_FILE_DATE_FMT``
        defaults to '%Y-%m-%d %H:%M:%S'
        date is UTC
        if configured as an empty string, the date will be output as a UTC timestamp


Log system health
-----------------

Defining the ``TEST_LOG_MONITORING`` environment variable will make Alignak add some log in the scheduler daemons log files to inform about the system CPU, memory and disk consumption.

On each scheduling loop end, if the report period ia happening, the Alignak scheduler gets the current cpu, memory and disk information from the OS and dumps them to the information log. The dump is formatted as a Nagios plugin output with performance data.

When this variable is defined, the default report period is set to 5. As such, each 5 scheduling loop, there is a report in the information log. If this variable contains an integer value, this value will define the report period.
::

   # Define environment variable
   setenv TEST_LOG_MONITORING 5


   [2017-09-19 15:54:36 CEST] INFO: [alignak.scheduler] Scheduler scheduler-master cpu|'cpu_count'=4 'cpu_1_percent'=42.20% 'cpu_2_percent'=38.40% 'cpu_3_percent'=35.40% 'cpu_4_percent'=48.10% 'cpu_1_user_percent'=37.90% 'cpu_1_nice_percent'=0.00% 'cpu_1_system_percent'=4.20% 'cpu_1_idle_percent'=57.80% 'cpu_1_irq_percent'=0.00% 'cpu_2_user_percent'=31.80% 'cpu_2_nice_percent'=0.00% 'cpu_2_system_percent'=6.10% 'cpu_2_idle_percent'=61.60% 'cpu_2_irq_percent'=0.50% 'cpu_3_user_percent'=31.00% 'cpu_3_nice_percent'=0.00% 'cpu_3_system_percent'=4.20% 'cpu_3_idle_percent'=64.60% 'cpu_3_irq_percent'=0.20% 'cpu_4_user_percent'=38.90% 'cpu_4_nice_percent'=0.00% 'cpu_4_system_percent'=9.20% 'cpu_4_idle_percent'=51.90% 'cpu_4_irq_percent'=0.00%
   [2017-09-19 15:54:36 CEST] INFO: [alignak.scheduler] Scheduler scheduler-master disks|'disk_/_total'=952725065728B 'disk_/_used'=93761236992B 'disk_/_free'=858963828736B 'disk_/_percent_used'=9.80%
   [2017-09-19 15:54:36 CEST] INFO: [alignak.scheduler] Scheduler scheduler-master memory|'swap_total'=2621424B 'swap_used'=33514B 'swap_free'=2587910B 'swap_used_percent'=1.30% 'swap_sin'=2687B 'swap_sout'=12851708B
   [2017-09-19 15:54:41 CEST] INFO: [alignak.scheduler] Scheduler scheduler-master cpu|'cpu_count'=4 'cpu_1_percent'=34.00% 'cpu_2_percent'=37.40% 'cpu_3_percent'=36.10% 'cpu_4_percent'=25.10% 'cpu_1_user_percent'=26.90% 'cpu_1_nice_percent'=0.00% 'cpu_1_system_percent'=7.00% 'cpu_1_idle_percent'=66.00% 'cpu_1_irq_percent'=0.00% 'cpu_2_user_percent'=30.10% 'cpu_2_nice_percent'=0.00% 'cpu_2_system_percent'=7.20% 'cpu_2_idle_percent'=62.60% 'cpu_2_irq_percent'=0.20% 'cpu_3_user_percent'=30.40% 'cpu_3_nice_percent'=0.00% 'cpu_3_system_percent'=5.60% 'cpu_3_idle_percent'=63.90% 'cpu_3_irq_percent'=0.20% 'cpu_4_user_percent'=19.20% 'cpu_4_nice_percent'=0.00% 'cpu_4_system_percent'=5.80% 'cpu_4_idle_percent'=74.90% 'cpu_4_irq_percent'=0.20%
   [2017-09-19 15:54:41 CEST] INFO: [alignak.scheduler] Scheduler scheduler-master disks|'disk_/_total'=952725061632B 'disk_/_used'=93761646592B 'disk_/_free'=858963415040B 'disk_/_percent_used'=9.80%
   [2017-09-19 15:54:41 CEST] INFO: [alignak.scheduler] Scheduler scheduler-master memory|'swap_total'=2621424B 'swap_used'=33514B 'swap_free'=2587910B 'swap_used_percent'=1.30% 'swap_sin'=2687B 'swap_sout'=12851710B
   [2017-09-19 15:54:46 CEST] INFO: [alignak.scheduler] Scheduler scheduler-master cpu|'cpu_count'=4 'cpu_1_percent'=28.70% 'cpu_2_percent'=24.60% 'cpu_3_percent'=36.40% 'cpu_4_percent'=41.00% 'cpu_1_user_percent'=21.20% 'cpu_1_nice_percent'=0.00% 'cpu_1_system_percent'=7.50% 'cpu_1_idle_percent'=71.30% 'cpu_1_irq_percent'=0.00% 'cpu_2_user_percent'=17.70% 'cpu_2_nice_percent'=0.00% 'cpu_2_system_percent'=6.80% 'cpu_2_idle_percent'=75.40% 'cpu_2_irq_percent'=0.20% 'cpu_3_user_percent'=27.90% 'cpu_3_nice_percent'=0.00% 'cpu_3_system_percent'=8.20% 'cpu_3_idle_percent'=63.60% 'cpu_3_irq_percent'=0.30% 'cpu_4_user_percent'=33.60% 'cpu_4_nice_percent'=0.00% 'cpu_4_system_percent'=7.10% 'cpu_4_idle_percent'=59.00% 'cpu_4_irq_percent'=0.30%
   [2017-09-19 15:54:46 CEST] INFO: [alignak.scheduler] Scheduler scheduler-master disks|'disk_/_total'=952725045248B 'disk_/_used'=93762039808B 'disk_/_free'=858963005440B 'disk_/_percent_used'=9.80%
   [2017-09-19 15:54:46 CEST] INFO: [alignak.scheduler] Scheduler scheduler-master memory|'swap_total'=2621424B 'swap_used'=33514B 'swap_free'=2587910B 'swap_used_percent'=1.30% 'swap_sin'=2687B 'swap_sout'=12851716B


.. note :: this feature allows to have some information about the system load with a running Alignak scheduler.

Log Scheduling loop
-------------------

Defining the ``TEST_LOG_LOOP`` environment variable will make Alignak add some log in the scheduler daemons log files to inform about the checks that are scheduled.

As an example:
::

    # Define environment variable
    setenv TEST_LOG_LOOP 1

    # Start Alignak daemons

    # Tail scheduler log files
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] --- 64
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] Items (loop): broks: 0, notifications: 0, checks: 0, internal checks: 0, event handlers: 0, external commands: 0
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] Items (total): broks: 52, notifications: 0, checks: 13, internal checks: 0, event handlers: 0, external commands: 0
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] Actions 'eventhandler/total': launched: 0, timeout: 0, executed: 0
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] Results 'eventhandler/total': total: 0,
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] Actions 'eventhandler/loop': launched: 0, timeout: 0, executed: 0
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] Results 'eventhandler/loop': total: 0,
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] Actions 'notification/total': launched: 0, timeout: 0, executed: 0
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] Results 'notification/total': total: 0,
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] Actions 'notification/loop': launched: 0, timeout: 0, executed: 0
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] Results 'notification/loop': total: 0,
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] Actions 'check/total': launched: 2, timeout: 0, executed: 2
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] Results 'check/total': total: 4, done: 4,
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] Actions 'check/loop': launched: 0, timeout: 0, executed: 0
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] Results 'check/loop': total: 2, done: 2,
    [2017-05-27 07:32:49 CEST] INFO: [alignak.scheduler] Checks (loop): total: 12 (scheduled: 11, launched: 0, in poller: 0, timeout: 0, done: 0, zombies: 0)
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Elapsed time, current loop: 0.00, from start: 63.20 (64 loops)
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Check average (loop) = 0 checks results, 0.00 checks/s
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Check average (total) = 13 checks results, 0.21 checks/s
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] +++ 64
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] --- 65
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Items (loop): broks: 0, notifications: 0, checks: 0, internal checks: 0, event handlers: 0, external commands: 0
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Items (total): broks: 52, notifications: 0, checks: 13, internal checks: 0, event handlers: 0, external commands: 0
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Actions 'eventhandler/total': launched: 0, timeout: 0, executed: 0
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Results 'eventhandler/total': total: 0,
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Actions 'eventhandler/loop': launched: 0, timeout: 0, executed: 0
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Results 'eventhandler/loop': total: 0,
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Actions 'notification/total': launched: 0, timeout: 0, executed: 0
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Results 'notification/total': total: 0,
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Actions 'notification/loop': launched: 0, timeout: 0, executed: 0
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Results 'notification/loop': total: 0,
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Actions 'check/total': launched: 2, timeout: 0, executed: 2
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Results 'check/total': total: 4, done: 4,
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Actions 'check/loop': launched: 0, timeout: 0, executed: 0
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Results 'check/loop': total: 2, done: 2,
    [2017-05-27 07:32:50 CEST] INFO: [alignak.scheduler] Checks (loop): total: 12 (scheduled: 11, launched: 0, in poller: 0, timeout: 0, done: 0, zombies: 0)
    [2017-05-27 07:32:51 CEST] INFO: [alignak.scheduler] Elapsed time, current loop: 0.01, from start: 64.21 (65 loops)
    [2017-05-27 07:32:51 CEST] INFO: [alignak.scheduler] Check average (loop) = 0 checks results, 0.00 checks/s
    [2017-05-27 07:32:51 CEST] INFO: [alignak.scheduler] Check average (total) = 13 checks results, 0.20 checks/s
    [2017-05-27 07:32:51 CEST] INFO: [alignak.scheduler] +++ 65


Log Alignak actions
-------------------

Defining the ``TEST_LOG_ACTIONS`` environment variable will make Alignak add some information in its daemons log files to inform about the commands that are launched for the checks and the notifications. This is very useful to help setting-up the checks because the launched checks and their results are available as INFO log

If this variable is set to 'WARNING', the logs will be at the WARNING level, else INFO.

As an example:
::

    # Define environment variable
    setenv TEST_LOG_ACTIONS 1

    # Start Alignak daemons

    # Tail log files
    ==> /usr/local/var/log/alignak/pollerd.log <==
    [2017-04-26 16:23:57 UTC] INFO: [alignak.action] Launch command: /usr/local/libexec/nagios/check_nrpe -H 93.93.47.81 -t 10 -u -n -c check_zombie_procs
    [2017-04-26 16:23:57 UTC] INFO: [alignak.action] Check for /usr/local/libexec/nagios/check_nrpe -H 93.93.47.81 -t 10 -u -n -c check_zombie_procs exited with return code 0
    [2017-04-26 16:23:57 UTC] INFO: [alignak.action] Check result for /usr/local/libexec/nagios/check_nrpe -H 93.93.47.81 -t 10 -u -n -c check_zombie_procs: 0, PROCS OK: 0 processes with STATE = Z
    [2017-04-26 16:23:57 UTC] INFO: [alignak.action] Performance data for /usr/local/libexec/nagios/check_nrpe -H 93.93.47.81 -t 10 -u -n -c check_zombie_procs: procs=0;5;10;0;


Log Alignak alerts and notifications
------------------------------------

Defining the ``TEST_LOG_ALERTS`` ``TEST_LOG_NOTIFICATIONS`` environment variables will make Alignak add some information in its daemons log files to inform about the alerts and notifications that are raised for the monitored hosts and services.

If these variables are set to 'WARNING', the logs will be at the WARNING level, else INFO.


Alignak processes list
======================

The daemons involved in Alignak are starting several processes in the system. All the processes started have a process title set by Alignak to help the user knowing which is which. Several processes types are present in the system processes list:

    * the main daemon process
        There will always be one process for each Alignak daemon type. The process title is the daemon type (eg. *alignak-arbiter*, *alignak-scheduler*,...)

    * the main daemon forked process.
        Each Alignak daemon forks a new process instance for each daemon instance existing in the configuration. If you defined several schedulers you will get a process for each scheduler instance. Each daemon instance process has a title built with the instance name (eg. *alignak-scheduler scheduler-master*)

    * the external modules processes
        The daemons that have some external modules attached, like the brokers or receivers, launch new processes for their modules. Those processes titles are made of the daemon instance name and the module alias (eg. *alignak-receiver-master module: nsca*)

    * the satellite workers processes
        The satellites daemons that need some worker processes (pollers and reactionners) launch several worker processes to execute their actions (checks or notifications). Those worker processes have a title made of the daemon instance name and the worker label (eg. *alignak-poller-master worker*)


 As an example, here is the processes list of an Alignak "simple" configuration with no spare daemons and no distributed configuration::

    alignak   5850  0.7  1.0 867048 43148 ?        Sl   10:54   0:00 alignak-scheduler scheduler-master
    alignak   5851  0.0  0.9 208644 37076 ?        S    10:54   0:00 alignak-scheduler
    alignak   5907  0.4  1.0 865080 42516 ?        Sl   10:54   0:00 alignak-poller poller-master
    alignak   5908  0.0  0.9 495000 37964 ?        Sl   10:54   0:00 alignak-poller
    alignak   5968  0.4  1.0 864756 42456 ?        Sl   10:54   0:00 alignak-reactionner reactionner-master
    alignak   5973  0.0  0.9 421272 38044 ?        Sl   10:54   0:00 alignak-reactionner
    alignak   6078  1.2  1.1 867732 45072 ?        Sl   10:55   0:00 alignak-broker broker-master
    alignak   6079  0.1  0.9 495276 40048 ?        Sl   10:55   0:00 alignak-broker
    alignak   6153  0.4  1.0 864576 42036 ?        Sl   10:55   0:00 alignak-receiver receiver-master
    alignak   6154  0.0  0.9 347940 37736 ?        Sl   10:55   0:00 alignak-receiver
    alignak   6216  1.6  1.1 867588 44528 ?        Sl   10:55   0:00 alignak-arbiter arbiter-master
    alignak   6217  0.0  0.9 211000 39376 ?        S    10:55   0:00 alignak-arbiter
    alignak   6230  0.0  0.9 864184 40452 ?        S    10:55   0:00 alignak-poller-master worker
    alignak   6240  0.0  1.0 864320 40960 ?        S    10:55   0:00 alignak-receiver-master module: nsca
    alignak   6250  0.2  1.0 866748 43228 ?        S    10:55   0:00 alignak-broker-master module: backend_broker
    alignak   6260  0.2  1.0 866748 43072 ?        S    10:55   0:00 alignak-broker-master module: logs
    alignak   6271  0.0  1.0 864196 40592 ?        S    10:55   0:00 alignak-poller-master worker
    alignak   6279  0.0  1.0 864188 40544 ?        S    10:55   0:00 alignak-reactionner-master worker


Log files
=========

When running, the Alignak daemons are logging their activity in log files that can be found in the
*/usr/local/var/log/* directory. Each daemon has its own log file. Log files are kept on the system
for a default period of 7 rotating days.

Each daemon log file configuration is found in the daemon configuration file (/usr/local/etc/alignak/daemons/*.ini*).

In case of problem, make sure that there is no ERROR and/or WARNING logs in the log files.

The log files are the very first information source about Alignak activity. You will find:

    * HOST ALERT information
    * SERVICE ALERT information
    * ...

to keep you informed about your system state.

As an example, the *schedulerd.log* file some few minutes after start::

    [1474548490] INFO: [Alignak] Loading configuration.
    [1474548490] INFO: [Alignak] New configuration loaded
    [1474548490] INFO: [Alignak] [scheduler-master] First scheduling launched
    [1474548490] INFO: [Alignak] [scheduler-master] First scheduling done
    [1474548490] INFO: [Alignak] A new broker just connected : broker-master
    [1474548490] INFO: [Alignak] [scheduler-master] Created 38 initial Broks for broker broker-master
    [1474548530] HOST ALERT: host_snmp;DOWN;SOFT;1;Alarm timeout
    [1474548581] SERVICE ALERT: host_snmp;Disks;CRITICAL;SOFT;1;CRITICAL : (>95%) Cached memory: 100%used(189MB/189MB) Physical memory: 95%used(1892MB/2000MB) Shared memory: 100%used(23MB/23MB)
    [1474548602] HOST ALERT: host_snmp;DOWN;SOFT;1;Alarm timeout
    [1474548614] SERVICE ALERT: host_snmp;Memory;WARNING;SOFT;1;Ram : 85%, Swap : 54% : > 80, 80 ; WARNING
    [1474548637] HOST ALERT: host_snmp;DOWN;SOFT;1;Alarm timeout
    [1474548662] SERVICE ALERT: host_snmp;NetworkUsage;UNKNOWN;SOFT;1;ERROR : Unknown interface eth\d+
    [1474548683] HOST ALERT: host_snmp;DOWN;SOFT;1;Alarm timeout
    [1474548700] SERVICE ALERT: host_snmp;Disks;CRITICAL;SOFT;2;CRITICAL : (>95%) Cached memory: 100%used(193MB/193MB) Physical memory: 96%used(1921MB/2000MB) Shared memory: 100%used(23MB/23MB)
    [1474548722] HOST ALERT: host_snmp;DOWN;SOFT;1;Alarm timeout
    [1474548734] SERVICE ALERT: host_snmp;Memory;WARNING;SOFT;2;Ram : 86%, Swap : 54% : > 80, 80 ; WARNING
    [1474548757] HOST ALERT: host_snmp;DOWN;SOFT;1;Alarm timeout
    [1474548783] SERVICE ALERT: host_snmp;NetworkUsage;UNKNOWN;SOFT;2;ERROR : Unknown interface eth\d+
    [1474548805] HOST ALERT: host_snmp;DOWN;SOFT;1;Alarm timeout
    [1474548819] SERVICE ALERT: host_snmp;Disks;CRITICAL;HARD;3;CRITICAL : (>95%) Cached memory: 100%used(193MB/193MB) Physical memory: 96%used(1930MB/2000MB) Shared memory: 100%used(23MB/23MB)
    [1474548829] HOST ALERT: host_snmp;DOWN;HARD;2;Alarm timeout
    [1474548829] HOST NOTIFICATION: admin;host_snmp;DOWN;notify-host-by-email;Alarm timeout
    [1474548854] SERVICE ALERT: host_snmp;Memory;WARNING;HARD;3;Ram : 86%, Swap : 54% : > 80, 80 ; WARNING
    [1474548902] SERVICE ALERT: host_snmp;NetworkUsage;UNKNOWN;HARD;3;ERROR : Unknown interface eth\d+

