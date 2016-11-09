.. _maginv3d:

MAGINV3D
========

This program actually performs the 3D inversion of magnetic data. Command line usage is:

``maginv3d maginv3d.inp [nThreads]``

For a sample input file type:

``maginv3d -inp``

The argument specifying the number of CPU threads used in the OpenMP format is optional. If this argument is not given to the program, chooses to use all of the CPU threads on the machine. This argument allows the user to specify half, for example, of the threads so that the program does not take all available RAM. Note that this option is not available in the MPI-based code used for clusters.

Input files
-----------

Input files can be any file name. If there are spaces in the path or file name, you *MUST* use quotes around the entire path (including the filename). Files that may be used by the inversion are:

#. ``obs.mag``: Mandatory :ref:`observations file <magfile>`.

#. ``maginv3d.mtx``: Mandatory sensitivity matrix from :ref:`magsen3d`

#. ``initial.sus``: Optional initial :ref:`model file <modelFile>`. This can be substituted by a value within the input file (see below).

#. ``ref.sus``: Optional reference :ref:`model file <modelFile>`. This can be substituted by a value within the input file (see below).

#. ``active.txt``: Optional ref:`active model file <activeFile>`

#. ``upperBound.sus``: Optional upper bounds :ref:`model file <modelFile>`. A value can be used to set a global bound (see below).

#. ``lowerBound.sus``: Optional lower bounds :ref:`model file <modelFile>`. A value can be used to set a global bound (see below).

#. ``weights.wt``: Optional :ref:`weighting file <weightsFile>`.

#. ``maginv3d.inp``: The control file containing the options. Does not need to be specifically called "maginv3d.inp".

Format of the control file has been changed since previous version. Any numeric entries beyond the trade-off parameter, and tolerance should be preceded by ``VALUE``. The input files has been modified as follows:


.. figure:: ../../images/maginv3d.png
     :align: center
     :figwidth: 75%

 

The parameters within the control file are:

-  ``mode``: An integer specifying one of two choices for determining the trade-off parameter.

   #. ``mode=1``: the program chooses the trade off parameter by carrying out a line search so that the target value of data misfit is achieved (e.g., :math:`\phi_d^*=N`).

   #. ``mode=2``: the user inputs the trade off parameter.


- ``par``, ``tolc`` Two real numbers that are depe.sust upon the value of mode.
   
   #. ``mode=1``: the target misfit value is given by the product of ``par`` and the number of data :math:`N` , i.e., ``par=1`` is equivalent to :math:`\phi_d^*=N` and ``par=0.5`` is equivalent to :math:`\phi_d^*=N/2` . The second parameter, ``tolc``, is the misfit tolerance in fractional percentage. The target misfit is considered to be achieved when the relative difference between the true and target misfits is less than ``tolc``. Normally, ``par=1`` is ideal if the true standard deviation of error is assigned to each datum. When ``tolc=0``, the program assumes a default value of ``tolc=0.02`` since this number must be positive.

   #. ``mode=2``: ``par`` is the user-input value of trade off parameter. In this case, ``tolc`` is not used by the program.

   | **NOTE:** When both ``par`` and ``tolc`` are used. When only ``par`` is used. When ``mode=3``, neither nor ``tolc`` are used. However, the third line should always have two values.

-  ``obs``: Input data file. The file must specify the standard deviations of the error. By definition these values are greater than zero.

-  ``matrixFile``: The binary file of sensitivities created by :ref:`magsen3d`.

-  ``initial``: The initial susceptibility model (SI) can be defined as a value for uniform models (e.g. ``VALUE 0.001``), or by a filename. The initial model must be within the upper and lower bounds.

-  ``ref``: The reference susceptibility model (SI) can be defined as a value for uniform models (e.g. ``VALUE 0``), or by a filename (for non-uniform reference models).

- ``active``: The :ref:`active model file <activeFile>` defining which cells in the model are allowed to be solved.

-  ``lowerBound``: The lower bounds model (SI) can be defined as a value for uniform models (e.g., ``VALUE -1``) or by a filename.

-  ``upperBound``: The upper bounds model (SI) can be defined as a value for uniform models (e.g., ``VALUE 1``) or by a filename.

- :math:`\alpha_s, \alpha_x, \alpha_y, \alpha_z`: Coefficients for the each model component. :math:`\alpha_s` is the smallest model component. Coefficient for the derivative in the easting direction. :math:`\alpha_y` is the coefficient for the derivative in the northing direction. The coefficient :math:`\alpha_z` is for the derivative in the vertical direction.

   If ``null`` is entered on this line, then the above four parameters take the following default values:  :math:`\alpha_s = 0.0001, \alpha_x = \alpha_y = \alpha_z = 1`. All alphas must be positive and they cannot be all equal to zero at the same time.

   **NOTE:** The four coefficients in line 9 of the control file may be substituted for three corresponding *length scales* :math:`L_x, L_y` and :math:`L_z` and are in units of metres. To understand the meaning of the length scales, consider the ratios :math:`\alpha_x/\alpha_s`, :math:`\alpha_y/\alpha_s` and :math:`\alpha_z/\alpha_s`. They generally define smoothness of the recovered model in each direction. Larger ratios result in smoother models, smaller ratios result in blockier models. The conversion from :math:`\alpha`\'s to length scales can be done by:

   .. math::
      L_x = \sqrt{\frac{\alpha_x}{\alpha_s}} ; ~L_y = \sqrt{\frac{\alpha_y}{\alpha_s}} ; ~L_z = \sqrt{\frac{\alpha_z}{\alpha_s}}

   When user-defined, it is preferable to have length scales exceed the corresponding cell dimensions. Typically having length scales of four cell widths are a good starting point.

- ``SMOOTH_MOD``: This option was not available in previous versions of the code and can be used to define the reference model in and out of the derivative terms. The options are: ``SMOOTH_MOD_DIF`` (reference model is defined in the derivative terms) and ``SMOOTH_MOD`` (reference model is defined in only the smallest term). See equation details on the :ref:`model objective function <mof>` section for details.

- ``weights``: Name of the :ref:`weights file <weightsFile>` containing weighting matrices. If ``null`` is entered, default values of unity are used (no extra weighting).


Example of control file
~~~~~~~~~~~~~~~~~~~~~~~

Below is an example of a control file with comments:

.. figure:: ../../images/maginv3dEx.png
     :align: center
     :figwidth: 75%



Output files
------------

Five general output files are created by the inversion. They are:

#. ``maginv3d.log``: The log file containing the minimum information for each iteration and summary of the inversion.

#. ``maginv3d.out``: The "developers" log file containing the details of each iteration including the model objective function values for each component, number of conjugate gradient iterations, etc.

#. ``maginv3d_xxx.sus``: Susceptibility (SI) :ref:`model files <modelFile>` output after each "xxx" iteration (i.e., ``maginv3d_012.sus``)

#. ``maginv3d_xxx.pre``: :ref:`Predicted data files <magFile>` (without uncertainties) output after each "xxx" iteration.

