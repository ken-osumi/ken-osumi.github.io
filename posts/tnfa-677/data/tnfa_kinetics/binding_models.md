# Package model docs

This document describes the kinetic models used to fit the binding curves in this package: for each model, the reaction scheme, the binding response, the system of differential equations, and the parameters — both the **equation parameters** that are fitted and the **derived parameters** computed from them, with their formulas, units, and allowed ranges.

The model used for a given fit is recorded in its `binding_model` field, in both `summaries/fits.csv` and `data/fits/fit_details.json`. The parameter names in the tables below are exactly the keys you will find under `parameters` (equation parameters) and `derived_parameters` (derived parameters) in `fit_details.json`.

## The fitted signal

The recorded response $R(t)$ is fitted as the sum of a **binding** contribution, two deterministic **nuisance** channels — a bulk refractive-index jump and a linear baseline drift — and a stochastic measurement-noise term:

$$
R(t) = R_{\mathrm{binding}}(t) + R_{\mathrm{RI}}(t) + R_{\mathrm{drift}}(t) + \varepsilon(t).
$$

$R_{\mathrm{binding}}(t)$ is the response from the bound complex(es) and is model-specific — it is the quantity described in each model section below. $R_{\mathrm{RI}}$ and $R_{\mathrm{drift}}$ are common to every model and are defined in the Noise model section.

## Conventions

- $t$ — time within a phase, in seconds (s), measured from the phase start.
- $R(t)$ — the response at time $t$. The response unit depends on the instrument: **RU** (resonance units) for SPR data, **nm** (the biolayer wavelength shift) for BLI data. Every quantity measured in response units — $R_{\max}$, the bound-complex levels, and the RI and drift terms — is in RU for SPR and nm for BLI; the tables below write this as "RU or nm".
- The reaction schemes use $A$ for the **ligand** — the molecule immobilised on the sensor surface or probe — and $B$ for the **analyte**, the molecule in solution at concentration $C$. Typically, the ligand is the designed binder and the analyte is the target. Bound complexes such as $AB$ are tracked in response units, and $R_{\max}$ is the total ligand capacity.
- A square-bracketed symbol $[X]$ denotes the amount of species $X$. The bound species — $[AB]$, $[AB^{*}]$, $[A_2B]$, $[AB_2]$, $[AB^{(2)}]$ — are surface quantities, in response units (RU or nm). The only exception is the free analyte $[B] = C$, which is a true solution concentration in molar (M).
- Binding curves have an **association** phase (analyte present at concentration $C$, complex forming) and a **dissociation** phase (analyte removed, complex decaying). Every model uses the same equations for both phases; the dissociation phase simply sets $C = 0$.
- Parameters recorded once per phase carry the suffix `_a` (association) or `_d` (dissociation) — e.g. `ri_a` / `ri_d`. A `*_start` parameter is the value of its quantity at the **start** of that phase: for example `drift_a_start` and `drift_d_start` are the baseline-drift levels entering the association and dissociation phases. Start values are chained across the phase boundary (the dissociation start is the value reached at the end of association) so the fitted curve is continuous.

### Common parameters

Every model uses the 1:1 primary parameters below, so they are not repeated in the individual model tables:

| parameter | parameter_name | allowed_range_min | allowed_range_max | unit | description |
| --- | --- | --- | --- | --- | --- |
| $k_{\mathrm{on}}$ | `kon` | $10^{3}$ | $10^{7}$ | $\mathrm{M^{-1}\,s^{-1}}$ | Association rate constant of the 1:1 step. |
| $k_{\mathrm{off}}$ | `koff` | $10^{-5}$ | $10^{2}$ | $\mathrm{s^{-1}}$ | Dissociation rate constant of the 1:1 step. |
| $R_{\max}$ | `rmax` | $0.5$ (BLI), $50$ (SPR) | $20$ (BLI), $2000$ (SPR) | nm (BLI), RU (SPR) | Total binding capacity (signal at saturation in 1:1 model). |
| $C$ | `concentration` | — (fixed) | — (fixed) | M | Analyte concentration; held fixed at the known assay value. |
| $[AB]_0$ | `ab_a_start`, `ab_d_start` | — | — | RU or nm | 1:1 complex at the phase start (ODE initial condition; $0$ for a fresh association, otherwise chained from the previous phase). |

Every model also exposes the 1:1 affinity as a derived parameter:

| derived parameter | parameter_name | allowed_range_min | allowed_range_max | unit | description |
| --- | --- | --- | --- | --- | --- |
| $K_D = \dfrac{k_{\mathrm{off}}}{k_{\mathrm{on}}}$ | `kd` | $10^{-12}$ | $10^{-5}$ | M | Equilibrium dissociation constant of the 1:1 step. |

## Binding models

### `standard` — 1:1 binding (Langmuir)

A single analyte binds a single ligand site in 1:1 stoichiometry.

$$
A + B \;\underset{k_{\mathrm{off}}}{\overset{k_{\mathrm{on}}}{\rightleftharpoons}}\; AB
$$

The binding response is the response of the single complex, $R_{\mathrm{binding}} = [AB]$ (in signal units), and follows a single first-order equation:

