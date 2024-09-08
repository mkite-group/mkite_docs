=============================
Example of project with mkite
=============================

To provide a clear example on how mkite can be used to perform larger-scale simulations, this tutorial shows how to structure the data, how to execute multiple calculations at once, and how to manage the results afterwards.
As before, we will use the ``mkite_conformers`` plugin to exemplify this tutorial.
The rationale for this is simple: one can easily run thousands of conformer generation calculations for small molecules without significant computational overhead.
Thus, the user learning this tutorial can have immediate feedback on how to use the mkite code without having to wait for actual, production DFT calculations.

This tutorial assumes that you have read the :doc:`basic tutorial <tutorial>` for mkite, which explains how to setup the databases, the engines, and so on.
See the requirements below before proceeding:

Requirements
------------

* You have :doc:`set up a PostgreSQL database <setup/postgres>` and created an ``.env`` :doc:`configuration file <setup/configs>` containing the credentials to access the database. The database is running, and you have full access to it.
* You have :doc:`set up a Redis engine <setup/engine>` and created a ``.yaml`` :doc:`configuration file <setup/configs>` containing the credentials to access the engine. The Redis engine is running, and you have full access to it.
* You have :doc:`set up a local engine <setup/engine>` (i.e., folders in the computer that will run the simulations) and created a ``.yaml`` :doc:`configuration file <setup/configs>` containing the credentials to access the engine.
* You have :doc:`set up your mkwind client <basic/mkwind>` along with the necessary configuration files to build jobs locally on the computer that will run the simulations. If you have not, we recommend using the `example configuration files <TODO>`_ as starting point, where you will define a local folder where your jobs will be built/run/postprocessed.
* You should have a functional scheduler where to run the jobs. If you are running on an HPC server that has SLURM, for instance, you can specify that on the configuration file above.
  If you would like to run the files locally (like this example), you can install a scheduler such as `pueue <TODO>`_ to manage the job submissions.

Downloading and understanding the example
-----------------------------------------

In a folder of interest, download the code for the examples:

.. code-block:: bash

   git clone https://github.com/mkite-group/example_project

This project has the following file structure:

- ``data/``: Contains YAML files with the SMILES of the molecules whose conformers have to be generated.
- ``workflows/``: Contains YAML files with descriptions on how to run the jobs
- ``scripts/``: Contains utility scripts for database management and orchestrating the simulations.

Below, we explain each of these folders and the files therein.

Data
^^^^

The ``data/`` folder contains YAML files that describe the molecules whose conformers have to be generated.
For example:

.. code-block:: yaml

    - smiles: "CCCCCN"
    - smiles: "Cc1ccncc1"
    - smiles: "NCCOCCO"
    - smiles: "CCNCC"
    - smiles: "CCCCN"
    - smiles: "c1ccncc1"

