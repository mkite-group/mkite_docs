=========================
Installing mkite packages
=========================

Core package
------------

The most basic ``mkite`` package that should be installed is ``mkite_core``. Inside the newly-created environment, you can install this package by running

.. code-block:: bash

   pip install mkite_core

The ``mkite_core`` will provide most functions to execute a job in any given client. Nevertheless, advanced job management in clients would often require the installation of two additional mkite packages: ``mkite_engines`` and ``mkwind`` to manage the jobs. You can install them using

Client/engines
--------------

.. code-block:: bash

   pip install mkite_engines mkwind

Plugins
-------

If you are setting up a client, you can install only the desired plugins that will be used for executing jobs in that particular client. For example:

.. code-block:: bash

   pip install mkite_conformer

Database interface
------------------

Finally, if you are setting up a server, you will have to install the main database and its associated dependencies:

.. code-block:: bash

   pip install mkite_db

If you are setting up a client only (say, for running jobs in an HPC environment), you do not need to install the ``mkite_db`` package.
