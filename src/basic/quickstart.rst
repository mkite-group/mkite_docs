=================
15-min quickstart
=================

This concise tutorial shows how to get started running jobs with mkite in 15 min. It avoids the process of setting up a database and an engine, so if you are interested in doing that, check the `complete tutorial <index>`_.

Install mkite
-------------

In an existing Python (>= 3.8) environment, install mkite, mkwind, and a few example plugins from pip:

.. code-block:: bash

   pip install mkite_core mkite_conformer

Running your first job with mkite
---------------------------------

Set up job specifications
^^^^^^^^^^^^^^^^^^^^^^^^^

Jobs in mkite are created using the ``JobInfo`` class in ``mkite_core.models``. A ``JobInfo`` is essentially a JSON file with a predefined schema, however. You can create the basic specifications for a conformer generation job like the following:

.. code-block:: json

    {
        "job": {},
        "recipe": {"name": "conformer.generation"},
        "inputs": [
            {"smiles": "CN1C=NC2=C1C(=O)N(C(=O)N2C)C"}
        ],
        "options": {"force_field": "mmff"}
    }

Save the content above in a file, say ``job_info.json``.

Running a job
^^^^^^^^^^^^^

The specifications above create the bare minimum to run the ``ConformerGenerator`` recipe in ``mkite_conformer``. Now, with your Python environment activated, run in the command line:

.. code-block:: bash

   kite run -i job_info.json

After a few seconds, a new file called ``jobresults.json`` is generated and the job finishes succesfully.

Understanding the output
^^^^^^^^^^^^^^^^^^^^^^^^

The ``jobresults.json`` file contains all major information from the job, including all the default parameters used in the calculation, the statistics on the run, and the actual results. For example, a possible output is:

.. code-block:: json

    {
        "job": {
            "status": "D",
            "options": {
                "force_field": "mmff",
                "num_conformers_returned": 20,
                "num_conformers_generated": 200,
                "num_attempts": 5,
                "prune_threshold": 0.1,
                "cluster_rmsd_tol": 2,
                "threads": 1,
                "random_seed": 6739324
            }
        },
        "runstats": {
            "host": "your_host",
            "cluster": "your_host",
            "duration": 1.616744,
            "ncores": 1,
            "ngpus": 0,
            "pkgversion": "mkite-conformer 0.1.0"
        },
        "nodes": [
            {
                "chemnode": {
                    "species": ["C", "N", "C", "C", "N", "C", "C", "O", "N", "C", "C", "O", "N", "C", "H", "H", "H", "H", "H", "H", "H", "H", "H", "H"],
                    "coords": [
                        [3.096, 1.163, -0.406],
                        [2.141, 0.093, -0.267],
                        [2.420, -1.247, -0.275],
                        [1.329, -1.972, -0.125],
                        [0.326, -1.057, -0.019],
                        [0.791, 0.220, -0.102],
                        [-0.049, 1.362, -0.020],
                        [0.379, 2.509, -0.096],
                        [-1.396, 1.030, 0.152],
                        [-1.916, -0.277, 0.243],
                        [-3.126, -0.469, 0.396],
                        [-1.008, -1.332, 0.151],
                        [-1.462, -2.708, 0.234],
                        [-2.359, 2.110, 0.250],
                        [3.058, 1.783, 0.492],
                        [4.099, 0.743, -0.522],
                        [2.837, 1.747, -1.293],
                        [3.423, -1.638, -0.392],
                        [-2.544, -2.773, 0.370],
                        [-0.973, -3.194, 1.085],
                        [-1.193, -3.230, -0.689],
                        [-3.097, 2.002, -0.550],
                        [-1.894, 3.096, 0.173],
                        [-2.879, 2.037, 1.211]
                    ],
                    "formula": {
                        "name": "H10 C8 N4 O2 +0",
                        "charge": 0
                    },
                    "mol": {
                        "inchikey": "RYYVLZVUVIJVGH-UHFFFAOYSA-N",
                        "smiles": "Cn1c(=O)c2c(ncn2C)n(C)c1=O"
                    },
                    "siteprops": {},
                    "attributes": {},
                    "@module": "mkite.orm.mols.models",
                    "@class": "Conformer"
                },
                "calcnodes": [
                    {
                        "energy": -122.528,
                        "forces": null,
                        "attributes": {},
                        "@module": "mkite.orm.calcs.models",
                        "@class": "EnergyForces"
                    }
                ]
            }
        ],
        "workdir": null
    }

In the JSON file above, each field has a different role:

1. **job**: provides specifications on the job that was run. For example, returns all the default parameters used in the task, as well as whether the job finished successfully (status "D", for DONE).
2. **runstats**: provides information on where the job was run, how long it took, the number of cores used etc.
3. **nodes**: contains the results of the calculation. A node is the main object in the database that is the input (or output) of a job, and can be a ChemNode (anything that resembles a chemical structure), or a CalcNode (anything that resembles a property or calculation result). In the example above, we generated:
   - One ChemNode of the class Conformer from the module ``mkite.orm.mols.models``, that contains the 3D coordinates of the molecule.
   - One CalcNode of the class EnergyForces from the module ``mkite.orm.calcs.models``, that contains the information of the energy of the calculation, as computed by the MMFF94 force field.
