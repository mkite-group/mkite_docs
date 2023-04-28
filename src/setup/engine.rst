====================
Setting up an engine
====================

This step is only required if you are using an intermediate engine (or message broker) to communicate between the main server and the clients. The engine can be something as simple as a folder in your filesystem, or a more complicated entity such as a Redis database.

Installing Redis locally
-------------------------------

.. tab:: macOS

    Install Redis using ``brew``:

    .. code-block:: bash

        brew install redis

    Then, start the Redis service with:

    .. code-block:: bash

        brew services start redis

    Instead of using ``brew services``, you can also start Redis manually with:

    .. code-block:: bash

        redis-server /usr/local/etc/redis.conf

.. tab:: Linux

    On Debian-based systems (e.g., Ubuntu), run:

    .. code-block:: bash

        sudo apt-get update
        sudo apt-get install redis-server

    On RHEL-based systems (e.g., CentOS, Fedora), run:

    .. code-block:: bash

        sudo yum install redis

    On SUSE-based systems (e.g., openSUSE), run:

    .. code-block:: bash

        sudo zypper install redis

    Then, enable and start the Redis service:

    .. code-block:: bash

        sudo systemctl enable redis
        sudo systemctl start redis

Accessing the Redis Database
------------------------------

You can access the Redis database by using the command line interface:

.. code-block:: bash

  redis-cli

To change databases, we should note that Redis does not use separate databases. Rather, it uses namespaces. By default, Redis provides 16 namespaces, numbered from 0 to 15. Within the ``redis-cli`` environment, you can switch between namespaces using the SELECT command followed by the namespace number.

.. code-block:: sql

  SELECT 0

Retrieving Redis Credentials
--------------------------------

Redis does not require a username by default. To set a password for your Redis server, open the Redis configuration file (``/usr/local/etc/redis.conf`` on macOS or ``/etc/redis/redis.conf`` on Linux) and find the line that starts with ``# requirepass``. Uncomment the line and set the password after ``requirepass``:

.. code-block:: text

  requirepass your_password

Save the file and restart the Redis service.

This file should also contain the port the Redis engine is listening on. These credentials are important and are going to be used later.

Hosting Redis on the Cloud
--------------------------------------

Several cloud providers offer managed Redis services, which can save you the effort of setting up, managing, and scaling your Redis instances (for a cost). Some options include:

- Redis Enterprise Cloud: https://redis.com/redis-enterprise-cloud/overview/
- AWS ElastiCache for Redis: https://aws.amazon.com/elasticache/redis/
- Google Cloud Memorystore for Redis: https://cloud.google.com/memorystore/docs/redis/
- Azure Cache for Redis: https://azure.microsoft.com/en-us/services/cache/

Choose a provider, sign up for an account, and follow their instructions to create and configure a Redis instance. The provider's documentation should also include instructions on how to retrieve the credentials (username, password, port number etc).

.. warning::
    As the Redis engine serves as a middle layer between the client and server, it is often widely accessible. Make sure your engine is secure in production.