The list of molecules was selected from the list of SMILES from [this paper](https://doi.org/10.1126/science.abh3350) as an example for this tutorial.

Workflows
^^^^^^^^^

The ``workflows/`` folder contains YAML files describing the simulation workflows and the jobs that have to be created.
Because this example tutorial is very simple, we have only two jobs: importing the YAML file containing the information on the SMILES; and creating jobs that apply to all imported SMILES.

Example workflow file (``workflows/02_conformer.yaml``):

.. code-block:: yaml

    - out_experiment: 02_conformer
      out_recipe: conformer.generation
      inputs:
        - filter:
            parentjob__experiment__name: 01_import
            parentjob__recipe__name: dbimport.MolFileImporter
      tags:
        - confgen

The YAML file above specifies that nodes created with experiment ``01_import`` and ``dbimport.MolFileImporter`` will be used as inputs for new jobs whose experiments are ``02_conformer`` and recipe ``conformer.generation``. The new jobs will receive the tag ``confgen``. The tag is arbitrary and can be anything chosen by the user.

Scripts
^^^^^^^

The ``scripts/`` folder contains utility scripts for managing the database and running simulations:

- ``create.sh``: this file loads each of the YAML files in the ``workflows`` folder and creates the jobs for each of them.
- ``submit.sh``: this script submits jobs with status ``READY`` on the database to the engine. In this case, the engine has to be specified, but it can be the Redis engine or a local folder.
- ``parse.sh``: this script parses jobs that have been postprocessed by ``mkwind`` and reside in the engine prior to being integrated into the production database.
- ``backup.sh``: this script backs up the production database to a tarfile

These scripts have to be modified to contain your own paths for the files, the configuration files for the engines, and so on.
You can also specify which database configuration will be used using the ``MKITE_ENV`` environmental variable.

.. important::

    You do not have to export globally the ``MKITE_ENV`` environmental variable.
    Rather, you can export it directly on the scripts above, ensuring that no conflict between databases emerge if you handle more than one project at once.
    This is also better in the context of crontabs, where relying on global environmental variables may be tricky.

.. note::

   The folder structure below is only an example on how to organize the files regarding mkite.
   You can feel free to choose the folder structure that best organizes your files, or best makes sense to you.
   In this case, make sure that you point to the right files when editing the scripts and so on.


Initializing the project
------------------------

If this is your first time running the database, make sure you perform the right migrations:

.. code-block:: bash

   kitedb makemigrations base jobs calcs mols structs workflow
   kitedb migrate

Now, you can start an example project with the commands below:

.. code-block:: bash

   kitedb create_project conformer
   kitedb create_experiment conformer 01_import
   kitedb create_experiment conformer 02_conformer

This initializes the project and experiments for this example.

.. note::

    ``mkite`` requires you to create the experiments before running the workflows.
    This is a design choice: you could want to create them automatically.
    However, it is safer for the user to define which are the names of experiments that are desired when organizing the workflow.

If you have not done so, also update your database with the recipes available:

.. code-block:: bash

   kitedb scanrecipes

After initializing the database and the experiments, make sure that your Redis and ``pueued`` daemons are running correctly, and that you have the right configurations for them.

Importing molecules to the database
-----------------------------------

To get started with the project and import new molecules .cd into the repository folder called `workflow`, run `01_import.sh`:

.. code-block:: bash

   cd workflow && ./01_import.sh

The contents of the ``01_import.sh`` file are very simple:

.. code-block:: bash

    kitedb dbimport MolFileImporter -p conformer -e 01_import -f ../data/smiles.yaml

This file essentially imports the SMILES strings in ``data/smiles.yaml`` and adds them to the database under the project ``conformer`` that was created above.
You can now verify if the molecules were correctly imported into the database by opening a shell that accesses the database:

.. code-block:: bash

   kitedb shell_plus

And with the shell open, you can query the results by counting the number of ``Molecule`` objects in the database:

.. code-block:: python

   print(f"Num. molecules: {Molecule.objects.count()}")

The result should show 388 molecules, which is the number of distinct SMILES in the ``smiles.yaml`` file.

.. code-block:: text

   Num. molecules: 388

If you want to determine a more advanced query, you could find, for instance, how many molecules contain phosphonium ions:

.. code-block:: python

    query = Molecule.objects.filter(smiles__contains="[P+]")
    print(f"Num. molecules: {query.count()}")

You can also analyze the jobs to understand that mkite created a separate job for each molecule that was added to the database:

.. code-block:: python

    mol = Molecule.objects.first()
    job = mol.parentjob
    print(job.options)

All other operations you would expect from Django are available.

Creating new jobs for the molecules
-----------------------------------

Once the molecules have been added to the database, you can create new jobs for these molecules.
As described in the mkite paper, the ``Molecule`` object is a subclass of ``ChemNode``, so jobs can be created for these nodes of interest.
The simplest way to do that is to cd into the repository folder called `scripts`, run `create.sh` to create the jobs in the database:


.. code-block:: bash

   cd scripts && ./create.sh

The contents of the ``01_import.sh`` file are very simple:

.. code-block:: bash

    #!/bin/bash

    echo "----------------------------"
    echo $(date)
    echo "----------------------------"

    WORKFLOW_DIR="$PWD/../workflows"

    CREATE_SIMPLE="kitedb create_from_file simple"
    CREATE_TUPLE="kitedb create_from_file tuple"

    $CREATE_SIMPLE $WORKFLOW_DIR/02_conformer.yaml

The ``create.sh`` script essentially create one job per ``ChemNode`` satisfying the rules described in the ``02_conformer.yaml`` file (``kitedb create_from_file simple`` command).
The ``02_conformer.yaml`` file, on its turn, is also somewhat straightforward to understand:

.. code-block:: yaml

    # Create conformer job for each of the SMILES imported in the previous step
    - out_experiment: 02_conformer
      out_recipe: conformer.generation
      inputs:
        - filter:
            parentjob__experiment__name: 01_import
            parentjob__recipe__name: dbimport.MolFileImporter
      tags:
        - confgen

This file specifies the creation of the job that has the following requirements:

- ``out_experiment``: the job to be created will correspond to the experiment ``02_conformer``.
  This is useful to organize the results of the calculations with interpretable tags, which are very helpful when performing queries and analyses.
- ``out_recipe``: the job to be created will have the recipe ``conformer.generation``. This recipe is assumed to be in the database already, having been added after the ``kitedb scanrecipes`` and installing the ``mkite_conformer`` plugin.
- ``inputs``: one job will be created for each of the inputs (``ChemNodes``) that satisfy the proposed filter. In this case, jobs are only created for ``ChemNodes`` whose parent jobs (i.e., the job that created them) belong to the experiment ``01_import`` and have been created with the recipe ``dbimport.MolFileImporter``.
- ``tags``: the tag ``confgen`` is applied to all jobs created by this file. Tags are arbitrary and optional, and are only useful if the user requires them for any reason.

Once you understand the file structure of workflows, you can start connecting the YAML files to define your own workflows.

Now, you can open the shell to the database again with ``kitedb shell_plus`` and query the jobs to see if they were created successfully:

.. code-block:: python

   jobs = Job.objects.filter(recipe__name="conformer.generation", status="Y")
   print(jobs.count())

The status ``Y`` of the jobs indicate that they are ready to be submitted to an engine.

in the folder `scripts`, run `submit.sh` to send the jobs to the Redis engine (make sure you point to the right credentials file by editing the script)
use `mkwind` (build, run, postprocess) to handle the job execution with pueue and interface with Redis. It should create 388 jobs of conformer generation for the molecules, which is enough to demonstrate that the HTS engine is useful, and too much to run by hand.
the jobs will run quite fast, be postprocessed, and return to the redis engine
in the folder `scripts`, run `parse.sh` to make the production database access the redis engine and ingest the results into the postgresql db.
from there on, you should be able to query your results.
