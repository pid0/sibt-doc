sibt – A configurable interface to existing backup tools.
=========================================================

SYNOPSIS
----------

::

  sibt [options...] list all|rules|schedulers|synchronizers

  sibt [options...] show|check|schedule <rule-patterns...>

  sibt [options...] versions-of <file>

  sibt [options...] restore [--to <destination>] <file>
    <version-substrings>

  sibt [options...] list-files [--null] [--recursive] <file>
    <version-substrings...>

  sibt --version

DESCRIPTION
--------------
sibt (simple backup tool) is a Python program that strives to integrate many
well-established backup tools by providing a common configuration format and
command line interface. It works with the notion of a :py:class:`SyncRule`.
For example, if you want to schedule regular backups of your entire system,
which you decide should be compressed and then sent to a remote server, you
define one of these rules.

.. py:class:: SyncRule

  .. py:attribute:: Synchronizer
    
    A synchronizer is an external program that copies data from locations to other locations
    so if files are lost at one of them, it can be restored from the others.
    Currently available synchronizers are:

      * rsync
      * rdiff-backup
      * tar
      * duplicity

  .. py:attribute:: Scheduler
    
     A scheduler is responsible for deciding when to execute a rule and for preparing the environment
     a synchronizer will run in, like ensuring that e.g., rsync will never be run twice with the same arguments in parallel.
     Currently available schedulers are:
      
      * anacron
      * A simple interval-based scheduler

  .. py:attribute:: Loc1

     A location is a local file path or a remote URL that the synchronizer can
     work with.
     The number of required locations is given by type of operation performed by
     the synchronizer.
     For example, rsync needs two locations and copies from the first to the second.

  .. py:attribute:: Loc2


Rules are declared as files with an ini-like syntax. The example could be defined with the following file::

  #import TarFullSystemBackup

  _destination = pcbackup@server:/mnt/backup
  _excludeAlso = /var/local

  [Scheduler]
  Name = anacron
  Interval = 3 days

  [Synchronizer]
  RemoteShellCommand = ssh -i /root/pc-backup/id_ed25519

Advantages over custom shell scripts run with cron.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
 * Extensive test suite ensuring that backup and restore operations work correctly in all
   circumstances.
 * Minimally surprising command line interface. For example, sibt restore has the same
   semantics as the traditional UNIX cp command. It is not necessary to remember syntactic quirks
   like the meaning of trailing slashes in an rsync command line.
 * sibt still does not eve* touch the contents of any of your files itself but
   relies on the synchronizers instead which, like rsync, have a 
 * Swapping out backup programs is relatively easy due to the common interface.
 * Sibt performs many checks to make sure backup rules do what a user is likely
   to expect from them. For instance, two rules are never allowed to write to the same
   location, or a rule may not write to a location another one reads. This works also
   for remote URLs. Configuration values have data types that are syntax checked.
 * Sibt leaves it to the scheduler to actually execute a rule but it replicates
   critical pieces of infrastructure like logging and employs a locking mechanism
   to prevent rules from being run twice.
 * It is a stated design goal to keep metadata, data, and logs in an uncorrupted state even after a power outage.

Advantages over database-based solutions.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
 
 * Very lightweight. Extensible with dependencies.
 * Simple to set up as an unprivileged user as well as root. A full system backup
   configuration is only a few lines long.
 * Possible to write backups to a variety of cloud servers using duplicity.
 * rsync and rdiff-backup mirror their locations which means the backup requires no
   special software to access in these cases. GNU Tar is also an extremely stable format.

FILES
---------
  
**/etc/sibt/rules** For system-wide rule declarations.

**/var/lib/sibt**

**~/.sibt/config/rules** For user-specific rule declarations.

**~/.sibt/var**

AUTHOR
----------
Patrick Plagwitz <patrick_plagwitz@web.de>
