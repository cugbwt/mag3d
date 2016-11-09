
.. _magsen3d:

MAGSEN3D
========

This program performs the sensitivity calculation. Command line usage:

``magsen3d magsen3d.inp [nThreads]``

For a sample input file type:

``magsen3d -inp``

The argument specifying the number of CPU threads used in the OpenMP format is optional. If this argument is not given to the program, chooses to use all of the CPU threads on the machine. This argument allows the user to specify half, for example, of the threads so that the program does not take all available RAM. Note that this option is not available in the MPI-based code used for clusters.

Input files
-----------

Format of the control file:

.. figure:: ../../images/magsen3d.png
     :align: center
     :figwidth: 75% 

The input parameters for the control file are:

- ``mesh``: Name of 3D :ref:`mesh file <meshFile>`.

- ``obs``: The :ref:`data file <magFile>` that contains the observation locations. Note for sensitivity calculations, standard deviations are not required, but this file may be the observations that will be used in the inversion (with uncertainties).

- ``topo``: Surface :ref:`topography <topoFile>`. If ``null`` is entered, the surface will be treated as being flat on top of the mesh.

- ``x_weight.txt``: A file (in the :ref:`model file format <modelFile>`) giving the sensitivity the weighting function. This can be user-specific or generated from :ref:`pfweight <pfweight>`. It does *not* have to specifically be named "distance_weight.txt" or "depth_weight.txt".

- ``wvltx``: A five-character string identifying the type of wavelet used to compress the sensitivity matrix. The types of wavelets available are Daubechies wavelet with 1 to 6 vanishing moments (``daub1``, ``daub2`` and so on) and Symmlets with 4 to 6 vanishing moments (``symm4``, ``symm5``, ``symm6``). Note that ``daub1`` is the Haar wavelet and ``daub2`` is the Daubechies-4 wavelet. The Daubechies-4 wavelet is suitable for most inversions, while the others are provided for user's experimentation. If ``NONE`` is entered, the program does not use wavelet compression.

- ``itol``, ``eps``: An integer and real number that specify how the wavelet threshold level is to be determined. This line is ignored if no wavelet compression is being used, however the line *must still* be in the input file.
    | ``itol=1``: program calculates the relative threshold and ``eps`` is the relative reconstruction error of the sensitivity. A reconstruction error of 0.05 (95%) is usually adequate.
    | ``itol=2``: the user defines the threshold level and ``eps`` is the threshold to be used. If ``null`` is entered on this line, a default relative reconstruction error of 0.05 (e.g. 5%) is used and the relative threshold level is calculated (i.e., ``itol=1`` , ``eps=0.05``).
    | **NOTE** The detailed explanation of threshold level and reconstruction error can be found in the :ref:`wavelet section <waveletSection>` of this manual.

- ``Diag``: Option to output (``Diag=1``) or not output (``Diag=0``) diagnostic files. These files are: (1) the predicted data for a model of :math:`\rho=0.1` with the wavelet compressed sensitivity, (2) the predicted data for a model of :math:`\rho=0.1` with the full sensitivity, (3) the averaged sensitivity in each cell based on the wavelet compression. An extra line in the log file is also written giving the user the achieved reconstruction error (e.g. ``eps`` when ``itol=1`` from above).

- ``Scale``: Option to scale the senstivities (``Scale=1``) or the model objective function (``Scale=0``) by the depth weighting function. This value **must be 0** to scale the sensitivities through the model objective function when the **compact or blocky model norms** are chosen. If scaling the senstivity is chosen, the inversion code will output an error at the beginning. This value is optional and will be set to ``Scale=0`` if the line is not given. 

     **NOTE**: Emperical testing has shown that the results are consistent with either option. Setting ``scale=0`` will allow the user to run both a smooth and blocky inversion without having to re-compute the sensitivities.

Example of input file
~~~~~~~~~~~~~~~~~~~~~

Below is an example of the input file prior to using the inversion code for blockiness (``scale=0``). No diagnostic files will be output. Wavelets are used with a 5% re-construction error. Distance weighting from the :ref:`pfweight` program is the weighting of choice.

.. figure:: ../../images/magsen3dEx.png
     :align: center
     :figwidth: 75% 


Output files
------------

The program ``magsen3d`` outputs three files (with diagnostic files being optional output). They are:

#. ``magsen3d.log``: A log file summarizing the run of the program.

#. ``maginv3d.mtx``: The sensitivity matrix file to be used in the inversion. This file contains the sensitivity matrix, generalized depth weighting function, mesh, and discretized surface topography. It is produced by the program and it's name is not adjustable. It is very large and may be deleted once the work is completed.

#. ``sensitivity.txt``: This file is a :ref:`model file <modelFile>` that contains the average sensitivity for each cell. This file can be used for depth of investigation analysis or for use in designing special model objective function weighting.

#. Diagnostic files as described above to examine the wavelet compression properties, if chosen (``Diag=1``).

