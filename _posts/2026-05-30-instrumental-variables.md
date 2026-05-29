---
layout: post
title:  "Instrumental Variables: Estimating Causal Effects When Regressors Are Endogenous"
#categories: econometrics
---

One of the central problems in empirical economics is **endogeneity**: the
explanatory variable we care about is correlated with the error term, so
ordinary least squares (OLS) no longer recovers a causal effect. Instrumental
variables (IV) estimation is the classic remedy. This post explains the
intuition, the assumptions, the algebra of two-stage least squares, and how we
test whether an instrument is doing its job.

## The endogeneity problem

Consider the simple regression

$$
y_i = \beta_0 + \beta_1 x_i + u_i,
$$

where $y_i$ is the outcome, $x_i$ the regressor of interest, and $u_i$ the
error. OLS is consistent only if the regressor is **exogenous**:

$$
\mathbb{E}[u_i \mid x_i] = 0 \quad\Longrightarrow\quad \operatorname{Cov}(x_i, u_i) = 0.
$$

When this fails — $\operatorname{Cov}(x_i, u_i) \neq 0$ — the OLS estimator is
biased and inconsistent. We can see exactly why by writing out the probability
limit of the slope estimator:

$$
\operatorname{plim}\,\hat{\beta}_1^{OLS}
= \beta_1 + \frac{\operatorname{Cov}(x_i, u_i)}{\operatorname{Var}(x_i)}.
$$

The second term is the **asymptotic bias**; it vanishes only under exogeneity.
Three common sources push $\operatorname{Cov}(x_i, u_i)$ away from zero:

- **Omitted variables.** A determinant of $y$ that also correlates with $x$ is
  buried in $u$ — for example, unobserved ability when regressing wages on
  schooling.
- **Simultaneity.** $y$ and $x$ are determined jointly, as with price and
  quantity in a market equilibrium.
- **Measurement error.** When $x$ is observed with noise, the mismeasured
  regressor is mechanically correlated with the error.

## The idea of an instrument

An instrument $z_i$ is a variable that affects $y$ **only through** $x$. It lets
us isolate variation in $x$ that is unrelated to $u$, and use only that "clean"
variation to estimate $\beta_1$. A valid instrument must satisfy two conditions.

**1. Relevance.** The instrument must actually move the endogenous regressor:

$$
\operatorname{Cov}(z_i, x_i) \neq 0.
$$

This is testable — it shows up in the first-stage regression.

**2. Exogeneity (the exclusion restriction).** The instrument must be
uncorrelated with the structural error:

$$
\operatorname{Cov}(z_i, u_i) = 0.
$$

This says $z$ influences $y$ *exclusively* through $x$ and has no direct effect
of its own. Crucially, this condition is **not** testable in a just-identified
model; it must be argued from economic theory and institutional knowledge.

A classic example is Angrist and Krueger's use of **quarter of birth** as an
instrument for years of schooling: compulsory-schooling laws tied to birth dates
shift education, but a person's birth quarter plausibly has no direct effect on
their earnings.

## The IV estimator

With a single instrument for a single endogenous regressor, the IV estimator has
a clean closed form:

$$
\hat{\beta}_1^{IV} = \frac{\operatorname{Cov}(z_i, y_i)}{\operatorname{Cov}(z_i, x_i)}.
$$

The numerator captures how the instrument moves the outcome; the denominator how
it moves the regressor. Dividing one by the other strips out everything except
the channel running through $x$. Substituting the structural equation shows its
consistency:

$$
\operatorname{plim}\,\hat{\beta}_1^{IV}
= \beta_1 + \frac{\operatorname{Cov}(z_i, u_i)}{\operatorname{Cov}(z_i, x_i)}
= \beta_1,
$$

where the last equality follows because the exclusion restriction kills the
numerator while relevance keeps the denominator nonzero.

## Two-stage least squares

When there are multiple instruments or multiple regressors, IV generalises to
**two-stage least squares (2SLS)**. Collect the exogenous and endogenous
regressors and the instruments into matrices and proceed in two steps.

**First stage.** Regress the endogenous regressor on all exogenous variables and
instruments, and keep the fitted values:

