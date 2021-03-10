# Libraries

The src folder contains all source files for the SeLaLib-library. It is
organized in the following subfolders containing the different building
blocks for kinetic and gyrokinetic simulations:

add\_ons
: Advanced features that extend various pieces of the library but there are no modules depending on these features.

data\_structures
: Definition of data structures and descriptors used throughout the library.

field\_solvers
: Solvers of the various field equations.

interfaces
: Modules that implement interfaces to external libraries.

interpolation
: Methods for numerical interpolation.

io
: Modules for input-output of the simulations.

linear\_solvers
: Methods to solve linear systems.

low\_level\_utilities
: Utilities for memory and error handling.

mesh
: Modules to define mesh parameters and coordinate
    transformations.

parallelization
: Modules providing routines for parallelization with MPI.

particle\_methods
: Modules implementing the building blocks for particle methods.

quadrature
: Numerical quadrature methods.

semi\_Lagrangian
: Modules specific to semi-Lagrangian methods.

simulation\_tools
: Tools to build simulations such as simulation base class and initialization of typical test problems.

splines
: Spline modules.

time\_solvers
: Methods for time stepping.

Simulation examples for various kinetic and gyrokinetic test cases are
found in the folder `selalib/simulations`.
