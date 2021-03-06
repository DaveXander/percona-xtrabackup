================================================================================
 Partial Backups
================================================================================

|xtrabackup| supports taking partial backups when the
:term:`innodb_file_per_table` option is enabled. There are three ways to create
partial backups: matching the tables' names with a regular expression, providing
a list of them in a file or providing a list of databases.

.. warning::

   If any of the matched or listed tables is deleted during the
   backup, |xtrabackup| will fail.

For the purposes of this manual page, we will assume that there is a database
named ``test`` which contains tables named ``t1`` and ``t2``.

Using :option:`xtrabackup --tables`
================================================================================

The first method is with the :option:`xtrabackup --tables`. The option's value
is a regular expression that is matched against the fully qualified tablename,
including the database name, in the form ``databasename.tablename``.

To back up only tables in the ``test`` database, you can use the
following command: ::

.. code-block:: bash

  $ xtrabackup --backup --datadir=/var/lib/mysql --target-dir=/data/backups/ \
  --tables="^test[.].*"

To back up only the table ``test.t1``, you can use the following command: ::

.. code-block:: bash

   $ xtrabackup --backup --datadir=/var/lib/mysql --target-dir=/data/backups/ \
   --tables="^test[.]t1"

Using :option:`xtrabackup --tables-file`
================================================================================

:option:`xtrabackup --tables-file` specifies a file that can contain multiple
table names, one table name per line in the file. Only the tables named in the
file will be backed up. Names are matched exactly, case-sensitive, with no
pattern or regular expression matching. The table names must be fully qualified,
in ``databasename.tablename`` format.

.. code-block:: bash

  $ echo "mydatabase.mytable" > /tmp/tables.txt
  $ xtrabackup --backup --tables-file=/tmp/tables.txt 

Using :option:`xtrabackup --databases` and :option:`xtrabackup --databases-file`
================================================================================

:option:`xtrabackup --databases` accepts a space-separated list of the databases
and tables to backup in the ``databasename[.tablename]`` form.

:option:`xtrabackup --databases-file` specifies a file that can contain multiple
databases and tables in the ``databasename[.tablename]`` form, one element name
per line in the file. Only named databases and tables will be backed up. Names
are matched exactly, case-sensitive, with no pattern or regular expression
matching.

Preparing the Backup
================================================================================

When you use the :option:`xtrabackup --prepare` option on a partial backup, you
will see warnings about tables that don't exist. That is because these tables
exist in the data dictionary inside InnoDB, but the corresponding :term:`.ibd`
files don't exist. They were not copied into the backup directory. These tables
will be removed from the data dictionary, and when you restore the backup and
start InnoDB, they will no longer exist and will not cause any errors or
warnings to be printed to the log file.

An example of the error message you will see during the prepare phase
follows. ::

  InnoDB: Reading tablespace information from the .ibd files...
  101107 22:31:30  InnoDB: Error: table 'test1/t'
  InnoDB: in InnoDB data dictionary has tablespace id 6,
  InnoDB: but tablespace with that id or name does not exist. It will be removed from data dictionary.

