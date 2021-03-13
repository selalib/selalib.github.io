# SeLaLib

*Modular library for the kinetic and gyrokinetic simulation of tokamak
plasmas by the semi-lagrangian or particle-in-cell methods*

In the context of the [Inria Project Lab
FRATRES](https://team.inria.fr/ipl-fratres/) and a longstanding
collaboration between the CEA-Cadarache (French Atomic Energy
Commision), the [MINGUS](https://www.inria.fr/equipes/mingus) 
and [TONUS](https://www.inria.fr/equipes/tonus) INRIA Project
Teams based respectively at the University of Rennes and
Strasbourg and the [Max-Planck-Institut für Plasmaphysik
(IPP)](https://www.ipp.mpg.de/ippcms/eng/for/bereiche/numerik) in
Garching, we are currently developing the Selalib library.

The SeLaLib software library is a modular library for kinetic and
gyrokinetic simulations of plasmas in fusion energy devices by
semi-Lagrangian or particle-in-cell methods.

The SeLaLib project arose from the need of researchers to develop
numerical methods with simplified test cases while also having
independently tested modules that would facilitate gradual changes in
existing production code. While originally envisioned to be specialized
on the semi-lagrangian method, the abstractions that we have built can
be used with other types of approaches, such as particle-in-cell.

SeLaLib contains a collection of individual building blocks for the
parallel simulation of the Vlasov equations and the gyrokinetic equation
either based on semi-Lagrangian schemes or particle methods. Besides
numerical algorithms the library provides low-level utilities,
input-output modules as well as parallelization strategies. Moreover, a
collection of simulations for typical test cases with various
discretization schemes supplements the library.

## Acknowledgements

SeLalib has many {doc}`contributors`, many of whom are member of these
{doc}`institutions`.
