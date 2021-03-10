# Top-Level Layer: Semi-Lagrangian Toolbox

## Meshes

### Description

The mesh types aim at storing array data, but including additional
information needed to put this data in context. There are several types
of meshes:

cartesian mesh

:   : an N-D fully regular mesh defined by an N-D data array and
    additional parameters that define the domain (i.e.: xmin, xmax) and
    other parameters like the number of cells in each dimension and the
    spacing of the data (i.e.: delta). The services provided should be
    like (with corresponding name changes, indicating the dimensionality
    of the data):

    -   allocation(new) and initialization functions,

    -   deletion,

    -   a copy constructor,

    -   computation of data interpolants (cubic splines for example),

    -   `get_value(mesh,x1,x2,...)`: where `x1`, `x2`,\... belong to the
        corresponding interval `(x1_min, x1_max)`, `(x2_min, x2_max)`,
        etc., and which returns the interpolated value at the desired
        point. This operation launches spline interpolations under the
        hood.

    -   `get_node_value(mesh,i,j,...)`: analogous to `get_value()` but
        does not need to launch any interpolation as the indices are
        integers and the requested value falls on a mesh node. This is
        implemented by a macro.

    -   `set_node_value(mesh,i,j,...,val)`: sets the node datum at
        i,j,\... with value. Implemented with a macro.

There are some pitfalls with the suggested interface:

-   the naming convention could get a little complicated depending on
    the type of data stored in the array (scalar, multiple-valued,
    etc.), as we had originally intended with the field/vec naming
    convention.

-   The interface may need to directly expose its underlying data, or
    pointers to sections of it for certain operations, like FFT. The
    whole data field of a mesh could need to be set to a whole data
    array in one step, during the initialization, such as after a remap
    operation. At least in these cases, the access to the data might be
    safer given the nature of the operations.

structured mesh

:   : a structured mesh is a mapped cartesian mesh in which the
    coordinates are decoupled. An ND mesh data is stored as an ND array.
    In addition, however, we need N 1D arrays to store the actual
    coordinates of the node locations in each dimension. As an
    alternative, we could use N functions $x_1=f(\eta_1)$,
    $x_2=f(\eta_2)$, \... , $x_n=f(\eta_n)$, to represent each
    transformation. We follow the convention that $\eta_1$, $\eta_2$,
    etc. are values in the $[0,1]$. The offered services ought to be:

    -   allocation, initialization, deletion and copy.

    -   `get_node_value(mesh, i, j, ... )`: which reads directly from
        the data array.

    -   `set_node_value(mesh, i, j, ... )`: which writes directly to the
        data array.

    -   In an analogous way to the `get_value()` functions described
        above, we could provide something like `get_value(mesh,`
        $\eta_1$`, `$\eta_2$`)`. This would also trigger an
        interpolation step using uniform splines generated with the ND
        data. This would require the user to be *thinking* in terms of
        the logical variables $\eta_i$.

    -   Similar functions could be provided such that the user could
        also request values at points $x_i$. For this, we could launch
        non-uniform splines, with the spacing determined by the $x_i$
        arrays and the ND data.

    -   This type may also need to grant direct access to the data array
        for use in operations like FFT and similar.

tensor product mesh

:   : This is a mapped cartesian mesh in which the coordinates are
    coupled. That is, the mappings have the form:
    $x_1=f(\eta_1, \eta_2, ..., \eta_n)$,
    $x_2=f(\eta_1,\eta_2, ..., \eta_n)$, \... ,
    $x_n=f(\eta_1,\eta_2, ..., \eta_n)$. The specification of this type
    of mesh requires an ND data array, N ND arrays that specify the
    transformations numerically, or N functions of arity N to specify
    the transformations. The services provided should be:

    -   allocation, initialization, deletion and copy.

    -   `get_node_value(mesh, i, j, ... )`: which reads directly from
        the data array.

    -   `set_node_value(mesh, i, j, ... )`: which writes directly to the
        data array.

    -   `get_value(mesh,` $\eta_1$`, `$\eta_2$`, ... )` which would also
        trigger uniform spline interpolations, and

    -   `get_value(mesh, x1, x2, ... )`, which would trigger nonuniform
        spline interpolations.

    We may need to include other operations here which would entail
    inverse mappings (maybe also with splines), or NURBS or something
    else. Need to fill in the details more here.