$$
x_i = \pi_0 + \pi_1 z_i + v_i, \qquad
\hat{x}_i = \hat{\pi}_0 + \hat{\pi}_1 z_i.
$$

The fitted value $\hat{x}_i$ is the part of $x_i$ explained by the (exogenous)
instrument — the clean variation.

**Second stage.** Regress the outcome on the fitted values:

$$
y_i = \beta_0 + \beta_1 \hat{x}_i + \varepsilon_i.
$$

In matrix form, with regressors $\mathbf{X}$ and instruments $\mathbf{Z}$, the
2SLS estimator is

$$
\hat{\boldsymbol{\beta}}_{2SLS}
= \left( \mathbf{X}^{\top} \mathbf{P}_Z \mathbf{X} \right)^{-1}
  \mathbf{X}^{\top} \mathbf{P}_Z \mathbf{y},
\qquad
\mathbf{P}_Z = \mathbf{Z}(\mathbf{Z}^{\top}\mathbf{Z})^{-1}\mathbf{Z}^{\top},
$$

where $\mathbf{P}_Z$ is the projection matrix onto the column space of the
instruments. A practical warning: never compute the second-stage standard errors
from a manual two-step regression — they are wrong because they ignore that
$\hat{x}_i$ is estimated. Always use a dedicated IV/2SLS command.

## Weak instruments

Relevance is not enough; the instrument must be **strongly** correlated with the
regressor. If $\operatorname{Cov}(z_i, x_i)$ is small, the IV denominator is
near zero, variance explodes, and even a tiny violation of the exclusion
restriction produces large bias — weak-instrument bias points toward OLS. The
standard diagnostic is the **first-stage F-statistic** on the excluded
instruments; the well-known rule of thumb is

$$
F > 10
$$

for a single endogenous regressor. Below that, IV estimates are unreliable.

## Over-identification

When we have more instruments than endogenous regressors, the model is
**over-identified**, and we can partially test the exclusion restriction with
the **Sargan–Hansen J test**:

$$
J = N \cdot R^2 \sim \chi^2_{(m-k)},
$$

where the $R^2$ comes from regressing the 2SLS residuals on all instruments, and
$m-k$ is the number of surplus instruments. A large $J$ (small $p$-value)
signals that at least one instrument is invalid. Note the test assumes *some*
instruments are valid; it cannot rescue a model where all of them are suspect.

## What does IV actually estimate? The LATE

So far we have written $\beta_1$ as if it were a single number that applies to
everyone. But causal effects often **differ across individuals**, and once they
do, IV no longer recovers the average effect in the whole population. Imbens and
Angrist showed that, under one extra assumption, IV estimates a very specific
quantity: the **Local Average Treatment Effect (LATE)**.

The extra assumption is **monotonicity**: the instrument moves everyone in the
same direction (it can push people into treatment but never out). Under
relevance, exclusion, and monotonicity, the IV estimator converges to

$$
\hat{\beta}_1^{IV} \;\xrightarrow{p}\;
\mathbb{E}\!\left[\, Y_i(1) - Y_i(0) \;\middle|\; \text{compliers} \,\right],
$$

where $Y_i(1)$ and $Y_i(0)$ are the potential outcomes with and without
treatment. The population splits into three groups: **compliers**, who take the
treatment only when the instrument nudges them; **always-takers**; and
**never-takers**. IV identifies the effect for the **compliers alone** — the
people whose behaviour the instrument actually changes.

This has a sharp practical implication. Two valid instruments for the same
regressor can give different IV estimates, not because either is wrong, but
because each induces a *different* group of compliers. The quarter-of-birth
instrument, for instance, estimates the return to schooling for those whose
education was bound by compulsory-attendance laws — teenagers on the margin of
dropping out — not the return for a prospective PhD student. The LATE is causal
and well defined, but it is **local**: always ask *whose* effect the instrument
is estimating before generalising the result.

## Takeaways

Instrumental variables turn an untestable correlation problem into a research
design. **Relevance** (a strong first stage) and the **exclusion restriction**
(no direct effect on the outcome) are the twin pillars; 2SLS is the workhorse
estimator; weak-instrument F-tests and over-identification J-tests are the
guardrails. When a credible instrument exists, IV recovers causal effects that
OLS cannot — but the credibility lives in the economics of the instrument, not
in the algebra.