4. **workdir**: this tag is often used when chained calculations are used. Check the advanced guide for more information.

Visualizing the output
^^^^^^^^^^^^^^^^^^^^^^

If you have a visualization package installed, you can see the result of the calculation by yourself. Open a Jupyter Notebook or similar environment and run the following:

.. code-block:: python

    import nglview as nv
    from mkite_core.models import JobResults, ConformerInfo

    results = JobResults.from_json("jobresults.json")
    conf = ConformerInfo.from_dict(results.nodes[0].chemnode)
    atoms = conf.as_ase()

    nv.show_ase(atoms)

The snippet above converts the ChemNode into a ConformerInfo, which is just a class that mimics the ``Conformer`` table in ``mkite_db``, but without the need for a database. Then, we convert the ConformerInfo into an ``ase.Atoms`` object that can be visualized with nglview. The result should be similar to:

..
    .. image:: _img/caffeine.png
        :align: center
        :alt: 3D conformation of a caffeine molecule

With that, we have run our first job in mkite.

Other ways to run the same job
------------------------------

Running the job directly from Python
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The job does not have to be executed from the command line. If you prefer to run the job directly from a Python environment, for example, you can easily do so. You can just instantiate the ``JobInfo`` and the desired recipe to run the job:

.. code-block:: python

   from mkite_core.models import JobInfo
   from mkite_conformer.recipes.rdkit import ConformerGenerationRecipe

   inputs = [{"smiles": "CN1C=NC2=C1C(=O)N(C(=O)N2C)C"}]
   options = {"force_field": "mmff"}

   # because we are using the recipe, we do not
   # have to specify it.
   info = JobInfo(
        job={},
        recipe={},
        inputs=inputs,
        options=options,
   )
   recipe = ConformerGenerationRecipe(info)
   results = recipe.run()

With the code above, we will generate the same ``JobResults`` that were previously in a JSON file, but now directly into the Python environment.

Not using mkite's schema
^^^^^^^^^^^^^^^^^^^^^^^^

If you want to reuse the software, but not use mkite's model schema to interact with the results, you can use the ``runners`` classes directly in each plugin. For example, the ``mkite_conformer.runners.rdkit.ConformerGenerator`` class enables generating a conformer directly from an ``rdkit.Chem.Mol``. We can use that class directly in Python:

.. code-block:: python

    from mkite_conformer.runners.rdkit import ConformerGenerator

    smiles = "CN1C=NC2=C1C(=O)N(C(=O)N2C)C"
    confgen = ConformerGenerator.from_smiles(smiles)
    mol, energies = confgen.run()

This independence between the runner, the recipe, and the job building enables mkite to be a fast tool for prototyping. At the same time, the framework is extensible and can be used for calculations at higher throughput.

Increasing the throughput
-------------------------

Creating and executing several jobs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now that we have created the conformer for one molecule, it is very simple to extend that functionality for others.
For example, if we wanted to increase the throughput of the calculations, we could create several JobInfo files.
Say we wanted to create conformers for each of the molecules in the `MD17 dataset from Chmiela et al. (2017) <https://doi.org/10.1126/sciadv.1603015>`_.
We can create different folders for each of the jobs and run them in parallel:

.. code-block:: python

    import os
    from mkite_core.models import JobInfo

    molecules = {
        "benzene": "c1ccccc1",
        "uracil": "O=C1NC=CC(=O)N1",
        "naphthalene": "c1ccc2ccccc2c1",
        "aspirin": "O=C(C)Oc1ccccc1C(=O)O",
        "salicylic acid": "O=C(O)c1ccccc1O",
        "malonaldehyde": "O=CC=O",
        "ethanol": "CC(O)C",
        "toluene": "Cc1ccccc1"
    }
    get_inputs = lambda smiles: [{"smiles": smiles}]
    options = {"force_field": "mmff"}

    for name, smiles in molecules.items():
        path = f"job_{name}"
        if not os.path.exists(path):
            os.mkdir(path)

        info = JobInfo(
            job={},
            recipe={},
            recipe={"name": "conformer.generation"},
            inputs=get_inputs(smiles),
            options=options,
        )
        info.to_json(os.path.join(path, "jobinfo.json"))

With all these jobs in their own directories, we can parallelize them by using a bash script:

.. code-block:: bash

    #!/bin/bash

    for job_folder in job_*
    do
        cd $job_folder
        kite run -i jobinfo.json &
        cd ..
    done

    wait

This script runs all conformer generation jobs and waits for their completion.

What if I wanted to handle thousands of jobs?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You could create thousands of files and use an excellent tool like `GNU Parallel <https://www.gnu.org/software/parallel/>`_ to perform your task. This would work well for conformer generation of small molecules. However, if you want to systematically perform many more calculations (or much slower ones, such as DFT for large systems) in an HPC environment, it is better to use a scheduler to execute that job. With the scheduler, many other questions would emerge, such as:

- How to manage the job submission after they have been created?
- How to save the results of calculations for later?
- How to handle different schedulers and clusters?
- ...

This is where this quickstart stops.
For more on the questions above and how to understand the mkite structure that enables high-throughput calculations, proceed to the :doc:`../basic/index`.
For a practical demonstration on how to do that, proceed to the :doc:`step-by-step practical example </basic/example>`.