### Exposed Interface

### Usage

### Status

## Scalar Fields

### Description

The physical quantities of interest are normally defined in a physical
space. For instance, the electric potential $\phi(\vec{x})$ is specified
by a function on the $\vec{x}$ variables ($x,y,z$). Even in simple
problems the borders of the physical domain may have relatively
complicated shapes which make the solution of differential equations
more difficult. Great flexibility (possibly at the expense of
efficiency) can be gained by transferring the data from the physical
space to a representation in a logical space in which the underlying
grid that we use for the numerical solution is always uniform. In such
uniform grid, the borders have a much simpler (straight line)
representation that may aid the solution of the equations. This means
that it is convenient to permit the alternative representation of the
data on variables on a physical space ($\vec{x}$) or on a logical space
($\vec{\eta}$). For these reasons, the scalar field should be thought of
data plus a coordinate transformation.

(place picture of an example coordinate transformation here.)

Consider a scalar field and the services that it should offer to permit
us to use it in our logical grid:

1.  The scalar field is a derived type.

2.  The type contains a simple array to store the data, i.e.: the values
    of the fields on a collection of points. Both, logical mesh values
    or physical mesh values can be stored in the same array.

3.  The type contains an object that fully specifies the associated
    coordinate transformation (a mapped mesh). The transformation should
    provide all the needed services to permit moving the representation
    of the data from the physical space to the logical space and
    vice-versa:

    1.  something

    2.  something else

## Quasi-Neutral Equation Solver

### Description

