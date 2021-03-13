# Equations

## Advection

$$
\partial_t f(t,\mathbf{x}) + A \cdot \nabla_{\mathbf{x}} f(t,\mathbf{x}) = 0.
$$

###  Rigit body rotation: 

$$A = \begin{pmatrix}
        0 & -1 \\ 1 & 0
        \end{pmatrix}
$$

### Translation

$A$ diagonal, time independent.

### Swirling flow 

Here we apply a time dependent velocity field {cite}`Crouseilles15a`
The swirling flow distorts the initial distribution and reaching a maximum 
distortion at $t=T/2$. At that point the flow reverses and the distribution 
returns to its initial value.

$$
\partial_t f + \partial_x \left(\sin^2(\pi x) \sin(2\pi y)g(t) f\right) + \partial_y \left(-\sin^2(\pi y) \sin (2\pi x) g(t)f\right) = 0.
$$

where $g(t) = cos(\pi t / T ) $.

## Vlasov--Poisson

$$
\partial_t f(t,\mathbf{x},\mathbf{v}) + 
\mathbf{v}\nabla_{\mathbf{x}} f(t,\mathbf{x},\mathbf{v}) + 
\frac{q}{m} \mathbf{E}(t,\mathbf{x}) \nabla_{\mathbf{v}} f(t,\mathbf{x},\mathbf{v}) = 0
$$

$$
-\Delta \phi(t,\mathbf{x}) =  1- \rho(t, \mathbf{x})
$$

- Dimensions: 1d1v, 2d2v, 3d3v
- Identifier in SeLaLib: `vp`
- Typical initial conditions:

### Landau damping

$$
f_0 = (1+\varepsilon \sum_j \cos(k_jx_j)) /\sqrt{2\pi}  (\exp(-0.5\|\mathbf{v}\|^2))
$$

- Linear Landau damping: $k_x=0.5$, $\varepsilon = 0.01$
- Non-linear Landau damping: $k_x =0.5$, $\varepsilon = 0.5$

### Two-stream instability:

  - Version 1 (cf. {cite}`Crouseilles08`):

   $$
    f_0 = (1+\varepsilon \cos(k_xx))0.5 /\sqrt{2\pi}  (\exp(-0.5 (v-v_0)^2)+ \exp(-0.5(v+v_0)^2))
   $$

  - Version 2 (cf. {cite}`Filbet03`):

   $$
    f_0 = \frac{2}{7\sqrt{2\pi}}\left( 1+ 5v^2 \right)
          \left( 1+ \alpha \left( (\cos(2kx)+\cos(3kx) 
          \right)/1.2 + \cos(kx) \right) \mathrm{e}^{-v^2/2}
   $$

### Bump-on-tail:

$$
 f_0(x,v) = \left(\frac{9}{10 \sqrt{2\pi}} \mathrm{e}^{-0.5 v^2} 
 + \frac{2}{10 \sqrt{2\pi}} \mathrm{e}^{-2 (v-4.5)^2} \right) (1+\alpha \cos(kx)),
$$

### KEEN waves (1d1v) (cf. {cite}`Afeyan14`): ponderomotive force drive electric field added

$$
 E_{\mathrm{pond}}(t,x) = a_{\mathrm{Dr}} k_{\mathrm{Dr}} a(t) \sin(k_{\mathrm{Dr}}x-\omega_{\mathrm{Dr}}t).
$$

## Vlasov--Maxwell

- Equations 1d2v: 

$$
\begin{aligned}
      &&\partial_t f(t,x,\mathbf{v}) + v_1 \partial_{x_1} f(t,x,\mathbf{v}) + \mathbf{E}(t, x) \cdot \nabla_{\mathbf{v}} f(t, x, \mathbf{v}) + B v_2 \partial_{v_1} f(t,x,\mathbf{v}) - B v_1 \partial_{v_2} f(t,x,\mathbf{v}) = 0, \\
      && \partial_t B(t,x) = - \partial_{x_1} E_2, \\
      && \partial_t E_2 = - \partial_{x_1} B - \int_{\mathsf{R}^1} v_2 f(t, x, \mathbf{v}) \, \mathrm{d}v,\\
      && \partial_t E_1 = - \int_{\mathsf{R}^1} v_1 f(t, x, \mathbf{v}) \, \mathrm{d}v.\end{aligned}
$$


### Weibel instability 

Initial condition (cf. {cite}`Crouseilles15`):

$$
f_0(x_1,v_1,v_2) = \frac{1}{\pi v_{th} \sqrt{T_r}} \mathrm{e}^{-(v_1^2+v_2^2/T_r)/v_{th}^2}(1+\alpha \cos(kx_1)
$$

Initial magnetic field: $B_0(x_1) = \beta \cos(kx_1)$

## Guiding center

Equation in polar coordinates:

$$
\partial_t f(t,r,\theta) - \frac{\partial_{\theta}\Phi(r, \theta)}{r} \partial_r f(t,r,\theta) +  \frac{\partial_{r}\Phi}{r} \partial_{\theta} f  = 0.
$$

Diocotron instability {cite}`Crouseilles14`

## Drift kinetic

Equation in polar coordinates: 

$$
\begin{aligned}
    &&\partial_t f(t,r, \theta, z, v) - \frac{\partial_{\theta}\phi}{r} \partial_r f(t,r, \theta, z, v) +\frac{\partial_r \phi}{r} \partial_{\theta} f(t,r, \theta, z, v) + \\
    && v \partial_z f(t,r, \theta, z, v) - \partial_z \phi \partial_v f(t,r, \theta, z, v) = 0,
    \\
    && - \left( \partial_r^2 \phi + \left( \frac{1}{r} + \frac{\partial_r n_0(r)}{n_0(r)} \partial_r \phi + \frac{1}{r^2} \partial^2_{\theta} \phi \right) \right)+ \frac{1}{T_e(r)} \left( \phi - \langle \phi \rangle \right) = \frac{1}{n_0(r)} \int_{\mathsf{R}} f(t,r, \theta, z, v) \, \mathrm{d}v -1.\end{aligned}
$$

- Dimensions: 3d1v
- Identifier in SeLaLib: `dk`
- References {cite}`Grandgirard06` {cite}`Crouseilles14`
