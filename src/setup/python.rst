==============================
Preparing a Python environment
==============================

Using conda
-----------

First, install either Miniconda (https://docs.conda.io/en/latest/miniconda.html) or Anaconda (https://www.anaconda.com/products/distribution). Then, run the following command to create the ``mkite`` environment:

.. code-block:: bash

    conda create -n mkite python=3.8

To activate the newly created environment, run:

.. code-block:: bash

    conda activate mkite

Using pip
---------

If ``virtualenv`` is not yet installed with your Python distribution, run the following command:

.. code-block:: bash

    pip install virtualenv

Then, navigate to the folder where you want to create a new environment (e.g., ``~/envs``). After that, run the following command:

.. code-block:: bash

    python3 -m venv mkite

To activate the newly created environment, run:

.. code-block:: bash

    source mkite/bin/activate

