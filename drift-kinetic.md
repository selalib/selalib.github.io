# Drift kinetic

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
