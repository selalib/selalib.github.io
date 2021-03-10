# Basic Utilities

## Numeric Types

### Description

As a *convenience*, Selalib offers aliases to some of Fortran's basic
numeric types. This is intended to:

1.  concisely and uniformly make clear the intended precision of a given
    variable,

2.  permit mixed precision representations when needed (for example, a
    developer could wish to represent a particular real number with a
    combination of a 32-bit integer and a 32-bit real instead of a
    single 64-bit real as is sometimes done in ultra-high performance
    software), and

3.  provide a centralized location to change the precision of a
    numerical representation program-wide.

### Exposed Interface

Selalib's numeric precision features are accessed through the following
aliases:

| alias         |shorthand for\...
| --------------|-------------------
| `sll_int32`   |`integer(kind=i32)`
| `sll_int64`   |`integer(kind=i64)`
| `sll_real32`  |`real(kind=f32)`
| `sll_real64`  |`real(kind=f64)`
| `sll_int`     |`integer(kind=user)`
| `sll_real`    |`real(kind=user)`
| `sll_comp32`  |`complex(kind=f32)`
| `sll_comp64`  |`complex(kind=f64)`

Where the kind type parameters `i32`, `i64`, `f32` and `f64` have been
defined to give a representation that is at least the denoted size for a
given number. `user` is available for a flexible kind type parameter.
These kind type parameters can also be used to specify the precision of
numerical constants in the usual Fortran way, i.e. `3.14159265_f32`. The
use of these constants is encouraged for development within the library
as then we can in principle change the numeric precision library-wide if
needed with only a parameter change.

### Usage

To use the module in a stand-alone way, use the line:

```c
#include "sll_working_precision.h"
```

These types are to be used like the native Fortran types that they are
aliasing:

## Memory Allocator

### Description

Selalib's memory allocators are simple wrappers around Fortran's native
allocators. We ask very little from these allocators:

1. to allocate memory,
2. to initialize it if requested (only for Fortran-native types),
3. to deallocate memory, and
4. to fail with as descriptive a message as possible.

### Exposed Interface

The interface to these allocators follows very closely the interface of
Fortran's `allocate()` and `deallocate()` functions, except for a
mandatory integer variable argument which does not need to be
initialized. The user may decide to allocate arrays or pointers of any
type, and up to the same number of dimensions as permitted by a given
Fortran implementation. The exposed macros are:

```fortran
SLL_ALLOCATE( array_and_lims, ierr )
```

This is the basic memory allocator with the same syntax as Fortran's
native `allocate()` but with a required integer error parameter. Any
type and dimension that the native fortran allocator would accept can be
given as the argument `array_and_lims`.

```fortran
SLL_CLEAR_ALLOCATE( array_and_lims, ierr )
```

Same behavior as `SLL_ALLOCATE()` but also initializes the allocated
memory to zero. This works for Fortran native types only or for derived
types for which an assignment operator ($= 0$) has been defined.

```fortran
SLL_DEALLOCATE( array_and_lims, ierr )
```

Basic deallocator. It is a wrapper around Fortran's native
`deallocate()` function but also nullifies the pointer given as an
argument after deallocation. This is a difference between the native
Fortran routine and this wrapper, as we have thus needed to distinguish
between the call that deallocates a pointer and a simple array:

```fortran
SLL_DEALLOCATE_ARRAY( array, ierr )
```

The array deallocator differs from the previous in that it does not
attempt to nullify the array name after the deallocation is complete.
This and the previous macro could in principle be merged into one for
simplification.

```fortran
SLL_INIT_ARRAY( array, val )
```

While not really an allocator/deallocator, we expose this macro for
convenience in initializing an array with a given constant value. For
consistency, it may be decided to eliminate this from the interface.

### Usage

To use the memory module in stand-alone fashion, use the line:

```c
#include "sll_memory.h"
```

An example of the use of the module is:

