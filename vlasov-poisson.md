# Vlasov--Poisson

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

## Landau damping

$$
f_0 = (1+\varepsilon \sum_j \cos(k_jx_j)) /\sqrt{2\pi}  (\exp(-0.5\|\mathbf{v}\|^2))
$$

- Linear Landau damping: $k_x=0.5$, $\varepsilon = 0.01$
- Non-linear Landau damping: $k_x =0.5$, $\varepsilon = 0.5$

## Two-stream instability:

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

## Bump-on-tail:

$$
 f_0(x,v) = \left(\frac{9}{10 \sqrt{2\pi}} \mathrm{e}^{-0.5 v^2} 
 + \frac{2}{10 \sqrt{2\pi}} \mathrm{e}^{-2 (v-4.5)^2} \right) (1+\alpha \cos(kx)),
$$

## KEEN waves (1d1v) (cf. {cite}`Afeyan14`): ponderomotive force drive electric field added

$$
 E_{\mathrm{pond}}(t,x) = a_{\mathrm{Dr}} k_{\mathrm{Dr}} a(t) \sin(k_{\mathrm{Dr}}x-\omega_{\mathrm{Dr}}t).
$$

