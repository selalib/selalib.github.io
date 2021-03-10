# Mid-Level Layer: Numerical and Parallel Utilities

## Tridiagonal System Solver

### Description

To solve systems of the form $Ax=b$, where $A$ is a tridiagonal matrix,
Selalib offers a native, robust tridiagonal system solver. The present
implementation contains only a serial version. The algorithm is based on
an *LU* factorization of a given matrix, with row pivoting. The
tridiagonal matrix must be given as a single array, with a memory layout
shown next. $$\begin{bmatrix}
    a(2) & a(3) &        &        &        &        &  a(1) \\
    a(4) & a(5) & a(6)   &        &        &        &       \\
         & a(7) & a(8)   & a(9)   &        &        &       \\
         &      & \ddots & \ddots & \ddots &        &       \\
         &      &        & \ddots & \ddots & \ddots &       \\
         &      &        &        & a(3n-5)& a(3n-4)&a(3n-3)\\
    a(3n)&      &        &        &        & a(3n-2)&a(3n-1)\\
  \end{bmatrix}$$

### Exposed Interface

Factorization of the matrix $A$ is obtained through a call to the
subroutine

         sll_setup_cyclic_tridiag( a, n, lu, ipiv )

where `a` is the matrix to be factorized, `n` is the problem size (the
number of unknowns), `lu` is a real array of size $7n$ where
factorization information will be returned and `ipiv` is an integer
array of length `n` on which pivoting information will be returned. From
the perspective of the user, `lu` and `ipiv` are only arrays that
`sll_setup_cyclic_tridiag` requires and do not need any further
consideration.

The solution of a tridiagonal system, once the original array $A$ has
been factorized, is obtained through a call to

         sll_solve_cyclic_tridiag( lu, ipiv, b, n, x )

where `lu`, `ipiv` are the arrays returned by
`sll_setup_cyclic_tridiag()`, `b` is the independent term in the
original matrix equation, `n` is the system size and `x` is the array
where the solution will be returned.

### Usage

To use the module in a stand-alone way, include the line:

         use sll_tridiagonal

The following code snippet is an example of the use of the tridiagonal
solver.

```fortran
         sll_int32 :: n = 1000
         sll_int32 :: ierr
         sll_real64, allocatable, dimension(:) :: lu
         sll_int32,  allocatable, dimension(:) :: ipiv
         sll_real64, allocatable, dimension(:) :: x

         SLL_ALLOCATE( lu(7*n), ierr )
         SLL_ALLOCATE( ipiv(n), ierr )
         SLL_ALLOCATE( x(n), ierr )
         
         ! initialize a(:) with the proper coefficients here... 
         and then:
         
         sll_setup_cyclic_tridiag( a, n, lu, ipiv )
         sll_solve_cyclic_tridiag( lu, ipiv, b, n, x )
         
          SLL_DEALLOCATE_ARRAY( lu, ierr )
          SLL_DEALLOCATE_ARRAY( ipiv, ierr )
          SLL_DEALLOCATE_ARRAY( x, ierr )
```

Note that if the last call had been made as in

```
         sll_solve_cyclic_tridiag( lu, ipiv, b, n, b )
```

the system would have been solved in-place.

### Status

Unit-tested.

## Boundary Condition Descriptors

### Description

This tiny module defines a few symbols that are meant to be used
library-wide as a means to indicate various boundary condition types or
their combinations. The objective is to have pre-defined symbols that
will prevent the need of using numeric values explicitly to specify this
information.

### Exposed Interface

The defined symbols are:

```fortran
         SLL_USER_DEFINED
         SLL_PERIODIC
         SLL_DIRICHLET
         SLL_NEUMANN 
         SLL_HERMITE 
         SLL_NEUMANN_MODE_0
         SLL_SET_TO_LIMIT 
```

### Usage

Simply pass the parameter as an argument to any routine call which
requires a *boundary condition descriptor*.

## Cubic Splines

### Description

The cubic splines module provides capabilities for 1D and 2D data
interpolation with cubic B-splines and different boundary conditions (at
the time of this writing: periodic, hermite). The data to be
interpolated are represented by a simple array. The spline coefficients
and other information are stored in a spline object, which is also used
to interpolate the fitted data. To use this module, simply declare an
instance of the type to be used, initialize the object with the desired
parameters, use the initialized object to do interpolations and when no
longer needed, release the resources through a call to the
`sll_delete( )` routine. More details below.

### Exposed Interface

Fundamental types:

```fortran
sll_cubic_spline_1d
sll_cubic_spline_2d
```

These types are declared as pointers and only manipulated through the
functions and subroutines described below. For more explicit examples,
see the usage section.

For the 1D case, the available routines are:

```fortran
new_cubic_spline_1D( num_points, 
                     xmin, 
                     xmax, 
                     bc_type, 
                     [slope_L], 
                     [slope_R] )  
compute_cubic_spline_1D( data, spline_object )
interpolate_value( x, spline_object )
interpolate_derivative( x, spline_object )
interpolate_array_values( a_in, a_out, np, spline_object )
interpolate_pointer_values( a_in, a_out, np, spline_object )
interpolate_array_derivatives( a_in, a_out, np, spline_object )
interpolate_pointer_derivatives( ptr_in, ptr_out, np, spline_object )
get_x1_min( spline_object )
get_x1_max( spline_object )
get_x1_delta( spline_object )
sll_delete( spline_object )
```