```fortran
integer :: err
real, dimension(:), allocatable :: a
real, dimension(:,:,:), pointer :: b=>null()

SLL_ALLOCATE( a(5000), err )
SLL_CLEAR_ALLOCATE( b(1:4,1:3,1:2), err)
SLL_INIT_ARRAY(b,0)
```

When finding an error condition, the user is informed about the location
of the failing call:

```
Memory allocation Failure. Triggered in FILE tester.F90, 
in LINE:   35
STOP ERROR: test_error_code(): exiting program
```

## Assertions

### Description

This is a very small but very useful capability that permits the
developer to devise many sorts of defensive programming techniques. The
simple idea is that of declaring a condition that is expected to be
true, thus triggering a descriptive error if the condition is violated.
The assertions are only present while compiling the library in `DEBUG`
mode, thus any overhead associated with the test can be eliminated by
changing the type of compilation. For this reason, the types of tests
that should be permanent should not be implemented with the assert
facility.

### Exposed Interface

Wherever a specific condition needs to be asserted, simply write:

```fortran
SLL_ASSERT( logical_condition )
```

Thus the condition to be checked must be cast in the form of a logical
statement. Falseness of such statement would trigger an assertion error.

The assertions can be used liberally since they are controlled by a
`-DEBUG` flag at compilation time (`DEBUG` compilation mode). Absence of
this flag (i.e. in `RELEASE` compilation mode) will delete all calls to
`SLL_ASSERT()` from the code. Hence, assertion conditions can be used
during a debugging or testing phase without increasing the overhead in
production code.

### Usage

To use the *assertions* module in a stand-alone way, use the line:

```c
#include "sll_assert.h"
```

However, all the following steps are necessary:

An example may be the checking of in-range indices on a protected array:

```fortran
function get_val( a, i )

    sll_int32 :: get_val
    sll_int32, intent(in) :: i
    sll_int32, dimension(:), intent(in) :: a
    SLL_ASSERT( (i .ge. 1) .and. (i .le. size(a)) )
    get_val = a(i)

end function get_val
```

Which could produce a behavior such as:

```
$ ./unit_test
  The size of a is: 1000
  a(1) =    0
  a(117) =    0
  Here we ask for the value of a(1001):

  (i .ge. 1) .and. (i .le. size(a)) : Assertion 
  error triggered in file unit_test.F90 in line       25
  STOP :  ASSERTION FAILURE
```

Such way of stopping a program is much less uncomfortable than the
sinking feeling one has when the program stops as in:

```
$ ./unit_test
  Array values: 
  The size of a is: 1000
  a(1) =    0
  a(117) =    0
  Incident de segmentation (core dumped)
```

which of course doesn't even need to happen at the moment of the first
error.

### Developer notes

1. the `SLL_ASSERT()    ` macro needs some other macros to convert
   parameters into strings, but the way to accomplish this may vary
   depending on the compiler used. If problems are found, make sure
   that the appropriate macro is used in the `sll_assert.h` to activate
   the proper behavior of such macros. As an example, when using
   **gfortran** compiler, notice the role played by the `-DGFORTRAN`
   flag.

2. when using **gfortran**, use pass along the flag
   `-ffree-line-length-none` to prevent compilation error as some of
   the included macros may go beyond Fortrans 132 character limit. Use
   the equivalent flag when changing compilers. All of this should be
   set within CMake.

## Timing Utility

### Description

When a high-resolution timer is available the measurable time interval
can be a bit smaller than 2 $\mu s$.

### Exposed Interface

The module is imported by

```fortran
use sll_timer
```

or

```fortran
include sll_timer
```

which defines the type `sll_time_mark` which is meant to be treated as
an opaque object i.e. only interact with it through the methods defined
for it. An instance of this type can be used to measure time intervals
with the following routines:

```fortran
set_time_mark( mark)
time_elapsed_since( mark )
time_elapsed_between( mark_0, mark_1 )
```