$$
\frac{dR_{\mathrm{binding}}}{dt} = k_{\mathrm{on}}\,C\,(R_{\max} - R_{\mathrm{binding}}) - k_{\mathrm{off}}\,R_{\mathrm{binding}}.
$$

In the dissociation phase $C = 0$ and the response decays as $\frac{dR_{\mathrm{binding}}}{dt} = -k_{\mathrm{off}}\,R_{\mathrm{binding}}$.

**Equation parameters:** the common parameters only.

**Derived parameters:** $K_D$ (above). For the 1:1 model the apparent affinity equals the microscopic one, $K_{D,\mathrm{app}} = K_D$.

### `conformational` — binding with a conformational change

After the initial 1:1 binding, the complex isomerises into a second state $AB^{*}$ with different binding dynamics.

$$
A + B \;\underset{k_{\mathrm{off}}}{\overset{k_{\mathrm{on}}}{\rightleftharpoons}}\; AB
\;\underset{k_{\mathrm{off,conf}}}{\overset{k_{\mathrm{on,conf}}}{\rightleftharpoons}}\; AB^{*}
$$

The binding response is the sum of both complexes, $R_{\mathrm{binding}} = [AB] + [AB^{*}]$. With free capacity $S = R_{\max} - [AB] - [AB^{*}]$, the states evolve as:

$$
\begin{aligned}
\frac{d[AB]}{dt} &= k_{\mathrm{on}}\,C\,S + k_{\mathrm{off,conf}}\,[AB^{*}] - \left(k_{\mathrm{off}} + k_{\mathrm{on,conf}}\right)[AB], \\[4pt]
\frac{d[AB^{*}]}{dt} &= k_{\mathrm{on,conf}}\,[AB] - k_{\mathrm{off,conf}}\,[AB^{*}].
\end{aligned}
$$

**Equation parameters** (in addition to the common ones):

| parameter | parameter_name | allowed_range_min | allowed_range_max | unit | description |
| --- | --- | --- | --- | --- | --- |
| $k_{\mathrm{on,conf}}$ | `kon_conf` | $10^{-4}$ | $10^{3}$ | $\mathrm{s^{-1}}$ | Forward rate of the conformational step ($AB \to AB^{*}$). |
| $k_{\mathrm{off,conf}}$ | `koff_conf` | $10^{-5}$ | $10^{2}$ | $\mathrm{s^{-1}}$ | Reverse rate of the conformational step ($AB^{*} \to AB$). |
| $[AB^{*}]_0$ | `abx_a_start`, `abx_d_start` | — | — | RU or nm | Isomerised complex at the phase start (ODE initial condition). |

**Derived parameters** (in addition to $K_D$):

| derived parameter | parameter_name | allowed_range_min | allowed_range_max | unit | description |
| --- | --- | --- | --- | --- | --- |
| $K_{D,\mathrm{conf}} = \dfrac{k_{\mathrm{off,conf}}}{k_{\mathrm{on,conf}}}$ | `kd_conf` | — | — | dimensionless | Equilibrium constant of the conformational step. |
| $K_{D,\mathrm{app}} = \dfrac{K_D}{1 + k_{\mathrm{on,conf}}/k_{\mathrm{off,conf}}}$ | `kd_app` | $10^{-12}$ | $10^{-5}$ | M | Overall apparent affinity, tightened by the conformational step. |

### `bivalent_analyte` — bivalent analyte (avidity / crosslinking)

The analyte carries two equivalent arms: it first binds one surface ligand, then its second arm can crosslink a second free ligand site.

$$
A + B \;\underset{k_{\mathrm{off}}}{\overset{k_{\mathrm{on}}}{\rightleftharpoons}}\; AB,
\qquad
AB + B \;\underset{k_{\mathrm{off,c}}}{\overset{k_{\mathrm{on,c}}}{\rightleftharpoons}}\; A_2B
$$

The binding response is $R_{\mathrm{binding}} = [AB] + [A_2B]$. The second binding is on-surface, so its rate scales with the free capacity rather than with $C$. With $S = R_{\max} - [AB] - 2[A_2B]$:

$$
\begin{aligned}
\frac{d[AB]}{dt} &= 2\,k_{\mathrm{on}}\,C\,S + 2\,k_{\mathrm{off,c}}\,[A_2B] - k_{\mathrm{off}}\,[AB] - k_{\mathrm{on,c}}\,[AB]\,S, \\[4pt]
\frac{d[A_2B]}{dt} &= k_{\mathrm{on,c}}\,[AB]\,S - 2\,k_{\mathrm{off,c}}\,[A_2B].
\end{aligned}
$$

The factors of $2$ on the first-binding ($k_{\mathrm{on}}$) and $A_2B$-dissociation ($k_{\mathrm{off,c}}$) terms are statistical factors for the two equivalent arms.

