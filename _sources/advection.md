# Advection

$$
\partial_t f(t,\mathbf{x}) + A \cdot \nabla_{\mathbf{x}} f(t,\mathbf{x}) = 0.
$$

##  Rigit body rotation: 

$$A = \begin{pmatrix}
        0 & -1 \\ 1 & 0
        \end{pmatrix}
$$

## Translation

$A$ diagonal, time independent.

## Swirling flow 

Here we apply a time dependent velocity field {cite}`Crouseilles15a`
The swirling flow distorts the initial distribution and reaching a maximum 
distortion at $t=T/2$. At that point the flow reverses and the distribution 
returns to its initial value.

$$
\partial_t f + \partial_x \left(\sin^2(\pi x) \sin(2\pi y)g(t) f\right) + \partial_y \left(-\sin^2(\pi y) \sin (2\pi x) g(t)f\right) = 0.
$$

where $g(t) = cos(\pi t / T ) $.