In the above list:

::: 
`new_cubic_spline_1D()` is responsible for allocating all the necessary
storage for the spline object and initializating it. Arguments:

num_points:

:   32-bit integer. Number of data points for which the spline needs to
    be computed, including the end-points (regardless of the boundary
    condition used).

xmin:

:   Double precision real. Lower bound of the domain in which the data
    array is defined. In other words, if we think of the data array
    (indexed 1:NP) as the values of a discrete function *f* defined over
    a sequence of $x_i$'s, then $x_1 = xmin$.

xmax:

:   Double precision real. Similarly to `xmin`, `xmax` represents the
    maximum extent of the domain. For data indexed 1:NP:
    $xmax = x_{NP}$.

bc_type:

:   Pre-defined parameter. Descriptor of the type of boundary condition
    to be imposed in the spline. Use only the symbols defined in
    Selalib's *boundary condition descriptors* module. Presently, one of
    `PERIODIC_SPLINE` or `HERMITE_SPLINE`. These are really aliases to
    integer flags, but in Selalib we avoid the use of non-descriptive
    flags.

slope_L:

:   Double precision real. Optional value representing the desired slope
    at $x = xmin$, in the hermite BC case.

slope_R:

:   Double precision real. Optional value representing the desired slope
    at $x = xmax$, in the hermite BC case.

`compute_cubic_spline_1D()` calculates the spline coefficients needed to
represent the given data and stores this information in the spline
object. Arguments:

 data:

:   Double precision 1D real array containing NP values.

 spline_object: 

:   Initialized object of type `sll_cubic_spline_1d`.

`interpolate_value()` returns the value of *f(x)* where *x* is a
floating-point value between `xmin` and `xmax`, and *f* is a continuous
function built with cubic B-splines and the user-defined boundary
conditions contained in the spline object passed. Essentially, the
spline object, together with the `interpolate_value()` function create
the illusion of having available a continuous function when originally
using the discrete data given. Arguments:

 x:

:   Double precision value representing the ordinate whose image is
    sought.

 spline_object: 

:   Initialized object of type `sll_cubic_spline_1d`. A call to
    `compute_cubic_spline_1D()` should have been made previous to
    calling this function.

`interpolate_array_values()` has an analogous functionality than the
previous interpolation function, but is capable of processing whole
arrays. This is useful to avoid the overhead associated with a call to
`interpolate_value()` inside a loop.

`interpolate_derivative()` returns the value of *f'(x)* where *x* is a
floating-point value between `xmin` and `xmax`, and *f* is a continuous
function built with cubic B-splines and the user-defined boundary
conditions contained in the spline object passed. Same types of
arguments as in `interpolate_value()`.

`interpolate_array_values()` is a subroutine that interpolates the
results of multiple values of *x*. Arguments:

a_in:

:   \[in\] double precision 1D array containing the values of the
    ordinates to be interpolated.

a_out:

:   \[out\]double precision 1D array to store the results, i.e., the
    images of the values contained in `a_in`.

np:

:   \[in\] 32-bit integer storing the number of points meant to be
    interpolated.

 spline_object: 

:   \[inout\] Initialized object of type `sll_cubic_spline_1d`.

`interpolate_pointer_values()` is a subroutine with analogous arguments
to `interpolate_array_values` except that it receives pointers instead
of arrays for its arguments.

`interpolate_array_derivatives()` computes multiple first-derivative
values. Its interface is identical to `interpolate_array_values()`.

`interpolate_pointer_derivatives()` computes multiple first-derivative
values, just as `interpolate_array_derivatives()` except that its
arguments are pointers.

`get_x1_min()` returns the minimum value of the domain in which the
spline object is defined. Arguments:

spline_object:

:   a pointer to a previously initialized object of type
    `sll_cubic_spline_1D`

`get_x1_max()` returns the maximum value of the domain in which the
spline object is defined. Arguments:

spline_object:

:   a pointer to a previously initialized object of type
    `sll_cubic_spline_1D`

`get_x1_delta()` returns the value of the spacing between the points
used to compute the spline values. This is determined from the extent of
the domain and the number of cells used. Arguments:

spline_object:

:   a pointer to a previously initialized object of type
    `sll_cubic_spline_1D`

`sll_delete()` subroutine to free the resources of a given spline
object. Arguments:

spline_object:

:   a pointer to a previously initialized object of type
    `sll_cubic_spline_1D`
:::

For the 2D case, the available routines are:

```fortran
new_cubic_spline_2d( num_pts_x1,  
                     num_pts_x2,  
                     x1_min,      
                     x1_max,     
                     x2_min,      
                     x2_max,     
                     x1_bc_type, 
                     x2_bc_type,  
                     const_slope_x1_min,
                     const_slope_x1_max,
                     const_slope_x2_min, 
                     const_slope_x2_max,
                     x1_min_slopes,
                     x1_max_slopes,
                     x2_min_slopes, 
                     x2_max_slopes )
compute_cubic_spline_2d( data, spline_object )
interpolate_value_2d( x1, x2, spline_object )
interpolate_x1_derivative_2d( x1, x2, spline_object )
interpolate_x2_derivative_2d( x1, x2, spline_object )      
get_x1_min( spline_object )
get_x1_max( spline_object )
get_x2_min( spline_object )
get_x2_max( spline_object )
get_x1_delta( spline_object )
get_x2_delta( spline_object )
sll_delete( spline_object )             
```

