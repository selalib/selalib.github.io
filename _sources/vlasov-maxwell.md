# Vlasov--Maxwell

- Equations 1d2v: 

$$
\begin{aligned}
      &&\partial_t f(t,x,\mathbf{v}) + v_1 \partial_{x_1} f(t,x,\mathbf{v}) + \mathbf{E}(t, x) \cdot \nabla_{\mathbf{v}} f(t, x, \mathbf{v}) + B v_2 \partial_{v_1} f(t,x,\mathbf{v}) - B v_1 \partial_{v_2} f(t,x,\mathbf{v}) = 0, \\
      && \partial_t B(t,x) = - \partial_{x_1} E_2, \\
      && \partial_t E_2 = - \partial_{x_1} B - \int_{\mathsf{R}^1} v_2 f(t, x, \mathbf{v}) \, \mathrm{d}v,\\
      && \partial_t E_1 = - \int_{\mathsf{R}^1} v_1 f(t, x, \mathbf{v}) \, \mathrm{d}v.\end{aligned}
$$


## Weibel instability 

Initial condition (cf. {cite}`Crouseilles15`):

$$
f_0(x_1,v_1,v_2) = \frac{1}{\pi v_{th} \sqrt{T_r}} \mathrm{e}^{-(v_1^2+v_2^2/T_r)/v_{th}^2}(1+\alpha \cos(kx_1)
$$

Initial magnetic field: $B_0(x_1) = \beta \cos(kx_1)$

