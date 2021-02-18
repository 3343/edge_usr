EDGE-V
======
EDGE-V is a tool prepare meshes for use in the core solver EDGE.
It includes the following key features:

* Parsing of velocity models and performing respective mesh annotations.
* Derivation of background fields with target-edge lengths for problem-aware meshing.
* Derivation of optimal Local Time Stepping (LTS) groups.
* LTS-aware partitioning.
* Mesh reordering based on the partitioning and time step groups.
* Splitting of the mesh into separate per-partition chunks and derivation of respective annotations, e.g., the communication structure, for use in EDGE.

Similar to the core-solver, EDGE-V is controlled by XML-configurations.
This part of the documentation has to be written still.
The :edge_opt:`La Habra mesh setups <bench/seismic/wp/la_habra/meshes/volume>` might serve as an example for seismic settings.