In the above list:

::: 
`new_cubic_spline_2D()` is responsible for allocating all the necessary
storage for the spline object and initializating it. Arguments:

num_points_x1:

:   \[in\] 32-bit integer. Number of data points in the first
    (memory-contiguous) direction for which the spline needs to be
    fitted. Endpoints at $x_{1,j} = x1_{min}$ and
    $x_{NPX1,j} = x1_{max}$ must be included.

num_points_x2:

:   \[in\] 32-bit integer. Number of data points in the second direction
    for which the spline needs to be fitted. Endpoints at
    $x_{i,1} = x2_{min}$ and $x_{i,NPX2} = x2_{max}$ must be included.

x1_min:

:   \[in\] Double precision real. Lower bound of the domain in which the
    data array is defined in the first (memory-contiguous) direction.

x1_max:

:   \[in\]Double precision real. Represents the maximum extent of the
    domain in the first (memory-contiguous) direction.

x2_min:

:   \[in\] Double precision real. Lower bound of the domain in which the
    data array is defined in the second direction.

x2_max:

:   \[in\]Double precision real. Represents the maximum extent of the
    domain in the second direction.

x1_bc_type:

:   \[in\] Pre-defined parameter. Descriptor of the type of boundary
    condition to be imposed in the spline in the first direction.
    Presently, one of `PERIODIC_SPLINE` or `HERMITE_SPLINE` as defined
    in Selalib's *boundary condition descriptors* module.

x2_bc_type:

:   \[in\] Pre-defined parameter. Descriptor of the type of boundary
    condition to be imposed in the spline in the second direction.
    Presently, one of `PERIODIC_SPLINE` or `HERMITE_SPLINE` as defined
    in Selalib's *boundary condition descriptors* module.

const_slope_x1_min:

:   \[optional, in\]Double precision real. Optional value representing
    the desired slope at $x1_{min}$, in the hermite BC case if a
    constant value for the whole edge is to be specified.

const_slope_x1_max:

:   \[optional, in\]Double precision real. Optional value representing
    the desired slope at $x1_{max}$, in the hermite BC case if a
    constant value for the whole edge is to be specified.

const_slope_x2_min:

:   \[optional, in\]Double precision real. Optional value representing
    the desired slope at $x2_{min}$, in the hermite BC case if a
    constant value for the whole edge is to be specified.

const_slope_x2_max:

:   \[optional, in\]Double precision real. Optional value representing
    the desired slope at $x2_{max}$, in the hermite BC case if a
    constant value for the whole edge is to be specified.

x1_min_slopes:

:   \[optional, in\] Array with the values of the slopes at each of the
    `num_points_x2` locations in the edge at $x1_{min}$. To be used with
    the hermite boundary condition only.

x1_max_slopes:

:   \[optional, in\] Array with the values of the slopes at each of the
    `num_points_x2` locations in the edge at $x1_{max}$. To be used with
    the hermite boundary condition only.

x2_min_slopes:

:   \[optional, in\] Array with the values of the slopes at each of the
    `num_points_x1` locations in the edge at $x2_{min}$. To be used with
    the hermite boundary condition only.

x2_max_slopes:

:   \[optional, in\] Array with the values of the slopes at each of the
    `num_points_x1` locations in the edge at $x2_{max}$. To be used with
    the hermite boundary condition only.

`compute_cubic_spline_2D()` calculates the spline coefficients needed to
represent the given 2D data and stores this information in the spline
object. Arguments:

 data:

:   \[in\] Double precision 2D real array containing NPX1\*NPX2 values.

 spline_object: 

:   Initialized object of type `sll_cubic_spline_2d`.

`interpolate_value_2d()` returns the value of *f(x1,x2)* where the
ordered pair $(x1,x2)$ is in the domain
$[x1_{min},x1_{max}] X [x2_{min},x2_{max}]$. *f* is a continuous
function built with cubic B-splines and the user-defined boundary
conditions contained in the spline object passed. Arguments:

 x1:

:   \[in\] Double precision value representing the value of the first
    coordinate.

 x2:

:   \[in\] Double precision value representing the value of the second
    coordinate.

 spline_object: 

:   Initialized object of type `sll_cubic_spline_2d`. A call to
    `compute_cubic_spline_2D()` should have been made previous to
    calling this function.

`interpolate_x1_derivative_2d()` returns the value of the first
derivative in the $x1$ direction. Uses same types of arguments as the
$interpolate_value_2d$ function.

`interpolate_x2_derivative_2d()` returns the value of the first
derivative in the $x2$ direction. Uses same types of arguments as the
$interpolate_value_2d$ function.

`get_x1_min()` returns the minimum value of $x1$ in the domain in which
the spline object is defined. Arguments:

spline_object:

:   a pointer to a previously initialized object of type
    `sll_cubic_spline_2D`

`get_x1_max()` returns the maximum value of $x1$ in the domain in which
the spline object is defined. Arguments:

spline_object:

:   a pointer to a previously initialized object of type
    `sll_cubic_spline_2D`

`get_x2_min()` returns the minimum value of $x2$ in the domain in which
the spline object is defined. Arguments:

spline_object:

:   a pointer to a previously initialized object of type
    `sll_cubic_spline_2D`

`get_x2_max()` returns the maximum value of $x2$ in the domain in which
the spline object is defined. Arguments:

spline_object:

:   a pointer to a previously initialized object of type
    `sll_cubic_spline_2D`

