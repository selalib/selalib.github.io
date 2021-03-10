# Coding guidelines

## General Coding style

**Documentation**

-   Write brief documentation before implementation.

-   Please share and communicate your developments on the list.

-   If you comment out a piece of code, put a clear comment about the
    reason to do it. Otherwise is not clear why something is commented.

-   Document your code with doxygen (see {doc}`doxygen` for detailed instructions).

**Interfaces**

-   Don't forget interfaces on top of each modules.

-   Only these interfaces should be public.

-   Type bound functions should only be made public through the type.

-   Future updates will be much easier if we have a good interface.

-   End user do not have to change his code to get upgrade benefits.

**Layout**

-   Indentation: 2 blank spaces (not tabs) for all structures.

-   Maximum line length: 80 characters.

**Names**

-   Prefer explicit (even if long) names;

-   Do not use the same name for different entities;

-   Lower case: Fortran keywords, variables, procedures, modules,
    filenames;

-   Upper case: preprocessor macros;

-   Exposed interface must always begin with `sll_`.

**Fortran**

-   All modules must be declared private and implicit none.

-   Always give the explicit name for optional arguments.

-   Variables must be initialized to zero if required.

-   Functions returning arrays should only be used if the array is
    small.

-   Avoid pointer dummy arguments unless strictly necessary.

**Assertions** Add sanity checks using `SLL_ASSERT`. Examples: Check
correct size of input/output arguments, check function error codes\...

## Procedure arguments definition

-   One variable per line.

-   The intent and the :: in the declaration must be aligned.

-   It must follow the same order that in the declaration.

-   The intent must be always declared.

-   A blank line must separate the declaration of local variables.

## Naming conventions for exposed Fortran interfaces {#sec:exposed_names}

When any SeLaLib Fortran entity is public and exposed to other modules,
its name must be chosen according to this rule:

-   `sll_m_[..]` for modules;

-   `sll_c_[..]` for abstract types;

-   `sll_t_[..]` for non-abstract types;

-   `sll_s_[..]` for subroutines (not attached to a type);

-   `sll_f_[..]` for functions (not attached to a type);

-   `sll_p_[..]` for parameters defining descriptors/flags;

-   `sll_v_[..]` for module variables;

-   `sll_o_[..]` for Fortran interfaces (overloading of
    subroutines/functions);

