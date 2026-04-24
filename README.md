# frame-guided-monte-carlo

A minimal scalar case demonstration of the FRAME guided Monte Carlo
reweighting pipeline. The goal of this first version is to stand up the
path sampling machinery and verify that the self normalized importance
sampling estimator built on top of an admissibility cost produces the
analytically expected behavior, with variance scaling and weight
degeneracy diagnostics that a reader can reproduce in under a minute.

## What this does

Paths $\gamma: [0, T] \to \mathbb{R}$ are drawn from a Brownian baseline
$P_0$ with $\gamma(0) = \theta_0$. An admissibility cost penalizes
deviation from a linear reference trajectory $p(t)$ that runs from
$\theta_0$ to a target $\theta_\ast$:

$$
C(\gamma) \;=\; \int_0^T \bigl(\gamma(t) - p(t)\bigr)^2 \, dt.
$$

The FRAME measure $\mu_F$ is the Boltzmann tilt of the baseline,

$$
\frac{d\mu_F}{dP_0}(\gamma) \;\propto\; \exp\bigl(-\beta \, C(\gamma)\bigr),
$$

and the scalar endpoint observable is
$\phi(\theta_T) = (\theta_T - \theta_\ast)^2$. Two estimators are
computed on the same pool of iid Brownian samples:

$$
\hat I_N^{\text{naive}} \;=\; \frac{1}{N} \sum_{i=1}^N \phi\bigl(\theta_T^{(i)}\bigr)
\qquad\text{(estimates } E_{P_0}[\phi]\text{)}
$$

$$
\hat I_N^{\text{FRAME}} \;=\; \frac{\sum_{i=1}^N w(\gamma_i) \, \phi\bigl(\theta_T^{(i)}\bigr)}{\sum_{i=1}^N w(\gamma_i)}
\qquad\text{(estimates } E_{\mu_F}[\phi]\text{)}
$$

with $w(\gamma) = \exp(-\beta \, C(\gamma))$. These are by design two
different integrals. The naive estimator asks what the endpoint
deviation looks like under unconstrained Brownian motion. The FRAME
estimator asks the same question under the measure that has been
reweighted to concentrate on admissible paths.

## Why this is the v0.1

This version deliberately avoids Bloch disk structure, operator
valued observables, and every other piece of richer FRAME geometry.
It is a minimum viable artifact whose only job is to show that the
reweighting pipeline is well defined, numerically stable, and produces
the analytically correct limits. Once those three properties are
visible on plots, upgrades to richer path spaces become incremental
rather than speculative.

## Results at a glance

Running `python run_v01.py` with the defaults produces two figures.
The convergence figure shows both estimators with one standard
deviation envelopes across 200 independent seeds on a log spaced grid
of sample counts from 50 to 5000. The variance panel confirms the
expected $\mathcal{O}(1/N)$ decay for both estimators. The beta sweep
figure shows the FRAME estimate descending monotonically from the
naive value at $\beta = 0$ toward zero as $\beta$ grows, while the
Kish effective sample fraction $N_{\mathrm{eff}} / N$ degrades
smoothly from unity, flagging the onset of weight collapse.

Default configuration reports:

```
theta_star = 1.0, beta = 2.0
At N = 5000:
  naive MC estimate:  1.9978 +/- 0.0327
  FRAME MC estimate:  0.7157 +/- 0.0124
  FRAME effective sample fraction: 0.639
```

The naive value matches the closed form
$E_{P_0}[\phi] = \mathrm{Var}(\theta_T) + (\theta_\ast - \theta_0)^2 = 1 + 1 = 2$.

## Files

| file                | purpose                                                       |
|---------------------|---------------------------------------------------------------|
| `frame_mc.py`       | path sampler, cost functional, observable, estimators         |
| `run_v01.py`        | convergence sweep, beta sweep, figure generation              |
| `figures/`          | output directory written by `run_v01.py`                      |
| `requirements.txt`  | numpy, matplotlib                                             |

`frame_mc.py` deliberately has no class hierarchy, no configuration
file, and no plotting code. Every function has a single
responsibility and a clean numpy signature so later versions can wrap
it without refactoring.

## Running

```bash
pip install -r requirements.txt
python run_v01.py
```

Two PNGs land in `figures/`. Runtime on a laptop is a few seconds.

## Roadmap

**v0.2: Bloch disk governed transport.**
Replace scalar $\theta$ with a state $r \in \mathbb{D}^2$ on the
Bloch disk. The reference trajectory becomes a geodesic segment on
the disk, and the observable becomes
$\phi(r_T) = \lVert r_T - p(T) \rVert^2$. The admissibility cost
generalizes to a path functional appropriate to the disk geometry.
The self normalized estimator structure is unchanged.

**v0.3: comparison against standard importance sampling.**
Add a direct Metropolis sampler for $\mu_F$ so the SNIS estimator can
be benchmarked against plain Monte Carlo drawn from the tilted
measure. Add a cross entropy tuned Gaussian proposal as a second
baseline. The headline result, if it holds, is that the FRAME
weighted estimator reaches a fixed relative error in fewer samples
than a standard importance sampling proposal on the same target.

## Scope

This repository is a computational demonstration of one piece of a
larger research program. It is not a theory paper and not a
production pipeline. What is in scope:

- the self normalized importance sampling estimator built from an
  admissibility weighted path measure
- the convergence and weight degeneracy diagnostics that establish
  the estimator is well defined
- a clean module boundary that permits richer path spaces in later
  versions without refactoring

What is deliberately out of scope for this repository:

- the theoretical justification for choosing a particular
  admissibility cost in a given domain
- the broader research program that motivates it
- any domain specific application or deployment

References to theoretical work will be added as those manuscripts
become publicly available.

## Semantics note

The naive and FRAME estimators here target different integrals. This
is the honest framing of v0.1: we are showing that the reweighting
transforms the estimand, not that FRAME reduces the variance of the
same estimand. The apples to apples variance comparison lives in
v0.3, where both methods estimate $E_{\mu_F}[\phi]$ and can be
measured against each other on identical ground.

## Author

Matthew Loving, PhD
ORS Quantum LLC
matthew.loving@orsquantum.com

## Citing

If you use this code or refer to it in academic work, please cite
via the `CITATION.cff` file in this repository or the following
short form:

> Loving, M. (2026). *frame-guided-monte-carlo* (v0.1.0)
> [Computer software]. ORS Quantum LLC.

## License

Apache 2.0. See LICENSE.