`get_x1_delta()` returns the value of the spacing between the points
used to compute the spline values in the $x1$ direction. This is
determined from the extent of the domain and the number of cells used.
Arguments:

spline_object:

:   a pointer to a previously initialized object of type
    `sll_cubic_spline_2D`

`get_x2_delta()` returns the value of the spacing between the points
used to compute the spline values in the $x2$ direction. This is
determined from the extent of the domain and the number of cells used.
Arguments:

spline_object:

:   a pointer to a previously initialized object of type
    `sll_cubic_spline_2D`

`sll_delete()` subroutine to free the resources of a given spline
object. Arguments:

spline_object:

:   a pointer to a previously initialized object of type
    `sll_cubic_spline_2D`
:::

### Usage

To use the module in a stand-alone way, include the line: 

         use sll_splines

The following example is an extract from the module's unit test.

Here we do not go in detail over every line but only highlight those
lines in which we interact with the splines module

Line 5:

:   Imports the spline module. The intent is to eventually not require
    this but to import a single module, say 'selalib' which will include
    all modules itself. For now, this is the way to include these
    individual capabilities.

Lines 13 - 14: 

:   Declaration of the spline pointers.

Lines 29, 34:

:   Allocation and partial initialization of the splines. Note the `=>`
    pointer assignment syntax.

Lines 33, 39:

:   Calculation of the spline coefficients. After this call the spline
    object becomes fully usable. Note that in the example we used both
    existing interfaces to call the initialization subroutine.

Lines 45, 52:

:   Value interpolation using the existing splines.

Lines 57, 67:

:   Destruction of spline objects.

### Status

Unit-tested.

## Gauss-Legendre Integrator

### Description

This is a low-level mathematical utility that applies the Gauss-Legendre
method to compute numeric integrals. This module aims at providing a
single interface to the process of integrating a function on a given
interval.

### Exposed Interface

To integrate the function `f(x)` (real-valued and of a single,
real-valued argument `x`) over the interval $[a, b]$, the simplest way
is through a function call such as:

          gauss_legendre_integrate_1D(f, a, b, n)

In the function above, `n` represents the desired number of *Gauss*
points used in the calculation:
$$\int_{-1}^1f(x) \mathrm{d} x \approx \sum_{k=1}^n w_kf(x_k)$$

Presently, the implementation accepts values of `degree` between $2$ and
$10$ inclusively. The function `gauss_legendre_integrate_1D` internally
does the proper scaling of the points to adjust the integral over the
desired interval.

The function `gauss_legendre_integrate_1D` is a generic interface and as
such, it hides some alternative integrators, which are selected
depending on the type of the passed arguments. For instance, we have
available the function

         gauss_legendre_integrate_interpolated_1D(f, spline, a, b, n)

which integrates a function represented by a spline object. The function
`f` in this case is the spline interpolation function. It looks like
this interface could be simplified and we could eliminate the first
parameter and pass only the spline object. The only reason to leave the
interpolation function as an argument is if we find some compelling
reason to parametrize the interpolation function as well.

It will be necessary to implement other integrators for functions with a
different signature, such as two- or three-parameter functions. It might
also be necessary to distinguish between one-dimensional and two- or
3-dimensional integration. While this is not yet implemented, here we
lay out some suggestions on how to proceed in such cases.

For the class of integrals that are done in one-dimension, such as the
above, the cleanest but somewhat more laborious approach appears to be
to write a different integrator for every function signature that is
needed. For instance, one may need to write specialized integrators for
$f(x_1, x_2)$, $f(x_1, x_2, x_3)$ and so on, with the convention that
the integral is carried out over, say, the first of the variables. The
variables which are not integrated can be used as parameters. The
alternative to this approach could be to write a single integrator that
is able to receive multiple parameters, through the use of arrays or
derived types, but the ugliness of this approach, and the need to
basically write glue-code (pack/unpack the arrays or derived types with
variables and parameters) every time one wants to integrate something
are reasons to reject this approach.

### Usage

As mentioned above, the name of the generic function that hides the
specialized functions is `gauss_legendre_integrate_1D`. The specialized
functions can be individually called to avoid the overhead of the
generic function call if desired. A one-dimensional function (user or
Fortran) can be integrated by a call like:

         gauss_legendre_integral_1D( test_function, &
                                     0.0_f64,       &
                                     sll_pi/2.0,    &
                                     4 )

A function that is represented by an underlying spline object can be
called like:

         gauss_legendre_integral_interpolated_1D( interpolate_value,&
                                                  sp1,              &
                                                  0.0_f64,          &
                                                  sll_pi,           &
                                                  4)

where sp1 is a spline object. It should be decided if this last case is
indeed that interface that is wished, or if something more simplified
should be implemented intstead.

### Status

Unit-tested.

## FFT

### Description

The FFT module intends to provide an unified interface to native or
external library FFT functions. This module plays a role analogous to
the *Collective* module, explained below, but in this case applied to
FFT capabilities instead of a parallelization library like MPI. In this
sense, the `sll_fft` module provides a set of types, functions and
subroutines that can in principle be implemented with a choice of
external libraries.

### Exposed Interface

