.. _utils:

ChIMES Calculator Utilities
========================================

The PES Generator
*****************

Input
^^^^^^^^^

A utility for generating ChIMES potential energy surface scans for *n*-body clusters is available in ``utils/pes_generator``. To use this utility, create a file name ``config.py`` in the desired working directory, structured as follows:

.. code-block:: python

    CHMS_REPO  = "/path/to/your/chimes_calculator/repository/"
    
    PARAM_FILE = "/path/to/your/chimes_calculator/repository/serial_interface/tests/force_fields/test_params.CHON.txt"
    
    PAIRTYPES  = [0,    3,    5   ] # Pair type index for scans, i.e. number after "PAIRTYPE PARAMS:" in parameter file
    PAIRSTART  = [1.0,  1.0,  1.0 ] # Smallest distance for scan
    PAIRSTOP   = [4.0,  4.0,  4.0 ] # Largest distance for scan
    PAIRSTEP   = [0.01, 0.01, 0.01] # Step size for scan
    
    TRIPTYPES  = [1,    4   ] # Triplet type index for scans, i.e. number after "TRIPLETTYPE PARAMS:" in parameter file
    TRIPSTART  = [1.0,  1.0 ] # Smallest distance for scan
    TRIPSTOP   = [4.0,  4.0 ] # Largest distance for scan
    TRIPSTEP   = [0.10, 0.10] # Step size for scan
    
    # The example parameter file doesn't contain four body interactions, so the following is not needed.
    # If four body scans are desired, keep in mind a small step size will take a long time to run
    # Start with something very large to get a handle on run time, and modify from there
    #
    #QUADTYPES  = [7   ] # Triplet type index for scans, i.e. number after "TRIPLETTYPE PARAMS:" in parameter file
    #QUADSTART  = [1.0 ] # Smallest distance for scan
    #QUADSTOP	= [4.0 ] # Largest distance for scan
    #QUADSTEP	= [1.00] # Step size for scan

Variables ``CHMS_REPO`` and ``PARAM_FILE`` specify the ``chimes_calculator`` repository location, and path to the ChIMES parameter of file. Note that paths should be provided in their absolute form. Following these variables, three sets of options are provided. Focusing on options beginning with ``PAIR``, one must provide the following: 

* A list of pair type indices for which scans should be generated

    * Indices should correspond to values following ``PAIRTYPE PARAMS:`` in the target parameter file
    
* A list of the minimum pair distance for each pair type to consider during the scan
* A list of the maximum pair distance for each pair type to consider during the scan
* A scan step size

All input and output distances are in Angstroms, and all energies are provided in kcal/mol. Additionally, note that the penalty function will be included in scan results unless ``PAIRSTART`` is greater than the sum of the pair interaction inner cutoff and the penalty kick-in distance, or if the user has set ``PAIR CHEBYSHEV PENALTY SCALING`` to zero in the parameter file. Similar variables must be set to specify desired 3- and 4-body scans. Note that empty lists can be provided if no scan is desired.


Output
^^^^^^^^^

All *n*-body scans will produce output scan files named like ``chimes_scan_<n>b.type_<index>.dat``, where <n> is the bodiedness, and <index> is the ``PAIRTYPES``, ``TRIPTYPES``, or ``QUADTYPES`` index. Many-body scans will produce additional files named like ``chimes_scan_2+3b.type_<index>.dat`` or ``chimes_scan_2+3+4b.type_<index>.dat``, which include contributions from lower-bodied interactions as well. 

The first line in each output file provides a comment (prepended by a ``#``) starting and stopping distances followed by the scan step size. Following, each line provides the *ij* (and if appropriate, *ik*, *il*, *jk*, *jl*, and *kl* distances, respectively) and the corresponding cluster energy. For example, consider the ``test_params.CHON.txt`` parameter file provided in ``serial_interface/tests/force_fields/``, which contains the following 3-body interaction:

.. code-block:: text

    TRIPLETTYPE PARAMS:
       INDEX: 5 ATOMS: C H O
       PAIRS: CH CO HO UNIQUE: 54 TOTAL: 54
         index  |  powers  |  equiv index  |  param index  |       parameter       
       ----------------------------------------------------------------------------
          0       0  1  1         0               0        6.500656496400314
          1       0  1  2         1               1        3.7493801790331345
          2       0  1  3         2               2        0.0
          3       0  2  1         3               3        -4.7147262741975711
          4       0  2  2         4               4        -2.0557295465375991
          5       0  2  3         5               5        -1.1723283559758126

In the above example, ``TRIPTYPES`` is 5, corresponding to i, j, and k atoms of type C, H, and O, respectively. Thus, lines in the corresponding resulting 3-body scan file would give the *ij* (C-H), *ik* (C-O), and *jk* (H-O) distances, followed by the corresponding cluster energy.

Visualizing
^^^^^^^^^^^

Two-body scans can be immediately be plotted by most software (e.g. matplotlib, xmgrace, etc.), however additional considerations are needed to plot the > 3 dimensional 3- and/or 4-body scans. Three body scans can visualized in slices. An additional utility is provided in  utils/pes_generator (i.e. ``gnuplotify``), which can be used to extract these slices in a gnuplot splot-friendly format. To use this script, the user must specify a 3-body scan file and a *ij* distance at which to make the slice. Note that the *ij* distance must be one listed in the 3-body scan file. For the ``test_params.CHON.txt`` and ``config.py`` example above, this can be achieved with:

.. code-block:: bash

    python3.X gnuplotlify.py chimes_scan_2+3b.type_0.dat 2.5
    
This command will produce a file named like ``chimes_scan_2+3b.type_0.dat.gnuplot.2.5`` that can be plotted in gnuplot via:

.. code-block:: bash

     splot 'chimes_scan_2+3b.type_0.dat.gnuplot.2.5' u 1:2:3 w pm3d
