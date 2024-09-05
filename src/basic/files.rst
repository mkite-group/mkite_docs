============================
Adding files to the database
============================

Very often, the production database requires importing files or lists of initial structures.
These structures can be, for example, molecules of interest in the form of SMILES strings, CIF files of crystals to be simulated, or instructions on how to import the data from other databases (e.g., Materials Project).
In this tutorial, we will show how to use the command `dbimport` from mkite to import structures into the database.

.. tip::

   Whereas adding files to a database simply parses the input as the result, it is often good to save these input configurations in a version-controlled repository. One option is to store them along with the config files or a separate repository where the information is stored. A separate repository is recommended when the files are large

Adding SMILES
-------------

When screening molecules from scratch, it is useful to add SMILES to the database, where they will be transformed into ChemNodes that can be inputs to other jobs.
The SMILES strings can be written as a JSON or YAML file:

.. code-block:: yaml

   - smiles: "CC"
     attributes:
        name: "ethane"
   - smiles: "CCC"
   - smiles: "CCCC"

With the `dbimport` command, one can import the file into the database:

.. code-block:: bash

   kitedb dbimport MolFileImporter -f FILE_PATH -p PROJECT_NAME -e EXPERIMENT_NAME

This will parse the file at ``FILE_PATH`` and add it to the database under the project ``PROJECT_NAME`` and experiment ``EXPERIMENT_NAME``.
Each line in the YAML file adds a new entry in the table ``Molecule`` in the database.
When an ``attributes`` entry is present, it adds the given information into the object. In the example above, the given information is the name of the molecule, which can be queried again in the future.

Adding CIF, XYZ, and other file formats
---------------------------------------

Often, one may want to add a CIF, XYZ, or any other file format.
Using the ``ase.io.read`` function, we can parse these files and convert them into the database schema.
To do so, simply use the ``AseFileImporter``:

.. code-block:: bash

   kitedb dbimport AseFileImporter -f FILE_PATH -p PROJECT_NAME -e EXPERIMENT_NAME

This will parse the file at ``FILE_PATH`` and add it to the database under the project ``PROJECT_NAME`` and experiment ``EXPERIMENT_NAME``.
If parsing conformers, you can specify which molecule it belongs to by setting a ``smiles`` attribute in the file (e.g., second line of an extended XYZ file).

.. note::

   The ``AseFileImporter`` reads whatever ASE is able to read.
   If there are periodic boundary conditions, the importer automatically converts the ``ase.Atoms`` into an mkite CrystalInfo.
   Otherwise, it creates an mkite ConformerInfo.
   Then, this Info file is parsed into the database.

Importing data from the Materials Project
-----------------------------------------

As public databases such as the Materials Project (MP) are excellent source of resources for further calculations, it is useful to have an interface that parses information from these databases directly into your mkite database.
To perform that, we use the ``mp_api`` interface to query the MP database according to a set of desired properties.
For example, an example of input file could be:

.. code-block:: json

    [
        {
           "rester": "summary",
           "query_function": "search",
           "chemsys": "Cu",
           "energy_above_hull": [0, 0.1],
           "num_sites": [1, 15],
           "fields": ["material_id", "structure", "energy_per_atom"]
        }
    ]

The file above uses the ``summary`` REST interface for the MP and searches all systems of composition "Cu" with ``energy_above_hull`` between 0 and 0.1 eV, and ``num_sites`` between 1 and 15.
Thus, the JSON file only contains the arguments passed to the MP rester in the original package.
More advanced queries can be performed by following the ``mp_api`` guide.

.. important::

   The ``dbimport`` command from mkite only provides an interface to the MP database, but not the credentials to do so.
   To obtain an API key, follow the instructions from the Materials Project team and export the API key in your environment as ``export MP_API_KEY="your_key"``.