The interface to the FFT functions is inspired by the FFTW interface in
which the FFT operation is carried out by the creation of a plan
followed by the execution of this plan. The \"plan\" stores all relevant
information (twiddle factor arrays, auxiliary arrays, etc.) that may be
needed by a given type of FFT operation. The application of the plan
executes the operation itself on a given data. Like all other native
types in Selalib, the FFT plan is declared through a pointer, which must
be allocated and initialized. To declare, call:

         type(sll_fft_plan), pointer :: fft_plan

Followed by an initialization step:

         fft_plan => sll_new_fft( sample_number,  
                                  data_type, 
                                  fft_flags )

Where the `data_type` parameter can take either of the values:
`FFT_REAL` or `FFT_COMPLEX`. Presently we only provide double precision
transforms (i.e.: 64-bit floating precision numbers). The `fft_flags`
can take the values: `FFT_NORMALIZE_FORWARD` and/or
`FFT_NORMALIZE_INVERSE`. If no flags are present, an unnormalized FFT
will be executed. The user may request that the FFT be normalized in
both directions by combining the flags with a ` ` + sign.

To execute the FFT plan:

sample_number:

:   

spline:

:   A pointer to the spline object to be filled or updated.

### Usage

### Status

## Collective Communications

### Description

Selalib applies the principle of modularization throughout all levels of
abstraction of the library and aims at keeping third-party library
modules as what they are: separate library modules. Therefore, in its
current design, even a library like MPI has a single point of entry to
Selalib. The collective communications module is such point of entry. We
focus thus on the functionality offered by MPI, assign wrappers to its
most desirable functionalities and write wrappers around them. These are
the functions that are actually used throughout the program. This allows
to adjust the exposed interfaces, do additional error-checking and would
even permit to completely change the means to parallelize a code, by
being able to replace MPI in a single file if this were ever needed.

### Exposed Interface

Fundamental type:

         sll_collective_t

Constructors, destructors and access functions:

         sll_new_collective( parent_col )
         sll_delete_collective( col )

When the Selalib environment is activated, there exists, in exact
analogy with `MPI_COMM_WORLD`, a global named `sll_world_collective`. At
the beginning of a program execution, this is the only collective in
existence. Further collectives can be created down the road. The above
functions are responsible for the creation and destruction of such
collectives. The following functions are used to access the values that
a particular collective knows about.

         sll_get_collective_rank( col )
         sll_get_collective_size( col )
         sll_get_collective_color( col )
         sll_get_collective_comm( col )

Since the wrapped library requires initialization, so does
`sll_collective`. To start and end the parallel environment, the user
needs to call the functions:

         sll_boot_collective( )
         sll_halt_collective( )

These functions would not be exposed at the top level, and would be
hidden by a further call to something akin to `boot_selalib` and
`halt_selalib`. Finally, the wrappers around the standard `MPI`
capabilities are presently exposed through the following generic
functions:

         sll_collective_bcast( col, buffer, size, root )
         sll_collective_gather( col, send_buf, send_sz, root, 
                                rec_buf )
         sll_collective_gatherv( col, send_buf, send_sz, recvcnts, 
                                 displs, root, recv_buf )
         sll_collective_allgatherv( col, send_buf, send_sz, sizes, 
                                    displs, rec_buf )
         sll_collective_scatter( col, send_buf, size, root, 
                                 rec_buf )
         sll_collective_scatterv( col, send_buf, sizes, displs, 
                                  rec_szs, root, rec_buf )
         sll_collective_all_reduce( col, send_buf, count, op, 
                                    rec_buf )

which presently stand for specialized versions that operate on specific
types. For instance:

         sll_collective_all_to_allv_real( send_buf, 
                                          send_cnts, 
                                          send_displs, 
                                          recv_buf, 
                                          recv_cnts, 
                                          recv_displs, 
                                          col )

### Usage

To use the module as stand-alone, include the line:

         use sll_collective

Any use of the module's functionalities must be preceeded by calling

         call sll_boot_collective()

and to \"turn off\" the parallel capabilities, one should finish by a
call to:

         call sll_halt_collective()

This *booting* of the parallel environment needs to be done only once in
a program.

Some more specific examples are needed here\...

### Status

Several core functionalities tested, but no comprehensive unit test done
yet

## Remapper

### Description

Written on top of `sll_collective`, the remapper is a powerful module
capable of rearranging non-overlapping data in a parallel machine. This
module also implements the concept of a *layout*, which is a data
structure that stores the index ranges of a given array to be found in
each processor. In other words, the layout contains a description of the
distribution of the data. Layouts are needed to specify remap
operations.

The use of the remapper module can be broken down in several stages:

1.  Declare the layouts and the remap plan object, as well as the arrays
    to be used.

2.  Initialize the layouts,

3.  use the layouts and the arrays to create the remapper plan and,

4.  apply the remapper plan using the input and output arrays as
    arguments.

5.  As a final step, the resources in the layouts and remap plan should
    be deallocated wtih `sll_delete()`

The remap operation is an out-of-place operation. Remap is designed to
consider only non-overlapping data configurations, that is, an
*all_gather*-type operation should not be attempted with a remap.

### Exposed Interface

Since the layouts are a pre-requisite for dealing with the remapper, we
start with these. Presently, Selalib offers the following layout types:

         layout_2D
         layout_3D
         layout_4D
         layout_5D
         layout_6D

