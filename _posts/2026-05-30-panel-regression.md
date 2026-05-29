---
layout: post
title:  "An Introduction to Panel Regression"
#categories: econometrics
---

Panel data — also called *longitudinal* or *cross-sectional time-series* data —
follow the same units (individuals, firms, countries) repeatedly over time.
This combination of a **cross-sectional** dimension and a **time** dimension is
what makes panel regression so powerful: it lets us control for characteristics
we cannot observe, something a single cross-section or a single time series can
never do on its own.

## The general model

A typical panel regression for unit $i$ at time $t$ is written as

$$
y_{it} = \alpha + \boldsymbol{\beta}^{\top} \mathbf{x}_{it} + c_i + u_{it},
\qquad i = 1,\dots,N, \quad t = 1,\dots,T,
$$

where $y_{it}$ is the outcome, $\mathbf{x}_{it}$ is a $K \times 1$ vector of
explanatory variables, $\boldsymbol{\beta}$ collects the slope coefficients,
$c_i$ is an **unobserved individual effect**, and $u_{it}$ is the idiosyncratic
error. The whole modelling question in panel econometrics boils down to: *what
do we assume about $c_i$?*

## Pooled OLS

The simplest approach ignores the panel structure entirely and runs ordinary
least squares on the stacked data, effectively folding $c_i$ into the error:

$$
y_{it} = \alpha + \boldsymbol{\beta}^{\top}\mathbf{x}_{it} + \varepsilon_{it},
\qquad \varepsilon_{it} = c_i + u_{it}.
$$

Pooled OLS is consistent only if $c_i$ is uncorrelated with the regressors,
i.e. $\operatorname{Cov}(\mathbf{x}_{it}, c_i) = 0$. If a firm's unobserved
management quality ($c_i$) is correlated with its observed investment
($\mathbf{x}_{it}$), this assumption fails and the estimates are biased. This
**omitted variable bias** is precisely what fixed and random effects are
designed to address.

## The fixed effects (within) estimator

The fixed effects (FE) model treats $c_i$ as a parameter to be eliminated rather
than estimated. The trick is to average the equation over time for each unit,

$$
\bar{y}_{i} = \alpha + \boldsymbol{\beta}^{\top}\bar{\mathbf{x}}_{i} + c_i + \bar{u}_{i},
$$

and then subtract this from the original equation. The **within
transformation** removes the time-invariant term $c_i$ completely:

$$
(y_{it} - \bar{y}_i) = \boldsymbol{\beta}^{\top}(\mathbf{x}_{it} - \bar{\mathbf{x}}_i) + (u_{it} - \bar{u}_i).
$$

Because $c_i$ has vanished, we can estimate $\boldsymbol{\beta}$ by OLS on the
demeaned data even when $c_i$ is correlated with the regressors. The FE
estimator is

$$
\hat{\boldsymbol{\beta}}_{FE} =
\left( \sum_{i=1}^{N}\sum_{t=1}^{T} \ddot{\mathbf{x}}_{it}\, \ddot{\mathbf{x}}_{it}^{\top} \right)^{-1}
\left( \sum_{i=1}^{N}\sum_{t=1}^{T} \ddot{\mathbf{x}}_{it}\, \ddot{y}_{it} \right),
$$

where $\ddot{z}_{it} = z_{it} - \bar{z}_i$ denotes a time-demeaned variable.
The great strength of FE is that it controls for *all* time-invariant
confounders, observed or not. The price is that any variable that does not
change over time (gender, a country's geography) is wiped out by the
transformation and its coefficient cannot be identified.

## The random effects estimator

The random effects (RE) model keeps $c_i$ in the equation but assumes it is a
*random draw* uncorrelated with the regressors:

$$
\operatorname{Cov}(\mathbf{x}_{it}, c_i) = 0,
\qquad c_i \sim (0, \sigma_c^2), \qquad u_{it} \sim (0, \sigma_u^2).
$$

Under this assumption the composite error $\varepsilon_{it} = c_i + u_{it}$ is
serially correlated within a unit, so plain OLS is inefficient. RE instead
applies **feasible generalized least squares**, which is equivalent to a
*partial* (quasi-) demeaning of the data:

$$
(y_{it} - \theta\,\bar{y}_i) = \alpha(1-\theta) +
\boldsymbol{\beta}^{\top}(\mathbf{x}_{it} - \theta\,\bar{\mathbf{x}}_i) +
(\varepsilon_{it} - \theta\,\bar{\varepsilon}_i),
$$

where the weight $\theta$ lies between 0 and 1 and is given by

$$
\theta = 1 - \sqrt{\dfrac{\sigma_u^2}{\sigma_u^2 + T\,\sigma_c^2}}.
$$

Notice the two limiting cases. When $\theta = 0$ no demeaning occurs and RE
collapses to pooled OLS; when $\theta = 1$ we recover full demeaning and RE
coincides with fixed effects. RE uses both the **within** and the **between**
variation, so when its assumption holds it is more efficient than FE — it uses
more of the information in the data.

## Choosing between FE and RE: the Hausman test

The central trade-off is consistency versus efficiency. FE is consistent whether
or not $c_i$ is correlated with the regressors; RE is more efficient but only
consistent when that correlation is zero. The **Hausman test** formalises the
choice by comparing the two coefficient vectors:

$$
H = \left(\hat{\boldsymbol{\beta}}_{FE} - \hat{\boldsymbol{\beta}}_{RE}\right)^{\top}
\left[ \operatorname{Var}(\hat{\boldsymbol{\beta}}_{FE}) -
\operatorname{Var}(\hat{\boldsymbol{\beta}}_{RE}) \right]^{-1}
\left(\hat{\boldsymbol{\beta}}_{FE} - \hat{\boldsymbol{\beta}}_{RE}\right).
$$

Under the null hypothesis that $c_i$ is uncorrelated with the regressors, both
estimators are consistent and should be similar, so $H$ is small; the statistic
follows a chi-squared distribution, $H \sim \chi^2_{K}$, with $K$ the number of
time-varying regressors. A large $H$ (small $p$-value) rejects the null and
points to **fixed effects**; a small $H$ favours the more efficient **random
effects** model.

## Two-way effects and clustered errors

Often we want to remove common shocks that hit every unit in a given period —
a recession, a new regulation. The **two-way fixed effects** model adds a time
effect $\delta_t$ alongside the individual effect:

$$
y_{it} = \boldsymbol{\beta}^{\top}\mathbf{x}_{it} + c_i + \delta_t + u_{it}.
$$

Finally, panel errors are rarely independent: observations on the same unit are
correlated over time. To get valid inference we therefore report **standard
errors clustered by unit**, which allow for arbitrary correlation within each
$i$ while assuming independence across units.

## Takeaways

Panel regression turns repeated observations into a tool for controlling
unobserved heterogeneity. **Pooled OLS** is the naive benchmark; **fixed
effects** sweeps away every time-invariant confounder via the within
transformation; **random effects** trades that robustness for efficiency under a
stronger exogeneity assumption; and the **Hausman test** tells us which to
trust. Master the algebra of demeaning and you have understood the heart of
panel econometrics.