-   `sll_i_[..]` for procedure interfaces (defined in an 'abstract
    interface' block).

No strict rule is enforced for private entities in a module.
Nevertheless, we recommend using `m_[..]`, `c_[..]`, `t_[..]`, `s_[..]`,
`f_[..]`, `p_[..]`, `v_[..]`, `o_[..]` and `i_[..]` where appropriate.

## Files and directories

SeLaLib is a collection of smaller libraries, each one defined in a
separate (sub)directory. Each library may be composed of multiple
Fortran modules, and each of these is defined in a separate source file.
In order to reflect this logic, we must:

-   Define one module per file and give both the same name: the module
    name is `sll_m_[MODNAME]` and the file name is
    `sll_m_[MODNAME].F90`;

-   Define one library per (sub)directory and give both the same name:
    the directory name is `LIBNAME` and the library name is
    `sll_[LIBNAME]`;

-   In each (sub)directory define the 'interface module' `sll_[LIBNAME]`
    (contained in `sll_[LIBNAME].F90`) that acts as a unique 'entry
    point' to the library: here we declare all public variables in the
    library by means of multiple `use sll_m_[MODNAME], only :: [..]`
    statements, with a very short inline comment about each exposed name
    (see § [1.5](#sec:library_interfaces){reference-type="ref"
    reference="sec:library_interfaces"}).

We note that each (sub)directory in SeLaLib should contain exactly one
'CMakeLists.txt' file, and that this file should contain exactly one
'ADD_LIBRARY($\dots$)' instruction. The compiled library will have name
`libsll_[LIBNAME].a` (if static) or `libsll_[LIBNAME].so` (if shared).

## Types and classes

Each type should have

-   a constructor called `init` (type-bound) or `sll_s_<type_name>_init`
    (non type-bound) being a subroutine, and

-   a destructor called `free` (type-bound) or `sll_s_<type_name>_free`
    (non type-bound) being a subroutine with a signature that has no
    additional arguments, and

-   other procedures that are called `<procedure_name>` (type-bound) or\
    `sll_<s/f>_<type_name_<procedure_name>` (non type-bound).

When implementing these procedures accompanying the type, the instance
of the type should be call `self`. In case the type is not polymorphic,
it is usually better to use the non type-bound option to avoid overhead
due to polymorphism introduced by the `class` keyword. If procedures are
defined as type-bound they should only be called type-bound.

**Note**: The old interfaces `sll_o_delete`, `sll_o_new`, etc. should
not be used any more. In general, interfaces should only be used with
care and only within a single module.

Abstract types should be used whenever we have several specific
implementations for the same problem (interpolators, field solvers,
etc.). If you want to avoid polymorphism for performance reason, you
should still try follow the signature of the abstract type.

In order to initialize an abstract type, two options are exemplified below.
In general, allocatables should be preferred over pointers. The `sll_f_new_<type_name>` functions returning
a pointer should be avoided in future.

(class_init)=
```fortran
class(sll_c_example), <allocatable or pointer> :: example

allocate( sll_t_example :: example )
select type ( example )
type is ( sll_t_example )
  call example%init( <special signature> )
end

call example%solve( <signature> )

call example%free()
deallocate( example )
```

```fortran
class(sll_c_example), allocatable, target :: specific
type(sll_t_example), pointer              :: example

allocate( specific )
call specific%init( <special signature> )
example => specific

call example%solve( <signature> )

call example%free()
deallocate( example )
```

**Note**: If your program just uses one specific instance of an abstract
type, directly declare this specific type as
`type(sll_t_example) :: example` for better performance.

**Optional performance improvement**: If you have a type that is derived
from another type but not extended itself, you can use a `select type`
statement within the declaration of type-bound procedures in order to
reduce the overhead due to polymorphism as shown here.


```fortran
subroutine sll_s_example_solve( self )
  class( sll_t_example ), intent ( in ) :: self
  
  select type( self )
  type is ( sll_t_example )

	<instructions>  
  
  end 

end
```

## SeLaLib library interfaces

SeLaLib contains many libraries (=directories), each of which is
comprised of several modules (=files). We propose to expose the public
entities in a library through one single 'interface module', which will
act as a single entry point to the library:

-   The module name is `sll_[LIBNAME]`;

-   The file name is `sll_[LIBNAME].F90`;

-   The public names from all modules in a library are exposed by means
    of multiple `use sll_m_[MODNAME], only :: [..]` statements;

-   Each exposed name is given a short inline comment.

As an example, the interface module should look like
 
```fortran
#include "sll_working_precision.h"
#include "sll_assert.h"

use sll_m_[MODNAME_1], only: &
  sll_t_[TYPENAME],  &
  sll_i_[INTRFNAME], &
  ...
use sll_m_[MODNAME_2], only: &
  sll_c_[CLASSNAME], &
  sll_s_[SUBNAME],   &
  ...
implicit none
public :: &
  sll_t_[MY_TYPENAME], &
  sll_s_[MY_SUBNAME],  &
  ...
private
```


There are several advantages to this approach:

-   In order to use one of the SeLaLib libraries, a user will find all
    exposed names in a single file/module;

-   The developer may easily switch between different versions of the
    same function/subroutine/type/etc. without effecting the user code,
    by using the 'renaming syntax' `[local_name] => [use_name]`;

-   When using SeLaLib libraries, only one '.mod' file per directory is
    needed at compilation time .

## SeLaLib module interfaces

The first part of a module should be seen as an 'interface', clearly
stating what names are imported from other modules and what names are
exposed to the outside. For this purpose, we should:

-   Always use the 'only' clause in a 'use' statement;

-   Insert an 'implicit none' statement before any variable declaration;

-   Specify all public entities in the module in a unique 'public'
    statement;

-   Follow the public statement with a blanket 'private' statement.

For example, the first part of a module should look like the code
snippet in
Figure [\[fig:module_interface\]](#fig:module_interface){reference-type="ref"
reference="fig:module_interface"}.

## SeLaLib header files

If needed, a 'header file' `sll_[LIBNAME].h` may be created inside a
`[LIBNAME]` SeLaLib directory. This file should only contain
preprocessor instructions, e.g.

-   User-defined variable types;

-   Preprocessor macro definitions.

A header may also contain some `use` statements, because the macros may
be based on user-defined functions and types from other modules. (Note
that in general, unless strictly needed, `use` statements in a header
file should be avoided.)

## Simulations

### Structure

SeLaLib simulations are independent of each other, but they naturally
depend on many SeLaLib libraries. A simulation usually consists of:

1.  A module containing a new simulation type that is derived from an
    abstract simulation class (methods exist for initializing from file
    and running the numerical simulation);

2.  One or more programs that instantiate a simulation object of the new
    type, initialize it, and run the numerical simulation;

3.  One or more input files.

### Naming conventions 

We define some additional naming conventions that are specific to the
simulations:

-   The simulation name must follow the pattern\
    `SIMNAME = [METHOD]_[EQN]_[DIMC]d[DIMV]v_[COORDS]_[EXTRA]`, where

    -   `METHOD` describes the class of numerical method employed
        (e.g. `pic` for particle-in-cell, `pif` for particle-in-Fourier,
        `bsl` for advective (backward) semi-Lagrangian, `fsl` for
        forward semi-Lagrangian, `csl` for conservative backward
        semi-Lagrangian, `eul` for Eulerian);

    -   `EQN` is the equation that is solved (e.g. `ad` for advection
        with given field, `vp` for Vlasov-Poisson, `vm` for
        Vlasov-Maxwell, `va` for Vlasov-Ampere, `dk` for drift kinetic,
        `gk` for gyrokinetic and `gc` for guiding center);

    -   `DIMC` is the number of dimensions in configuration space;

    -   `DIMV` is the number of dimensions in velocity space (if no put
        0v);

    -   `COORDS` is the type of coordinates used (e.g. `cart` for a
        Cartesian grid, `polar` for a polar grid, and `unstr` for an
        unstructured grid, `hex` for a hexagonal grid, `curv` for
        curvilinear from cartesian, `hexcurv` for curvilinear from
        hexagonal);

    -   `EXTRA` is an additional description (short!) that distinguishes
        this specific simulation from the others; in particular, for
        Eulerian solvers specify the method (`dg` for discontinuous
        Galerkin, `fv` for finite volume, `fe` for continuous finite
        elements, `ft` for Fourier); add `serial` to a serial simulation
        if there is a parallel simulation for the same test case;

-   The simulation module must be named `sll_m_sim_[SIMNAME]`, and the
    file containing it must be named `sll_m_sim_[SIMNAME].F90` as usual;

-   The simulation directory must be named

    -   `simulations/serial/[SIMNAME]/` for serial simulations;

    -   `simulations/parallel/[SIMNAME]/` for parallel simulations;

-   Naming conventions for executables:

    -   Program file name is `sim_[SIMNAME].F90`;

    -   Program name is `sim_[SIMNAME]`;

    -   Executable name is `sim_[SIMNAME]`;

    -   CTest name (if any) is `sim_[SIMNAME]`.

## Unit tests

### General guidelines

-   Always test against some a-priori solution, set a tolerance if
    needed;

-   Each function in a module should be tested;

-   If possible, a failing test should indicate the function that
    originated the error;

-   Input parameters should be varied across the widest possible
    admissible range;

-   Tests execution time should be as short as possible (seconds, not
    minutes!);

-   Exactly 1 CTest per module, with name `[MODNAME]`, but in addition
    there can be tests that check the combination of two modules;

-   All files connected to tests should be inside of a library
    subdirectory called `[LIBNAME]/testing`. If there are auxiliary
    files for the tests, they should be within a subdirectory
    `[LIBNAME]/testing/[MODNAME]/`.

### Naming convention

-   Fortran file name is `test_[TESTNAME].F90`;

-   Fortran program name is `test_[TESTNAME]`;

-   Executable name is `test_[TESTNAME]`;

-   CTest name is `[TESTNAME]`.

NOTE: In the future, only one CTest should exist for each module, and
this CTest may call multiple executables (see guidelines in previous
section); in such a situation we should have `[TESTNAME]=[MODNAME]`.

### Examples

-   The module `sll_m_io_utilities` provides functions to handle input
    and/or reference files in testing.

-   An example how to use a reference file to validate the results can
    be found in the test `test_xml`.

-   To test with a set of different parameter values, a python script
    can be used. For an example, see the test `test_ode_integrator`.
