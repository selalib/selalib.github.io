# Naming Convention

## General guidelines

-   Prefer long variable names;
-   Do not use the same name for different entities;
-   Lower case: Fortran keywords, variables, procedures, modules, filenames;
-   Upper case: preprocessor macros;
-   Exposed interface must always begin with `sll_`.

## Exposed Fortran interface

When any SeLaLib Fortran entity is public and exposed to other modules,
its name must be chosen according to this rule:

-   `sll_m_[..]` for modules;
-   `sll_c_[..]` for abstract classes;
-   `sll_t_[..]` for types;
-   `sll_s_[..]` for subroutines (not attached to a type);
-   `sll_f_[..]` for functions (not attached to a type);
-   `sll_i_[..]` for procedure interfaces (defined in an 'abstract
    interface' block).

No strict rule is enforced for private entities in a module.
Nevertheless, we recommend using `m_[..]`, `c_[..]`, `t_[..]`, `s_[..]`,
`f_[..]`, and `i_[..]` where appropriate.

## Files and directories

SeLaLib is a collection of smaller libraries, each one defined in a
separate (sub)directory. Each library may be composed of multiple
Fortran modules, and each of these is defined in a separate source file.
In order to reflect this logic, we must:

- Define one module per file and give both the same name: the module
  name is `sll_m_[MODNAME]` and the file name is
  `sll_m_[MODNAME].F90`;
- Define one library per (sub)directory and give both the same name:
  the directory name is `LIBNAME` and the library name is
  `sll_[LIBNAME]`;
- In each (sub)directory define the 'interface module' `sll_[LIBNAME]`
  (contained in `sll_[LIBNAME].F90`) that acts as a unique 'entry
  point' to the library: here we declare all public variables in the
  library by means of multiple `use sll_m_[MODNAME], only :: [..]`
  statements, with a very short inline comment about each exposed
  name.

We note that each (sub)directory in SeLaLib should contain exactly one
'CMakeLists.txt' file, and that this file should contain exactly one
'ADD_LIBRARY($\dots$)' instruction. The compiled library will have name
`libsll_[LIBNAME].a` (if static) or `libsll_[LIBNAME].so` (if shared).

## Simulations

SeLaLib simulations are independent of each other, but they naturally
depend on many SeLaLib libraries. A simulation usually consists of:

1. A module containing a new simulation type that is derived from an
   abstract simulation class (methods exist for initializing from file
   and running the numerical simulation);

2. One or more programs that instantiate a simulation object of the new
   type, initialize it, and run the numerical simulation;

3. One or more input files.

We define some additional naming conventions that are specific to the
simulations:

- The simulation name must follow the pattern\
  `SIMNAME = [METHOD]_[DIMC]d[DIMV]v_[EQN]_[COORDS]_[EXTRA]`, where
  - `METHOD` describes the class of numerical method employed
    (e.g. `pic` for particle-in-cell, `sl` for semi-Lagrangian);
  - `DIMC` is the number of dimensions in configuration space;
  - `DIMV` is the number of dimensions in velocity space;
  - `EQN` is the equation that is solved (e.g. `adv` for advection,
    `vp` for Vlasov-Poisson, `vm` for Vlasov-Maxwell and );
  - `COORDS` is the type of coordinates used (e.g. `cart` for a
    Cartesian grid, `polar` for a polar grid, and `unstr` for an
    unstructured grid);
  - `EXTRA` is an additional description that distinguishes this
    specific simulation from the others;
- The simulation module must be named `sll_m_sim_[SIMNAME]`, and the
  file containing it must be named `sll_m_sim_[SIMNAME].F90` as usual;
- The simulation directory must be named
  - `src/simulations/sequential/[SIMNAME]/` for sequential simulations;
  - `src/simulations/parallel/[SIMNAME]/` for parallel simulations;
- The executable should have name :
  - Program name is `sim_SIMNAME`;
  - Program file name is `sim_SIMNAME.F90`;
  - Executable name is `sim_SIMNAME`;
  - CTest name (if any) is `sim_SIMNAME`.

## Unit tests

### General guidelines

- Always test against some a-priori solution, set a tolerance if
  needed;
- Each function in a module should be tested;
- If possible, a failing test should indicate the function that
  originated the error;
- Input parameters should be varied across the widest possible
  admissible range;
- Tests execution time should be as short as possible (seconds, not
  minutes!);
- Exactly 1 CTest per module, with name `[MODNAME]`;
- All files connected to tests should be inside of a library
  subdirectory called `[LIBNAME]/testing/[MODNAME]/`

### Naming convention 

- Fortran file name is `test_[TESTNAME].F90`;
- Fortran program name is `test_[TESTNAME]`;
- Executable name is `test_[TESTNAME]`;
- CTest name is `[TESTNAME]`;

NOTE: In the future, only one CTest should exist for each module, and
this CTest may call multiple executables (see guidelines in previous
section); in such a situation we should have `[TESTNAME]=[MODNAME]`.