The second on-rate $k_{\mathrm{on,c}}$ is a surface (per-signal) rate, so its allowed range scales with $R_{\max}$; its units are $\mathrm{RU^{-1}\,s^{-1}}$ (SPR) / $\mathrm{nm^{-1}\,s^{-1}}$ (BLI). It is also reported in its $R_{\max}$-rescaled effective form $k_{\mathrm{on,c}}^{\mathrm{eff}} = k_{\mathrm{on,c}}\,R_{\max}$ (units $\mathrm{s^{-1}}$), which has a clean, method-independent range — see the derived parameters below.

**Equation parameters** (in addition to the common ones):

| parameter | parameter_name | allowed_range_min | allowed_range_max | unit | description |
| --- | --- | --- | --- | --- | --- |
| $k_{\mathrm{on,c}}$ | `kon_complex` | $R_{\max}$-dependent (see $k_{\mathrm{on,c}}^{\mathrm{eff}}$) | $R_{\max}$-dependent (see $k_{\mathrm{on,c}}^{\mathrm{eff}}$) | $\mathrm{RU^{-1}\,s^{-1}}$ / $\mathrm{nm^{-1}\,s^{-1}}$ | On-rate of the crosslinking step (per free-site). |
| $k_{\mathrm{off,c}}$ | `koff_complex` | $10^{-5}$ | $10^{2}$ | $\mathrm{s^{-1}}$ | Off-rate of the crosslinking step. |
| $[A_2B]_0$ | `a2b_a_start`, `a2b_d_start` | — | — | RU or nm | Crosslinked (2:1) complex at the phase start (ODE initial condition). |

**Derived parameters** (in addition to $K_D$), with first-step affinity $K_{D1} = k_{\mathrm{off}}/k_{\mathrm{on}}$ and second-step affinity $K_{D2} = k_{\mathrm{off,c}}/k_{\mathrm{on,c}}$:

| derived parameter | parameter_name | allowed_range_min | allowed_range_max | unit | description |
| --- | --- | --- | --- | --- | --- |
| $k_{\mathrm{on,c}}^{\mathrm{eff}} = k_{\mathrm{on,c}}\,R_{\max}$ | `kon_complex_eff` | $10^{-4}$ | $10^{3}$ | $\mathrm{s^{-1}}$ | Effective (Rmax-scaled) on-rate of the crosslinking step. |
| $K_{D,\mathrm{app}} = \dfrac{K_{D1}\,K_{D2}}{R_{\max} + 2\,K_{D2}}$ | `kd_app` | $10^{-12}$ | $10^{-5}$ | M | Overall apparent affinity including the avidity contribution; the value to compare against the `standard` $K_D$. |

## Noise model

On top of the binding response, every model adds two deterministic nuisance channels and the AR(1) measurement noise of The fitted signal. Each phase (association / dissociation) carries its own copies of the nuisance parameters.

### Refractive-index (RI) jump

At the phase boundaries, the bulk refractive index changes abruptly, producing a step in the signal that relaxes over a short timescale. Each phase is modelled by

$$
\frac{d\,R_{\mathrm{RI}}}{dt} = -k_{\mathrm{RI}}\left(R_{\mathrm{RI}} + r_i\right),
\qquad
R_{\mathrm{RI}}(t) = \left(R_{\mathrm{RI}}^{\mathrm{start}} + r_i\right) e^{-k_{\mathrm{RI}}\,t} - r_i,
$$

where $r_i$ is the asymptotic RI offset for the phase, $R_{\mathrm{RI}}^{\mathrm{start}}$ is the value carried over from the end of the previous phase, and $k_{\mathrm{RI}} = 10^{6}\ \mathrm{s^{-1}}$ is a fixed (very fast) rate — so the RI channel is effectively an instantaneous jump that relaxes from $R_{\mathrm{RI}}^{\mathrm{start}}$ towards $-r_i$. The association and dissociation phases have independent offsets $r_i^{(a)}$ and $r_i^{(d)}$.

### Linear drift

Slow baseline drift (thermal effects, evaporation, instrument warm-up) is modelled as a linear ramp:

$$
R_{\mathrm{drift}}(t) = R_{\mathrm{drift}}^{\mathrm{start}} + s\,t,
$$

where $s$ is a single drift slope shared between the two phases and $R_{\mathrm{drift}}^{\mathrm{start}}$ is a per-phase starting value, chained across the phase boundary so the drift is continuous.

### Noise parameters

These are reported in `fit_details.json` but are not of biological interest.

| parameter | parameter_name | allowed_range_min | allowed_range_max | unit | description |
| --- | --- | --- | --- | --- | --- |
| $r_i^{(a)}$, $r_i^{(d)}$ | `ri_a`, `ri_d` | data-derived | data-derived | RU or nm | RI offset; bounds set per curve from the data. |
| $R_{\mathrm{RI}}^{\mathrm{start}}$ | `ri_a_start`, `ri_d_start` | — | — | RU or nm | RI value at the phase start (chained across phases). |
| $s$ | `drift_slope` | — | — | (RU or nm)/s | Linear baseline drift slope (shared between phases). |
| $R_{\mathrm{drift}}^{\mathrm{start}}$ | `drift_a_start`, `drift_d_start` | — | — | RU or nm | Drift value at the phase start (chained across phases). |