These types are each accompanied by their own constructors, destructors
and accessors. A comprehensive list of the procedures that operate with
the layouts is:

         new_layout_2D( collective )
         new_layout_3D( collective )
         new_layout_4D( collective )
         new_layout_5D( collective )
         new_layout_6D( collective )
         
         get_layout_collective( layout )
         get_layout_num_nodes( layout )
         
         get_layout_global_size_i( layout )
         get_layout_global_size_j( layout )
         get_layout_global_size_k( layout )
         get_layout_global_size_l( layout )
         get_layout_global_size_m( layout )
         get_layout_global_size_n( layout )
         
         get_layout_i_min( layout, rank )
         get_layout_i_max( layout, rank )
         get_layout_j_min( layout, rank )
         get_layout_j_max( layout, rank )
         get_layout_k_min( layout, rank )
         get_layout_k_max( layout, rank )
         get_layout_l_min( layout, rank )
         get_layout_l_max( layout, rank )
         get_layout_m_min( layout, rank )
         get_layout_m_max( layout, rank )
         get_layout_n_min( layout, rank )
         get_layout_n_max( layout, rank )
         
         set_layout_i_min( layout, rank, val )
         set_layout_i_max( layout, rank, val )
         set_layout_j_min( layout, rank, val )
         set_layout_j_max( layout, rank, val )
         set_layout_k_min( layout, rank, val )
         set_layout_k_max( layout, rank, val )
         set_layout_l_min( layout, rank, val )
         set_layout_l_max( layout, rank, val )
         set_layout_m_min( layout, rank, val )
         set_layout_m_max( layout, rank, val )
         set_layout_n_min( layout, rank, val )
         set_layout_n_max( layout, rank, val )

         initialize_layout_with_distributed_2d_array( 
              global_npx1, 
              global_npx2, 
              num_proc_x1, 
              num_proc_x2, 
              layout)
              
         initialize_layout_with_distributed_3d_array( 
              global_npx1, 
              global_npx2, 
              global_npx3, 
              num_proc_x1, 
              num_proc_x2, 
              num_proc_x3, 
              layout)
              
         initialize_layout_with_distributed_4d_array( 
              global_npx1, 
              global_npx2, 
              global_npx3, 
              global_npx4, 
              num_proc_x1, 
              num_proc_x2, 
              num_proc_x3, 
              num_proc_x4, 
              layout)
              
         initialize_layout_with_distributed_5d_array( 
              global_npx1, 
              global_npx2, 
              global_npx3, 
              global_npx4, 
              global_npx5, 
              num_proc_x1, 
              num_proc_x2, 
              num_proc_x3, 
              num_proc_x4, 
              num_proc_x5, 
              layout)
              
         initialize_layout_with_distributed_6d_array( 
              global_npx1, 
              global_npx2, 
              global_npx3, 
              global_npx4, 
              global_npx5, 
              global_npx6, 
              num_proc_x1, 
              num_proc_x2, 
              num_proc_x3, 
              num_proc_x4, 
              num_proc_x5, 
              num_proc_x6, 
              layout)

         compute_local_sizes( 
              layout_Nd, 
              loc_sz_1, 
              loc_sz_2, 
              ..., 
              loc_sz_N )

         sll_delete( layout )

In brief:

::: 
`new_layout_ND()` function, is responsible for allocating all the
necessary storage for the layout. Returns a pointer to a layout_ND
object, ready for initialization. Arguments:

collective:

:   \[pointer\] sll_collective type to be associated with the layout.

`get_layout_collective` returns a pointer to the sll_collective object
associated with a given layout. Arguments:

layout:

:   \[pointer\] layout whose collective is requested.

`get_layout_num_nodes` returns a 32-bit integer with the number of
processors in the collective associated with the given layout.
Arguments:

layout:

:   \[pointer\] layout whose size (in number of processes) is being
    requested.

`get_layout_global_size_X` returns a 32-bit integer with the size of the
global array (in an abstract sense, i.e. the array that is distributed)
in the direction X. Arguments:

layout:

:   \[pointer\] layout whose size (in number of processes) is being
    requested.

`get_layout_X_min` returns a 32-bit integer with the minimum index in
direction X contained in the given process rank. Arguments:

layout:

:   \[pointer\] layout whose size (in number of processes) is being
    requested. \[in\] 32-bit integer, rank of the process under inquiry.

`get_layout_X_max` returns a 32-bit integer with the maximum index in
direction X contained in the given process rank. Arguments:

layout:

:   \[pointer\] layout whose size (in number of processes) is being
    requested. \[in\] 32-bit integer, rank of the process under inquiry.
:::

Specifically, the constructors are:

Note that each layout descriptor needs to be allocated by providing an
instance of `sll_collective`. This can be understood by reasoning about
the data layout as a group of processors (the collective) and a
specification of the data boxes contained in each one. More concretely,
a layout for an N-dimensional array associated with a collective of size
$N_p$ can be thought of as a collection of $N_p$ N-dimensional boxes,
each representing the intervals of the array indices contained in a
given processor (rank: 0 \... $N_p-1$). After calling any of the
`new_layout` functions, the returned instance becomes associated to the
given collective and enough memory is allocated (size of the collective)
to hold the boxes specification. While the memory has been allocated,
the layout itself still needs to be initialized.

There are at least two ways to accomplish the initialization. The manual
way is through the individual access functions defined for the layouts.
The second method involves helper routines.

The following is a list of the access functions defined for the layouts:

In the above list, all the routines are generic, thus they will accept
layouts of any dimension as long as the field to be accessed is defined
in the layout. The array dimensions are labeled `i, j, k, l, m, n`
(although in the arguments we use the numbers 1, 2, \... , 6). Thus, for
example, a `layout_2D` contains information on the indices `i`, `j`
only. Accessing the `k` index would yield a compilation error. The
`get_` routines are functions. Except for `get_layout_collective( )`
which returns a pointer to the collective associated with a given
layout, all the other `get_`-functions return the value of a 32-bit
integer value, representing the minimum or maximum value of an index
contained in a given rank, or in the case of the
`get_layout_global_size_X` functions, the global size of the (abstract)
array that is split among the different processors in the way described
by the layout.

The `set_` procedures are subroutines. These can be used to set a given
minimum/maximum index value in a given layout to a desired value. These
are the accessors that may be used when initializing a layout manually.
However this is not encouraged. In general, it is recommended to
initialize layout with the provided helper functions, as shown next.

The above procedures are subroutines. Except for the layout to be
initialized, all the arguments are 32-bit integers. The `global_npxX`
arguments denote the global size of the array (its total size in the
$X$-th direction) that is to be divided amongst multiple processors in
accordance with the partition determined by the `num_proc_xX` arguments.
To understand the role of the `num_proc_xX` parameters it is necessary
to imagine the domain represented by the N-dimensional array as being
divided in a number of blocks in each dimension, just as if the
processors themselves were arranged in a *processor mesh* of dimensions
`num_proc_x1` \* `num_proc_x2` \* \... \* `num_proc_xN` = number of
processes in the collective. Thus, if distributing a 2D array of sizes
`global_npx1` $= 1024$, `global_npx2` $= 512$, in a process mesh of
dimensions `num_proc_x1` $= 2$ and `num_proc_x2` $= 1$, this would yield
a partition of the original array in one block, of dimensions
$512 * 512$ in each of the two processors. Note again that the number of
processors corresponds to the product of the `nproc_xX` arguments.

The information in the initialized layouts can be used to inquire about
the size of the data distributed as per a given layout. The procedures
for this are hidden behind the generic interface:

Once a layout is declared and initialized it can be used as an aid to
allocate the memory of the array(s) whose distribution it represents.
For instance, suppose that you start with a multidimensional array that
needs to be distributed among $N_p$ processors. The layout of the data
(that is, the description of what ranges of the data are contained in
each processor) is specified by an instance of the type `layout_XD`,
(where `X` is the dimension of the data). The layout contains a notion
of an $N_p$-sized collection of boxes, each box representing a
contiguous chunk of the multidimensional array stored in each processor.
If in the course of a computation, you wish to reconfigure the layout of
the data (for example, if you wished to re-arrange data in a way that
would permit launching serial algorithms locally in each processor),
then you would create and initialize a new layout descriptor with the
target configuration (i.e.: you to define the box to be stored in each
node). Armed with the layout descriptions for the initial and target
configurations, one can execute a remap operation to reconfigure the
data. This is an out-of-place operation.

         NEW_REMAPPER_PLAN_3D( initial_layout, 
                               target_layout, 
                               data_size_in_integer_sizes )
         NEW_REMAPPER_PLAN_4D( initial_layout, 
                               target_layout, 
                               data_size_in_integer_sizes ) 
         NEW_REMAPPER_PLAN_5D( initial_layout, 
                                 target_layout, 
                               data_size_in_integer_sizes )

will yield an instance of the type `remap_plan_3D_t`, or
`remap_plan_4D_t` or `remap_plan_5D` , respectively, that will contain
all the information necessary to actually carry out the data
re-distribution. Finally, a call to

         apply_remap_3D( plan, data_in, data_out )
         apply_remap_4D( plan, data_in, data_out )
         apply_remap_5D( plan, data_in, data_out )

will actually redistribute `data` (as an out-of-place operation)
according to `plan` in an optimized way[^3].

To appreciate the power of such facility, note that in principle, the
construction of a (communications latency-limited) parallel
quasi-neutral solver can be based exclusively on remapping operations.
This is an important tool in any problem that would require global
rearrangements of data. The remapper thus is able to present a single
powerful abstraction that is general, reusable and completely hides most
of the complications introduced by the data distribution.

The access functions for the `layout` types are are always prefixed with
the corresponding `get_layout_XD`/`set_layout_XD` (where the '`X`'
denotes the dimensionality of the data), and they presuppose knowledge
of the convention for ordering the indices as in `i, j, k, l, m`, for
the dimensions. Specifically, to get/set values inside the `layout`
types we have available for 3D layouts:

As a very inelegant convenience, the layout type allows direct access to
its collective reference.

The above functions define the interface that will allow you to declare
and initialize the `layout` types as desired. This is where the work
lies when using this module. Note that all the above functions could be
coalesced into a set of functions of the type
`set_layout_X_XXX(layout, rank, val)` if we choose to hide all the above
functions behind a generic interface. The selection would be done
automatically depending on the type of layout passed as an argument.

The type `remap_plan` exists also in multiple flavors, depending on the
dimensionality of the data to be remapped:

         remap_plan_3D_t
         remap_plan_4D_t
         remap_plan_5D_t

The `remap_plan_t` type stores the locations of the memory buffers that
will be involved in the communications, the specification of the data
that will be sent and received, as well as the collective within which
the communications will take place. There are, however, declaration
functions available. The choice depends on the dimensionality of the
data:

         NEW_REMAPPER_PLAN_3D( initial_layout, 
                               final_layout, 
                               array_name )
         NEW_REMAPPER_PLAN_4D( initial_layout, 
                               final_layout, 
                               array_name )
         NEW_REMAPPER_PLAN_5D( initial_layout, 
                               final_layout, 
                               array_name )

