# Particle-In-Cell

## Particle Distribution Function

### Description

### Exposed Interface

Fundamental type:

```
sll_distribution_function_t
```

All the fundamental types in the library are implemented as pointers.
This choice has been made to ease the addition of Python bindings, in
case that an even higher-level interface is desired some day.

Constructor and destructor:

```
sll_new_distribution_function( nr, ntheta, nphi, nvpar, mu )
sll_delete_distribution_function( f )
```

The constructor essentially limits itself to allocating the memory for
the type. An initialization step is required afterwards:

```
sll_initialize_df( boundary_type_r, 
                   boundary_type_vpar, 
                   species_charge )
```

The functions that get/set a particular value of a distribution function
on a node are defined as macros to be able to keep a single interface
while not risking a penalization when used in critical loops. Other
queries on this type are implemented as ordinary functions. (Need to
define this better, for instance, if some query functions would operate
on integer arguments and/or real coordinates.)

```
SLL_GET_DF_VAL( i, j, k, l, df )
SLL_SET_DF_VAL( val, i, j, k, l, df )
sll_interpolate_df( r, theta, phi, vpar, mu )
sll_compute_derivative( f, r, theta, phi, vpar, mu )
sll_get_df_nr( df )
sll_get_df_nphi( df )
sll_get_df_ntheta( df )
sll_get_df_nvpar( df )
sll_get_df_mu( df )
```

The type also offers the services:

```
sll_compute_moments( df, ... )
```

### Usage

### Status

