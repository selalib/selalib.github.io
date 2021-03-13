# User guide

Selalib is the **Se**mi **La**grangian **Lib**rary, a collection of types and
its associated methods that are useful for creating parallel plasma
physics simulations that use the semilagrangian methodology for the
solution of the *Vlasov* equation. In its design, we have attempted to
expose to the users an interface that expresses as naturally as possible
the problems that arise when using the semilagrangian approach. However,
most of the modules contained in the library are not intrinsically
related with the semilagrangian method, but are more general utilities
which could be used in different contexts. A principle in the
development of the library has been to continually identify specific
functionalities and to attempt to promote them into their own standalone
module, with a sufficiently general interface which may still permit a
reasonably efficient implementation.

Selalib is structured in (loosely defined) layers. One can think about
these layers as libraries in their own right. A given layer can use the
capabilities offered by a lower layer through the exposed interface, but
never from a higher level. The layer with the lowest level of
abstraction presently contains basic utilities like memory allocators,
assertions and basic numeric types. The second layer is composed of
numerical and parallel utilities. The third and highest level of the
library contains the semi-lagrangian methodology tools, types and
methods. This manual will ultimately describe each of these layers and
the modules therein.

Most of the native types and operations provided by Selalib are prefixed
by `sll_`. In this way you can at least have an expectation of finding
documentation (if a user) and a starting point of where to start looking
if you wish to dive into some particular aspect of an implementation (if
a developer). This convention has not been always followed, so it would
be advantageous to slowly but surely include this prefix throughout as a
first means to create a Selalib namespace.

Selalib is written in Fortran 2003. We have deviated from the original
intent to make Selalib fully f95-compatible for multiple reasons: the
newer standard permits a separation of interface and implementation
which is desirable in a development like this, the reduction in coding
that is obtained with the use of abstract classes is appealing for a
project with quite limited manpower as this one, and finally, the desire
to make this product as forward-looking as possible.

Some features of the library are implemented as *macros*. To the user of
the library, it makes no difference whether some functionality is
implemented in the form of a procedure or a macro, with the exception
that presently, macro names are written in `ALL-CAPS`. The use of the
macro is required to offer certain capabilities, like informative error
messages. For a developer, the use of the macros is needed in many cases
to reduce code redundancy. The need for the use of macros will hopefully
be more understandable when the reader sees the behavior of calls to
simple macros like `SLL_ALLOCATE()` or `SLL_ASSERT()`.

A macro is a pre-processor directive. Selalib uses only very simple
macros that are handled by *fpp*, the Fortran Pre-Processor. *fpp*'s
capabilities are very limited and one peculiarity of its output is that
macros are expanded into a single long line, which can easily surpass
the 132 character limit that Fortran systems have. For this reason
alone, the compilation of Selalib requires the use of a compilation
flag: `-ffree-line-length-none` (in **gfortran**) or its equivalent in
another compiler. On a similar vein, some compilers require the
extension `.F90` (as opposed to `.f90`) in order to apply a
preprocessing step. Thus, all files in Selalib use the `.F90` extension.
Clients of the library which want to use the macro facilities should
thus also use this extension as well.