Finally, the way to execute the plan on a particular data set is through
a call of the appropriate subroutine (here presented as generic
interfaces)

         apply_remap_3D( plan, data_in, data_out )
         apply_remap_4D( plan, data_in, data_out )
         apply_remap_5D( plan, data_in, data_out )

### Usage

For use in stand-alone way, use the line:

    #include "sll_remap.h"

While verbose, the best way to demonstrate the usage of the remapper is
with a complete program. Below it, we examine the different statements.

Lines 1 - 5:

:   Required preamble at the time of this writing. Eventually this will
    be replaced by a single statement to include the whole library.
    Presently, we include various headers individually, so bear in mind
    that this is not the way this will end up being. Line 3 specifically
    loads the remapper facility. Here it is brought as a header file as
    the NEW_REMAPPER_PLAN_XD() is implemented as a macro.

Lines 9 - 12: 

:   For this example we allocate two 3D arrays for the input and output
    of the remap operation.

Lines 14 - 22: 

:   Definition of the array size from a global perspective. In other
    words, the array to be remapped is a $8*8*1$ array, to be
    distributed on a processor mesh of dimensions $4*4*1$.

Lines 24 - 36

:   Miscellaneous integer variables that we will use.

Lines 38 - 41

:   Pointers to the initial and final layouts and the remap plan.

Line 44

:   Presently we boot from collective. Eventually this will be replaced
    by a call to something like `boot_selalib()` or something similar,
    where we declare and initialize anything we need in a single call.

Lines 46 - 57

:   Initialization of the variables.

Lines 59 - 78

:   This is where the actual work is when using the remapper. We need to
    initialize a layout, in this case the initial configuration. We use
    the access functions `set_layout_x_xxx( )` to populate the fields.
    Here we obviously take into account the geometry of our 'process
    mesh' to find out the rank of the process that we are initializing.

Lines 80 - 90

:   We need to initialize the data, here we choose simply to assign the
    index of the array, considered as a 1D array. Note the use of the
    helper function `local_to_global_3D( layout, triplet )`. We exploit
    the knowledge of the global layout of the data to find out the
    global indices of a local 3-tuple.

Lines 92 - 112

:   The other main part of the work, the initialization of the target
    layout. In this case, we chose a simple transposition, which is
    achieved by switching `i` and `j`.

Lines 114 - 115

:   Here we allocate and initialize the remap plan, using the initial
    and final configurations as input. The third argument is passed to
    inform the remapper of the type of data to be passed. The call to
    `apply_remap_3D()` is a call to a generic function, hence, a
    type-dependent sub-function must have been defined to be able to
    successfully make this call. At the time of this writing, only
    single precision integers and double precision floats have been
    implemented.

Line 116

:   Here we apply the plan. This function is type-dependent due to the
    input/output arrays. Please refer to the implementation notes for
    some commentary on our options with this interface.

Lines 122 - 125

:   Cleanup. The layouts need to be deleted to prevent memory leaks.

### Implementation Notes

The biggest challenge with the remapper is to attain a desired level of
genericity and to preserve the modularity of the library. These two
problems are intimately related. Ideally, we should be able to apply a
remap operation on data of any type, including user-derived types.
Another requirement has been to confine a library like MPI to a single
entry point into Selalib. This means that we do not want the MPI derived
types to pollute the higher abstraction levels of the library:
especially at the top level, we want to express our programs with the
capabilities of the FORTRAN language alone.

These requirements were solved in the prototype version of the remapper
through the use of a single datatype to represent all other types of
data at the moment of assembling the exchange buffers and launching the
MPI calls. In our case, we have chosen to represent all data as
'integers'. This means that the exchange buffers that are stored in the
remap plans are integer arrays. Thus, the design decision in the
prototype has been to choose flexibility and ease of change over
execution speed. In contrast with the C language, the constant call to
the `transfer()` function to store and retrieve data from the exchange
buffers carries with it a possibly significant execution time penalty.

The function `NEW_REMAPPER_PLAN_XD()` is by nature type-independent, as
the design of the plan only depends on the layouts. However, it is also
convenient to store the send/receive buffers in the plan, and the
allocation of these buffers requires knowledge of the amount of memory
required. This information is passed in the third argument. The macro
will internally select an element of this array and determine its size
in terms of the fundamental datatype being exchanged (i.e.: `integer`).
This way we now how much memory to allocate in the buffers.

Another means to achieve the illusion of genericity are Fortran's
built-in features in this regard. For example, we can have specialized
`apply_remap_3D()` functions for the most commonly used datatypes, all
hidden behind the same generic name. These specialized functions would
not depend on the current choice of using a single type for the exchange
buffers, eliminating any penalty that we are definitively paying at
present, with the calls to the `transfer()` intrinsic function. This
solution would mean writing redundant code, something that could be
addressed with preprocessor macros, but this would not be a solution for
eliminating the penalizations of the `transfer()` intrinsic when we are
exchanging derived types. A solution that can exchange these arbitrary
data while not requiring the use of the MPI derived types at the higher
levels is yet to be found. It could be that the Fortran way to solve
this problem would be to accept the invasion of MPI at the higher
levels\...

### Status

In testing.

