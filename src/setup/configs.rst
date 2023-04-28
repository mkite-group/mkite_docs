=================
Configuring mkite
=================

Configuration files
-------------------

Now that mkite is installed and the database/engine is up, we can combine all configuration files for mkite in a single folder. We provide a template for this folder at https://github.com/mkite-group/configs_template, which we will use to fill in the information when setting up a new server.

.. tip::

   While you do not have to create Git repositories for your own configuration files, it is highly recommended to do so. Keeping your files backed up and with version control will greatly help configuring and managing files across heterogeneous computing resources.

First, clone the configs template or create your own blank directory. 
One suggestion is to create one directory for ``mkite_db`` configurations, one for ``mkite_engines``, and another one for ``mkwind``. 
However, you may choose the names and file structure you desire.
An example of configuration file structure is:

.. code-block:: bash

    .
    ├── clusters
    │   ├── cluster1.yaml
    │   ├── cluster2.yaml
    │   └── local.yaml
    ├── db
    │   ├── local.env
    │   └── cloud.env
    ├── engines
    │   ├── global.yaml
    │   ├── archive.yaml
    │   ├── redis-cloud.yaml
    │   └── redis-local.yaml
    └── mkwind
        ├── cluster1.yaml
        ├── cluster2.yaml
        └── local.yaml

Configurations for mkite usually rely on two things:

- Environmental variables, used minimally. Needed for finding database environment files or setting up commands in third-party packages (e.g., where to find your executable when running a simulation with ASE).
- Configuration files, usually in the form of YAML or JSON files. These files should be kept in a private location, but can still be used with version control and (secure) remote repositories.

.. figure:: _img/configs.svg
    :align: center
    :alt: illustration of configuration files for mkite
    :class: figtextwidth
    :width: 80%

Below, we will set up the main credentials for all relevant parts of mkite and mkwind.

Settings for mkite
------------------

Adding database credentials
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Inside the ``mkite_db`` folder (or however you want to call it), you can create a ``.env`` file to setup your database credentials. For example, you can create a file called ``production.env`` to contain the credentials for your production database:

.. code-block:: bash

    DEBUG=False
    SECRET_KEY='your_django_secret_key'
    DATABASE_URL="postgres://username:password@host:port/dbname"

In the example above, you have to replace the database credentials (username, password, host, port, and database name) by your own credentials. In `our previous example <postgres>`_, we created a PostgreSQL database in the localhost (port usually 5432) named mydb:

.. code-block:: bash

    DATABASE_URL="postgres://username:password@localhost:5432/mydb"

Furthermore, the ``SECRET_KEY`` is used by Django for cryptographic signing. You can read more about it `on the official Django documentation <https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SECRET_KEY>`_. Generating a new secret key is straightforward. With your Python environment activated, run the following command:

.. code-block:: bash

    python3 -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'

Finally, you will need to let `mkite_db` know where to find these files. To do so, you will have to set the environmental variable ``MKITE_ENV``. You can add to your ``~/.profile`` or similar dotfile:

.. code-block:: bash

   export MKITE_ENV="/path/to/production.env"

With this, mkite should be able to access your database.

Adding Redis engine
^^^^^^^^^^^^^^^^^^^

In addition to the PostgreSQL database, you will have to write down the credentials to the Redis engine. Inside the folder that contains the credentials for ``mkite_engines``, you can create a YAML file with the hostname, port, and password for the Redis database. For example:

.. code-block:: yaml

    _module: mkite_engines.redis
    host: redis.cloud.server.com
    port: 1234
    password: your_password

.. attention::

   In the engine files, you have to specify which module it is going to use.
   In the case above, it is the ``mkite_engines.redis`` module, which loads the engines for Redis.
   If not specified, the package will not know how to instantiate an engine.

.. note::
    You can add any additional arguments that are going to be passed to the `redis Python package <https://github.com/redis/redis-py>`_, such as ``ssl``, ``ssl_cert_reqs`` etc.

Adding local engine
^^^^^^^^^^^^^^^^^^^

Several operations in mkite and mkwind require local file transfer.
For example, the local job submission (``mkwind run``) uses a folder in the filesystem where to store jobs in execution.
Similarly, the archiving procedure moves completed raw files to a storage folder.
These operations require pointing to a local folder in the filesystem where the files will be moved to.