`set_time_mark( mark )` 
: is a routine that takes an `sll_time_mark`
object and sets is internals to mark the time at the moment of the
routine call.

`time_elapsed_since( mark )` 
: is a function that takes a time mark as an
argument and returns the time, *in seconds* between the clock reading
contained in the passed argument and the moment at which the function
was called.

`time_elapsed_between( mark_0, mark_1 )` 
: is a function which takes two
time marks that must have been set before obviously and will return the
time elapsed between them.

### Usage

Here is an example of the use of this module:

```fortran
 1     program timer_test
 2     use sll_timer
 3     implicit none
 4     integer(8) :: i, j, k
 5     real(c_double), dimension(:), allocatable :: dt
 6     type(sll_time_mark) :: tmark
 7
 8     integer, parameter :: ITERATIONS = 1000
 9 
 10    allocate(dt(ITERATIONS))
 11
 12    do i=1,ITERATIONS
 13       call sll_set_time_mark(tmark)
 14       do k=1,100000    ! Just a delay
 15          j = i * i - i
 16       end do
 17       dt(i) = sll_time_elapsed_since(tmark) 
 18    end do
 19    ! print diagnostics here, averages, min, max, etc.
 20    end program
```

In brief:

Line 2:
:   Imports the timer module.

Line 5:
:   Here we define an array to store the results of the timing test.
    This array stores the return values of the function
    `time_elapsed_since()`, which returns a double precision value
    (`sll_real64`).

Line 6: 
:   Declaration of the time marker.

Lines 13:
:   Sets the time marker with a current clock reading.

Line 17:
:   We store the result of the timing operation.

Any number of time markers can be declared and set at different points.

## Constants

### Description

Selalib provides a single file that specifies commonly used constants as
read-only parameters that can be used library-wide or by users.

### Exposed Interface

Selalib simply defines a list of symbols, all in units of the
International System when applicable. To use, just write either

```fortran
use sll_constants
```

The symbols defined are:

sll\_pi 
: mathematical constant $\pi$

sll\_c  
: speed of light \[$m/s$\].

sll\_epsilon\_0 
: permittivity of free space ($\epsilon_0$) \[$F/m$\].

sll\_mu\_0 
: permeability of free space ($\mu_0$) \[$N/A^2$\].

sll\_e\_charge 
: fundamental charge \[$C$\].

sll\_e\_mass 
: mass of the electron \[$kg$\].

sll\_proton\_mass 
: mass of the proton \[$kg$\].

sll\_g 
: gravitational acceleration at sea level \[$m/s^2$\]

## Logical Meshes

### Description

Selalib solves its PDEs on structured grids[^1]. Furthermore, the actual
solutions of the equations are carried out on rectangular, uniform grids
(i.e. cartesian) which within Selalib are referred to as *logical
meshes*. This means that when dealing with *physical domains* (i.e. the
actual geometry that is to be modeled) there is a coordinate
transformation that maps the logical mesh to the physical domain[^2].
Such coordinate transformations will be discussed on a different
chapter. Here we occupy ourselves with the description of the logical
mesh itself. The specification of a logical mesh is quite simple, as at
a minimum it only needs the number of cells in each dimension and the
values of the endpoints. As a convention, the coordinates in the logical
meshes are always referred to by the symbol $\eta$. Thus the coordinate
for the 1D mesh is $\eta_1$, for the 2D mesh they are $\eta_1$ and
$\eta_2$ and so on.

[^1]: We use the terms *mesh* and *grid* interchangeably.

[^2]: Or multiple coordinate transformations, in case that the physical
    domain is represented by multiple patches.


### Exposed Interface

Public types:

```fortran
sll_logical_mesh_1d
sll_logical_mesh_2d
sll_logical_mesh_3d
sll_logical_mesh_4d
```

Routines:

```fortran
new_logical_mesh_1d( num_cells,         
                     eta1_min, 
                     eta1_max )
                                              
new_logical_mesh_2d( num_cells1, 
                     num_cells2, 
                     eta1_min, 
                     eta1_max, 
                     eta2_min, 
                     eta2_max )
                          
new_logical_mesh_3d( num_cells1, 
                     num_cells2,
                     num_cells3, 
                     eta1_min, 
                     eta1_max, 
                     eta2_min, 
                     eta2_max, 
                     eta3_min, 
                     eta3_max )
                         
new_logical_mesh_4d( num_cells1, 
                     num_cells2,
                     num_cells3,
                     num_cells4, 
                     eta1_min, 
                     eta1_max, 
                     eta2_min, 
                     eta2_max, 
                     eta3_min, 
                     eta3_max, 
                     eta4_min, 
                     eta4_max )          
                          
sll_display( mesh )  ! generic, for any mesh dimension      
delete( mesh )  ! generic, for any mesh dimension
```

The 'new' functions return pointers to allocated and initialized meshes.
All the parameters that specify the limits of the values of *eta* are
optional. These limits default to 0.0 and 1.0 for their minimum and
maximum values. The routine `sll_display( )` is helpful to show the
contents of a given mesh.

As a rare case in Selalib, the interface of the logical meshes permits
direct access to their internals. The slots available for each type are:

```
num_cellsX
etaX_min
etaX_max
delta_etaX
```

where 'X' stands for the index of the direction of interest (1, 2, 3,
etc.). Direct access is meant for reading, as the 'new' functions are
enough to produce duly initialized logical meshes. Thus a user can
directly access any of these slots as in:

```
mesh%delta_eta1
```

### Usage

```fortran
  1 program logical_meshes
  2 use sll_logical_meshes
  3  implicit none
  4
  5  type(sll_logical_mesh_2d), pointer :: m2d
  6  
  7  m2d => new_logical_mesh_2d(64, 64)
  8
  9  call sll_display(m2d)
  10
  11 call delete(m2d)
  12 end program logical_meshes
```

## File IO

### Description

Presently, Selalib offers facilities to output data in the XDMF file
format, readable by the VisIt program. To use, import the module
`sll_file_io`. The interface allows to open a file, write
multi-dimensional data in it and then close it. The main purpose of this
module is to provide the low-level building blocks that can be used by
higher-level functions to conveniently write data to files.

### Exposed Interface

The current interface is a collection of generic functions:

```fortran
sll_xdmf_open( file_name,
               mesh_name,
               nnodes_x1,nnodes_x2,[nnodes_x3,]
               file_id,
               error )
sll_xdmf_write_array( mesh_name,
                      array,
                      array_name,
                      error,
                      xmffile_id,
                      center )
sll_xdmf_close(file_id,error)
```

Where the parameters in brackets are used depending of the
dimensionality of the data.

`sll_xdmf_open( )` 
: opens a file with a given name and returns a file
identifier on `file_id`.

`sll_xdmf_write_array( )` 
: writes a given multi-dimensional array on a file identified by `xmffile_id`.

`sll_xdmf_close( )` 
: closes the given file.

```{note}
There is another library to manipulate xdmf files with object orienting approach.
[sll_xml](https://selalib.github.io/selalib/namespacesll__m__xml.html)
```

### Critical Commentary

ECG: This interface can be improved. To consider:

The job of `sll_xdmf_open( )` should be to open a file, with a given
filename; it should take as parameters some flags to indicate what
behavior to exercise in case of error or if the file exists (overwrite?
append?) and nothing else. The nnodes parameters are used to write
XDMF-related preambles, but this should be sent to the other functions
that actually write into the file.

- The mesh\_name parameter is not clear. Why mesh? 

  *Mesh coordinates are written in a separate file, so you can write
  it only one time at the beginning of the computation. When you plot
  a field you still need to link your data to a mesh.*

- What is the 'center' parameter in the write\_array() function? 

  *When you plot field you can specify if your values are node centered or
cell centered*
