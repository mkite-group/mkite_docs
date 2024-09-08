========================
Projects and experiments
========================

In any "lab notebook", information has to be arranged according to projects, experiments, and many other metadata.
This ensures searchability of the results, as well as reproducibility of each aspect of the experiments.
Analogously, computational simulations require creating an organized set of jobs (e.g., simulations or calculations) that are searchable and human-readable.

The mkite package requires the organization of these jobs with the *Project* and *Experiment* metadata.
In mkite's definitions, a Project is a set of Experiments. 
Experiments may or not be ordered, and represent different stages of the calculation branch.
For example, as described in `mkite's paper <https://arxiv.org/abs/2301.08841>`_, different branches of a calculation graph can be grouped under a single experiment.
Then, new experiments can be created by grouping the information from previous experiments into new jobs.

Creating new projects
---------------------

Projects are basically defined by one metadata, its name.
New projects can be easily created in the database interface (e.g., Django's ``shell_plus`` or PostgreSQL's ``psql``).
For convenience, they can also be added to the database with the ``create_project`` command:

.. code-block:: bash

   kitedb create_project PROJECT_NAME

Creating new experiments
------------------------

Similarly, experiments are defined by a name and a project.
They can be created using the ``create_experiment`` command as:

.. code-block:: bash

   kitedb create_experiment PROJECT_NAME EXPERIMENT_NAME

If this is your first time running ``kitedb`` you will need to initialize the database

.. code-block:: bash

   kitedb makemigrations base jobs calcs mols structs workflow
   kitedb migrate


With this, a new experiment titled ``EXPERIMENT_NAME`` will be created under the project ``PROJECT_NAME``.