Specifying a local engine with mkite configuration files is simple.
Create a YAML file named ``local.yaml`` (or any other name) containing the following contents:

.. code-block:: yaml

    _module: mkite_engines.local
    root_path: $HOME/jobs
    move: True

This will create a folder at ``$HOME/jobs`` (environmental variable expanded to the user home).

Settings for mkwind
-------------------

Configuration files for ``mkwind`` involve two separate types of configurations:

- General settings: these configuration files specify which scheduler does the HPC system use, where to archive the files, where to log etc. They are general settings for each computing environment, and typically have to be changed less frequently.
- Build settings: these files provide details of how to build each new job. This is required to build jobs in each filesystem according to local specifications, such as: number of CPUs/GPUs to use; which account to charge; walltime for each recipe; which commands to run; and so on.

In this part, we will set up these two types of configurations assuming a local host, and another host running SLURM.

General configurations
^^^^^^^^^^^^^^^^^^^^^^

Local configurations specify general aspects of the mkwind client. Inside the folder dedicated to ``mkwind``, you can create a YAML file named ``local.yaml`` following the template below:

.. code-block:: yaml

   MAX_PENDING: 10
   MAX_RUNNING: 100
   MAX_READY: 10
   SCHEDULER: pueue
   BUILD_CONFIG: ${_self_}/../clusters/cluster1.yaml
   ENGINE_EXTERNAL: ${_self_}/../engines/redis-cloud.yaml
   ENGINE_LOCAL: ${_self_}/../engines/local.yaml
   ENGINE_ARCHIVE: ${_self_}/../engines/archive.yaml
   LOG_PATH: /home/user/logs

In the configuration above:

- The ``mkwind`` runner daemon will keep at most 100 jobs running and 10 pending using the `pueue <https://github.com/Nukesor/pueue>`_ scheduler. 
- At most 10 jobs will be built at every cycle of the builder daemon.
- The ``mkwind build`` daemon will use the ``BUILD_CONFIG`` file when building the new jobs (see next section)
- The engine to be used for interacting with the external world is specified by `redis-cloud.yaml`
- The engine to be used for managing local jobs (i.e., file transfer for a ``run`` daemon) is described by `local.yaml`
- The engine to be used for archiving jobs (i.e., tarfiles of raw files) is described by `archive.yaml`

Different specifications can be provided depending on the client. For example, the scheduler could be ``slurm`` and the log path could be different than the home directory.

.. note::

   To facilitate the file structure, environmental variables in the YAML file are expanded based on your environment.
   This means that strings such as ``$ENV`` and ``${ENV}`` are replaced by the value of ``ENV`` if you have it in your environment.
   Furthermore, the variable ${_self_} is mapped to the directory where the config file is.
   With this shortcut, it is easier to create the config file without absolute paths.

Building configurations
^^^^^^^^^^^^^^^^^^^^^^^

Building configurations require a bit more careful deliberation. For each client where jobs will be built, configurations have to be provided. One example is:

.. code-block:: yaml

    default:
      nodes: 1
      tasks_per_node: 36
      walltime: 24:00:00
      partition: pbatch
      account: acct
      pre_cmd: |
        source /home/user/.bashrc
        source /home/user/envs/mkite/bin/activate
      cmd: kite run
      post_cmd: |
        touch mkwind-complete

    vasp:
      account: vasp_acct
      pre_cmd: |
        source /home/user/.bashrc
        source /home/user/envs/mkite/bin/activate
        module load vasp

    conformer.generation:
      nodes: 1
      tasks_per_node: 1
      walltime: 30:00
      partition: pdebug
      cmd: kite run

In the configuration above, jobs will be run with the default settings (1 node, 36 tasks/node etc). However, depending on the recipe, it is possible to specify different accounts and partitions. For example, recipes starting with ``vasp`` will use the ``vasp_acct`` account instead of the default ``acct``, but still use 36 tasks/node. On the other hand, the ``conformer.generation`` recipe will use only one node, one task per node, and 30 min of walltime on the ``pdebug`` partition.

For a complete description of all settings and overwriting defaults, please check the advanced guide.
