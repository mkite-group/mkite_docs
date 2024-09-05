================================
Setting up a PostgreSQL database
================================

This part is only required if you are going to be setting up a server (i.e., the main database). If you are setting up only the client, you can skip this step.

Installing PostgreSQL
---------------------

.. tab:: Linux

    On Debian-based systems (e.g., Ubuntu), you can install PostgreSQL with:

    .. code-block:: bash

        sudo apt-get update
        sudo apt-get install postgresql postgresql-contrib

    For other systems, please check the `original PostgreSQL website <https://www.postgresql.org/download/linux/>`_ for instructions.

.. tab:: macOS

    In your terminal, use brew to install the PostgreSQL database:

    .. code-block:: bash

        brew install postgresql

Starting the PostgreSQL database
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. tab:: Linux

    You can start the PostgreSQL service by running:

    .. code-block:: bash

        sudo systemctl enable postgresql
        sudo systemctl start postgresql

    If you do not have ``sudo`` priviledges in your system, you can start the database locally by following the `instructions on PostgreSQL <https://www.postgresql.org/docs/current/server-start.html>`_.

.. tab:: macOS

    To start the PostgreSQL service, run:

    .. code-block:: bash

        brew services start postgresql

    Alternatively, to start PostgreSQL manually, run:

    .. code-block:: bash

        pg_ctl -D /usr/local/var/postgres start

    This command may vary according to your installation. Please verify instructions after installing postgres.

Creating a new PostgreSQL database
----------------------------------

With the PostgreSQL database installed, you can create a new database to store the jobs and results of the calculations. To do so, follow these steps:

1. In your terminal, run the following command to create a new database with the name ``mydb`` (replace by name of your choice):

.. code-block:: bash

    createdb mydb

2. Connect to the newly created database using psql:

.. code-block:: bash

    psql mydb

3. By default, PostgreSQL uses the current system user as the database user and does not require a password for local connections. To set a password for the database user, run the following SQL command within ``psql``, replacing ``your_password`` with the desired password:

.. code-block:: sql

    ALTER USER current_user WITH PASSWORD 'your_password';

After setting the password, update the PostgreSQL configuration file (``pg_hba.conf``) to require a password for connections. Find the line that starts with local or host, followed by the database name, user, and connection type (e.g., trust). Change the connection type to md5 or password to require a password for authentication.

4. Finally, to retrieve the current database name, user, and host, run the following SQL command within ``psql``:

.. code-block:: sql

    SELECT current_database(), current_user, inet_server_addr();

Now you have initialized a PostgreSQL database and retrieved the credentials (username, password, and database name) to access the database. Save these credentials, as we will use them later to access the database.
