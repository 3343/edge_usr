Installation
============
This chapter describes the manual installation of EDGE and its dependencies from scratch.
If you are looking for an automated installation, consider skipping to EDGE's support for cloud infrastructure in Sec. :doc:`../setup/cloud`

Examples
--------
EDGE's entire installation process is continuously tested.
If you get stuck with this user guide, you might be able to find additional installation hints in the respective configurations:

* :edge_git:`Travis CI <blob/master/.travis.yml>`
* :edge_git:`GoCD <blob/master/tools/gocd/cruise-config.tmpl>`

Getting the Code
----------------
EDGE's sources are hosted at :edge_git:`GitHub <>`.
The repository has two branches :edge_git:`master <tree/master>` and :edge_git:`develop <tree/develop>`.
``master`` is the most recent stable version of EDGE.
The minmimum acceptance requirement for ``master`` is passing EDGE's :edge_dev_pub:`continuous delivery pipeline </chapters/cont/cont.html>`.
Periodically EDGE also provides tags, which are simply snapshots of the master branch.
Typically tags passed additional, manual testing.
We recommend to use the most recent tag to get started with EDGE.
``develop`` is the bleeding edge version of EDGE.
Changes in ``develop`` are intended to be merged into master.
However, ``develop`` is for ongoing development and broken from time to time.

The procedure for obtaining the code is as follows:

1. Clone the git-repository and navigate to the root-directory through:

   .. code-block:: bash

     git clone https://github.com/3343/edge.git
     cd edge

   This gives you the ``master``-branch.
   If this is what you want, jump over the next step.

2. Checkout the desired tag:

   .. code-block:: bash

     git checkout YY.MM.VV

   `Remark:` ``YY.MM.VV`` is a place holder.
   You have to replace this with your actual tag.
   Available tags are shown at the GitHub-homepage or directly through ``git tag``.

3. Initialize and update the submodules:

   .. code-block:: bash

     git submodule init
     git submodule update

LIBXSMM
-------
The single core backend of EDGE's high-performance kernels is provided through the library `LIBXSMM <https://github.com/hfp/libxsmm>`_.
LIBXSMM is optional, but highly recommended due to severe performance-limitations of the vanilla kernels.
Install LIBXSMM by running:

  .. code-block:: bash

    ./tools/build/deps/libxsmm.sh -i $(pwd)/submodules/libxsmm -o $(pwd)/deps

zlib
----
`zlib <http://zlib.net>`_ is a requirement for the HDF5 library.
Install zlib by running:

  .. code-block:: bash

    ./tools/build/deps/zlib.sh -o $(pwd)/deps

HDF5
----
`HDF5 <https://portal.hdfgroup.org/display/HDF5/HDF5>`_ is a requirement for point source descriptions and EDGE's mesh interface.
Install HDF5 by running:

  .. code-block:: bash

    ./tools/build/deps/hdf5.sh -z $(pwd)/deps -o $(pwd)/deps

MOAB
----
EDGE's mesh interface EDGE-V (see below) uses the library `MOAB <http://sigma.mcs.anl.gov/moab-library/>`_.
Install MOAB by running:

  .. code-block:: bash

    ./tools/build/deps/moab.sh -z $(pwd)/deps -5 $(pwd)/deps -e $(pwd)/submodules/eigen -i $(pwd)/submodules/moab -o $(pwd)/deps

METIS
----
`METIS <http://glaros.dtc.umn.edu/gkhome/metis/metis/overview>`_ is used for partitioning in EDGE-V.
Install METIS by running:

  .. code-block:: bash

    ./tools/build/deps/metis.sh -o $(pwd)/deps

EDGE-V
------
At runtime EDGE-V interfaces the mesh and respective annotations as a library.
Further details on preprocessing use-cases, e.g., the derivation of local time stepping schemes or mesh partitioning, are given in Sec. :doc:`../tools/edge_v`.
To install EDGE-V do the following:

1. Navigate to EDGE-V's source directory:

  .. code-block:: bash

    cd tools/edge_v

2. Run the build script:

  .. code-block:: bash

    scons parallel=omp zlib=../../deps hdf5=../../deps moab=../../deps metis=../../deps install_dir=../../deps

EDGE
----
EDGE (and EDGE-V) use `SCons <http://scons.org/>`_ as build tool.
``scons --help`` returns all of EDGE's build-options.
All build options are given in the respective :ref:`sub-section <sec-setup-config-build>` of Sec. :doc:`../setup/config`.
You can enable the libraries in EDGE either by passing their installation directory explicitly (recommended) or by setting the environment variables ``CPLUS_INCLUDE_PATH`` and ``LIBRARY_PATH``.
For example, let's assume that you installed LIBXSMM in the directory ``$(pwd)/deps``.
Than we could either enable LIBXSMM by passing ``xsmm=$(pwd)/deps`` to EDGE's SCons-script or by using ``CPLUS_INCLUDE_PATH=$(pwd)/deps/include LIBRARY_PATH=$(pwd)/deps/lib scons [...] xsmm=yes``.

If something goes wrong with finding a library, EDGE will tell you so.
For example, if we did not install LIBXSMM in ``/tmp``, but tell EDGE so anyways, we get:

.. code-block:: bash

    scons equations=elastic order=4 cfr=1 element_type=tet4 xsmm=/tmp
    [...]
    Checking for C++ static library libxsmmnoblas..no
      Warning: Could not find libxsmm, continuing without.

Further information on what went wrong is logged in the file ``config.log``, which, in this case, shows that the compiler could not find the LIBXSMM-header:

::

    [...]
    scons: Configure: Checking for C++ static library libxsmmnoblas..
    .sconf_temp/conftest_2.cpp <-
      |#include <libxsmm.h>
      |int main(int i_argc, char **i_argv) { return 0; }
    g++ -o .sconf_temp/conftest_2.o -c -std=c++11 -Wall -Wextra -Wno-unknown-pragmas -Wno-unused-parameter -Werror -pedantic -Wshadow -Wundef -O2 -ftree-vectorize -DPP_N_CRUNS=1 -DPP_T_EQUATIONS_ELASTIC -DPP_T_ELEMENTS_TET4 -DPP_ORDER=4 -DPP_PRECISION=64 -I. -Isrc -I/tmp/include .sconf_temp/conftest_2.cpp
    .sconf_temp/conftest_2.cpp:1:21: fatal error: libxsmm.h: No such file or directory
    compilation terminated.
    scons: Configure: no