Here we present a simplified but hopefully reasonably clear step-by-step
derivation of the quasi-neutral equation (the gyrokinetic Poisson
equation) which this module solves. The intent is to make clear some of
the assumptions that are built into this model. We follow the general
argument given by Krommes (cite reference here for \"Nonlinear
gyrokinetics: a powerful tool for the description of microturbulence in
magnetized plasma\").

The model is based on the idea that the fast gyrations of particles
around the magnetic field lines can be averaged away while still
preserving the most important long-term physics (hence the name of this
type of approach: gyrokinetics). The simplest model that we will look
into first deals directly in the distribution function of the
gyrocenters, instead of the particles'.

We start with the Poisson equation:

$$-\nabla ^2 \phi = \frac{1}{\epsilon_0}\rho,$$ where $\phi$ is the
electric potential, $\rho$ is the volumetric charge density and
$\epsilon_0$ is the permittivity of free space. The main assumptions
built into the model are introduced through the treatment of $\rho$. We
decompose the charge density in its two main constituents: the
contribution by the ions ($\rho_i$) and the electrons' ($\rho_e$). As
mentioned earlier, the main idea behind gyrokinetics is the averaging of
the fast particle motions around the field lines, hence some of the
effects that occur on longer time-scales are explicitly introduced into
the model, for example by particle drifts (e.g.: $\vec{E}\times\vec{B}$,
polarization drift, etc.). In doing so, we allow ourselves to further
separate the charge density into a polarization charge and a charge at
the particles' gyrocenters (gyrocenters do not polarize). Thus,
Poisson's equation becomes

$$\label{eq:poisson_2}
-\nabla ^2 \phi = \frac{1}{\epsilon_0}(\rho^G_i + \rho^{pol}_i - \rho^G_e - \rho^{pol}_e),$$
where the indices $G$, $pol$, $i$ and $e$ indicate *gyrocenters*,
*polarization*, *ions* and *electrons* respectively. The polarization
drift velocity, whose derivation can be found in introductory plasma
physics texts is given by

$$\vec{v}^{pol} = \pm \frac{1}{\omega_{ci}B}\frac{\partial \vec{E}_{\perp}}{\partial t}.$$
Here $\omega_{ci}$ is the ion cyclotron frequency, $B$ is the magnitude
of the local magnetic field and $\vec{E}_{\perp}$ is the electric field
perpendicular to the magnetic field. The plus/minus sign in the equation
applies to positive and negative particles respectively. Heuristically,
the polarization charge obeys a continuity equation:

$$\label{eq:rho_pol_continuity}
\frac{\partial \rho^{pol} }{\partial t} = - \nabla \cdot \vec{j}^{pol} = - \nabla \cdot (nZe \vec{v}^{pol})=- \nabla \cdot \bigg( n Ze \frac{1}{\omega_{ci}B}\frac{\partial \vec{E}_{\perp}}{\partial t}\bigg) .$$

Here, $Z$ is the charge state of the ions under consideration (obviously
1 for a hydrogen-burning fusion plasma), $e$ is the electron charge and
$n$ is the particle density per unit volume. As long as none of the
factors of the time derivative depends itself on time, we can integrate
immediately and arrive at an expression for $\rho^{pol}$:

$$\label{eq:rho_pol}
\rho^{pol}(\vec{x}) = \nabla \cdot \bigg( \frac{n_i(\vec{x})Ze}{\omega_{ci}(\vec{x}) B(\vec{x}) }\nabla_{\perp}\phi(\vec{x},t) \bigg).$$

Here we have introduced the assumption of $n_i$, the ion particle
density, being independent of time. In equation
([\[eq:rho_pol\]](#eq:rho_pol){reference-type="ref"
reference="eq:rho_pol"}), we can consider that the differential
operators act only in the directions perpendicular to the magnetic
field, as these are the only terms that will survive the taking of the
divergence. Finally, by multiplying by the proper unit factors and using
the relation

$$\omega_p = \bigg( \frac{ne^2} {\epsilon_0 m} \bigg)^{\frac{1}{2}}$$ we
can recast equation ([\[eq:rho_pol\]](#eq:rho_pol){reference-type="ref"
reference="eq:rho_pol"}) into

$$\label{eq:rho_pol_epsilon}
\rho^{pol}(\vec{x}) = \epsilon_0 \nabla_{\perp} \cdot \big( \varepsilon^G(\vec{x})\nabla_{\perp}\phi(\vec{x},t) \big).$$
where

$$\varepsilon^G(\vec{x}) \equiv \frac{\omega^2_{pi}(\vec{x})}{\omega^2_{ci}(\vec{x})}$$
is called the *dielectric constant of the gyrokinetic vacuum*. With
equation
([\[eq:rho_pol_epsilon\]](#eq:rho_pol_epsilon){reference-type="ref"
reference="eq:rho_pol_epsilon"}), and neglecting the polarization of the
electrons, the modified Poisson equation
([\[eq:poisson_2\]](#eq:poisson_2){reference-type="ref"
reference="eq:poisson_2"}) can be written as:

$$-\nabla ^2 \phi(\vec{x},t) - \nabla_{\perp} \cdot \big(\varepsilon ^G(\vec{x},t)\nabla_{\perp} \phi(\vec{x},t) \big) = \frac{1}{\epsilon_0}(\rho^G_i - \rho^G_e).$$
In fusion applications, $\varepsilon^G >> 1$ thus we neglect the
ordinary laplacian term on the left-hand side.

The charge density of the electrons also receives a special treatment.
The basic assumption here is that the electrons can move very quickly
along a magnetic field line and thus are able to rapidly react to
electric potential variations through changes in the electron particle
density. Thus, it is assumed that along a field line, the electrons obey
the Boltzmann relation:

$$\label{eq:boltzmann}
n_e(\vec{x},t) = \bar{n}_e(\vec{x},0) \exp \bigg(\frac{e}{k_BT_e}(\phi(\vec{x},t) - <\phi(\vec{x},t)>_{\ell})\bigg).$$
In equation ([\[eq:boltzmann\]](#eq:boltzmann){reference-type="ref"
reference="eq:boltzmann"}), $n_e(\vec{x},t)$ is the instantaneous
electron particle density, $\bar{n}_e(\vec{x})$ is the average electron
density along the magnetic field line, $k_B$ is Boltzmann's constant,
$T_e$ is the electron temperature and the $< \cdot >_{\ell}$ average is
taken along the magnetic field line. By making yet another assumption of
very small deviations from the average electric potential, the previous
equation can be linearized:

$$\label{eq:boltzmann_linear}
n_e(\vec{x},t) = \bar{n}_e(\vec{x},0) \bigg(1+\frac{e}{k_BT_e}(\phi(\vec{x},t) - <\phi(\vec{x},t)>_{\ell})\bigg).$$
At the risk of being too sloppy, we will equate the electron particle
density with the electron gyrocenter density. A more careful step would
involve gyroaveraging directly the original electron distribution
function. With this assumption, our modified poisson becomes:

$$- \nabla_{\perp} \cdot \big(\varepsilon ^G(\vec{x},t)\nabla_{\perp} \phi(\vec{x},t) \big) = \frac{1}{\epsilon_0}\bigg(\rho^G_i - \bar{\rho}_{e0}^G\Big(1+\frac{e}{k_BT_e}(\phi(\vec{x},t) - <\phi(\vec{x},t)>_{\ell})\Big)\bigg).$$
Rearranging terms and using the relation: $$\label{eq:debye_length}
\lambda_D=\bigg(\frac{\epsilon_0k_BT_e}{ne^2}\bigg)^\frac{1}{2}=\bigg(\frac{\epsilon_0k_BT_e}{\rho e}\bigg)^\frac{1}{2},$$
we arrive at: $$\label{eq:gk_poisson}
- \nabla_{\perp} \cdot \big(\varepsilon ^G(\vec{x},t)\nabla_{\perp} \phi(\vec{x},t) \big)+\frac{1}{\bar{\lambda}_{D0}} (\phi(\vec{x},t) - <\phi(\vec{x},t)>_{\ell})= \frac{1}{\epsilon_0}\Big(\rho^G_i - \bar{\rho}_{e0}^G\Big).$$
But for a normalization of the variables, equation
([\[eq:gk_poisson\]](#eq:gk_poisson){reference-type="ref"
reference="eq:gk_poisson"}) is virtually the same as the one stated in
the report by Latu (list here reference for \"Scalable Quasineutral
solver for gyrokinetic simulation\"). Equation
([\[eq:gk_poisson\]](#eq:gk_poisson){reference-type="ref"
reference="eq:gk_poisson"}) is not yet ready to solve due to the
presence of the $<\phi(\vec{x},t)>$ term. We need to average the
solution we seek, after all. To obtain an additional equation that will
help us find this term, we take the average of both sides of the
equation in the same sense that we have been averaging before: along a
magnetic field line (note that in practice, due to ergodicity, this line
may extend and fill a surface or even worse.) The barred quantities have
already been averaged so we can get them out of the averaging operator
when necessary. Averages of averages remain unchanged and thus we arrive
at:

$$\label{eq:gk_poisson_aux}
- \nabla_{\perp} \cdot \big(<\varepsilon ^G(\vec{x},t)\nabla_{\perp} \phi(\vec{x},t)>_{\ell} \big)= \frac{1}{\epsilon_0}(<\rho^G_i>_{\ell} - <\bar{\rho}_{e0}^G>_{\ell}).$$
By making two final assumptions: that the quantities involved in the
calculation of $\varepsilon^G(\vec{x},t)$ are not time-dependent but
given by the initial ion distribution, and that this quantity does not
vary along the domain of the average procedure (the magnetic field
line), we can extract $\varepsilon^G(\vec{x},t)$ from the average
operator, yielding the auxiliary equation that we need to compute
$<\phi(\vec{x},t)>_{\ell}$:

$$\label{eq:gk_poisson_aux}
- \nabla_{\perp} \cdot \big(\varepsilon ^G\nabla_{\perp} <\phi(\vec{x},t)>_{\ell} \big)= \frac{1}{\epsilon_0}(<\rho^G_i>_{\ell} - <\bar{\rho}_{e0}^G>_{\ell}).$$
The average ion charge density can be computed from
$\rho^G_i(\vec{x},t)$ which is input data for the solver. To compute the
average of the electron density we need to explicitly invoke the initial
quasineutrality condition, i.e.: $\rho_i = \rho_e$. With this,
$<\bar{\rho}^G_{e0}>_{\ell} = <\bar{\rho}^G_{i0}>_{\ell}$ and we are
able to compute all the quantities involved based on the initial ion
distribution profile.

Once we calculate $<\phi(\vec{x},t)>_{\ell}$ with equation
([\[eq:gk_poisson_aux\]](#eq:gk_poisson_aux){reference-type="ref"
reference="eq:gk_poisson_aux"}) we can use this to solve the final
version of our quasi neutral equation, after introducing all the
assumptions:

$$\label{eq:gk_poisson_final}
- \nabla_{\perp} \cdot \big(\bar{\varepsilon}^G(\vec{x},0)\nabla_{\perp} \phi(\vec{x},t) \big)+\frac{1}{\bar{\lambda}_D} (\phi(\vec{x},t) - <\phi(\vec{x},t)>_{\ell})= \frac{1}{\epsilon_0}(\rho^G_i - \bar{\rho}_{i0}^G).$$

### Exposed Interface

Fundamental type: None. It is a function that operates on other
top-level types. Function:

         sll_solve_quasi_neutral_equation( electron_T_profile_2D,
                                           initial_rho_ion_profile_2D,
                                           charge_density,
                                           phi )

### Usage

### Status

## Particle Distribution Function

### Description

### Exposed Interface

Fundamental type:

         sll_distribution_function_t

All the fundamental types in the library are implemented as pointers.
This choice has been made to ease the addition of Python bindings, in
case that an even higher-level interface is desired some day.

Constructor and destructor:

         sll_new_distribution_function( nr, ntheta, nphi, nvpar, mu )
         sll_delete_distribution_function( f )

The constructor essentially limits itself to allocating the memory for
the type. An initialization step is required afterwards:

         sll_initialize_df( boundary_type_r, 
                            boundary_type_vpar, 
                            species_charge )

The functions that get/set a particular value of a distribution function
on a node are defined as macros to be able to keep a single interface
while not risking a penalization when used in critical loops. Other
queries on this type are implemented as ordinary functions. (Need to
define this better, for instance, if some query functions would operate
on integer arguments and/or real coordinates.)

         SLL_GET_DF_VAL( i, j, k, l, df )
         SLL_SET_DF_VAL( val, i, j, k, l, df )
         sll_interpolate_df( r, theta, phi, vpar, mu )
         sll_compute_derivative( f, r, theta, phi, vpar, mu )
         sll_get_df_nr( df )
         sll_get_df_nphi( df )
         sll_get_df_ntheta( df )
         sll_get_df_nvpar( df )
         sll_get_df_mu( df )

The type also offers the services:

         sll_compute_moments( df, ... )

### Usage

### Status

## Advection Field

### Description

### Exposed Interface

Fundamental type:

         sll_advection_field_3D_t

This implies that one of the options is to have multiple
representations, for 3D, 2D, 1D.

### Usage

### Status

## Advection

### Description

### Exposed Interface

Fundamental type: None. This is a function that operates on multiple
top-level types. Function:

         sll_advect( distribution_function, 
                       advection_field, 
                     dt, 
                     space_mesh
                     scheme )

Above, `scheme` is the functional parameterization of the various
methods in use (PSM, BSL, \...) and for which we need a standardized
interface. The above assumes that we can devise a standard functional
interface.

### Usage

### Status

[^1]: We use the terms *mesh* and *grid* interchangeably.

[^2]: Or multiple coordinate transformations, in case that the physical
    domain is represented by multiple patches.

[^3]: The optimizations carried out by the remapper are related with
    finding out the minimally-sized communicators to launch the
    exchanges and the selection of the lower-level communications
    functions (alltoall vs. alltoallv, for instance). Other
    optimizations depend on how the local MPI is tuned.
