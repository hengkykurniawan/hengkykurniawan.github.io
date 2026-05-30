---
layout: post
title:  "Difference-in-Differences: Theory, an Example, and Stata Code"
#categories: econometrics
---

Difference-in-differences (DID) is one of the most widely used research designs
in applied economics. It answers a deceptively simple question: *what was the
effect of a policy, program, or shock that hit one group but not another?* By
comparing how an outcome changes over time for a **treated** group against the
change for an untreated **control** group, DID strips away both fixed
differences between groups and common trends affecting everyone. This post
develops the theory carefully, works through a concrete numerical example, and
finishes with ready-to-run Stata code.

## The core idea

Suppose a state raises its minimum wage and we want to know what this did to
employment. The naive approaches both fail:

- **Before-vs-after in the treated state alone** confounds the policy with
  everything else that changed over time (a recession, seasonal hiring, national
  trends).
- **Treated-vs-control after the policy alone** confounds the policy with
  pre-existing differences between the two places (one state may simply have
  more restaurants, higher wages, a different industry mix).

DID combines the two comparisons to cancel both problems. We take the
*change* in the treated group and subtract the *change* in the control group.
The first difference removes anything fixed about each group; the second
difference removes anything common to both over time. What remains — the
**difference of the differences** — is the causal effect, under an assumption we
make precise below.

## Setup and notation

Consider two groups and two time periods. Let $D_i \in \{0,1\}$ indicate whether
unit $i$ belongs to the **treatment group** (the group that will eventually be
exposed to the policy), and let $T_t \in \{0,1\}$ indicate the **post-treatment
period**. The treatment is actually *in effect* only for treated units in the
post period, captured by the interaction $D_i \times T_t$.

Denote the expected outcome in each of the four cells by

$$
\mu_{dt} = \mathbb{E}[Y \mid D = d,\, T = t], \qquad d, t \in \{0, 1\}.
$$

So $\mu_{11}$ is the treated group after, $\mu_{10}$ the treated group before,
$\mu_{01}$ the control group after, and $\mu_{00}$ the control group before.

## The parallel trends assumption

DID rests on one central, untestable identifying assumption: **parallel
trends**. In words, *in the absence of treatment, the average outcome of the
treated group would have followed the same trend over time as the control
group.* Using potential-outcomes notation, let $Y_{it}(0)$ be the outcome unit
$i$ would have in period $t$ if untreated. Parallel trends requires

$$
\mathbb{E}[Y_{i1}(0) - Y_{i0}(0) \mid D = 1]
= \mathbb{E}[Y_{i1}(0) - Y_{i0}(0) \mid D = 0].
$$

The left side is the trend the treated group *would have experienced* without
the policy; the right side is the trend the control group actually experienced.
The assumption says these are equal. Note what it does **not** require: the two
groups can differ in *levels* by any fixed amount. DID allows the treated state
to always have higher employment — it only assumes the two groups would have
*moved together* absent the policy.

This assumption cannot be verified directly because the counterfactual trend for
the treated group is never observed. What we *can* do is look at **pre-treatment
periods**: if the groups tracked each other before the policy, that lends
credibility to the assumption (more on this with event studies below).

## The DID estimator from means

Under parallel trends, the **average treatment effect on the treated** (ATT) is
identified by the difference-in-differences of the four cell means:

$$
\delta^{DID} = (\mu_{11} - \mu_{10}) - (\mu_{01} - \mu_{00}).
$$

The first parenthesis is the change for the treated group; the second is the
change for the control group. To see why this recovers the causal effect, write
the treated post-period mean as its counterfactual plus the treatment effect,
$\mu_{11} = \mathbb{E}[Y_{i1}(0)\mid D{=}1] + \text{ATT}$. Substituting and
applying parallel trends, every counterfactual term cancels:

$$
\delta^{DID}
= \underbrace{\big(\mathbb{E}[Y_{i1}(0)\mid D{=}1] + \text{ATT} - \mu_{10}\big)}_{\text{treated change}}
- \underbrace{(\mu_{01} - \mu_{00})}_{\text{control change}}
= \text{ATT}.
$$

The estimator is just the sample analogue: replace each $\mu_{dt}$ with the
sample mean $\bar{Y}_{dt}$ in the corresponding cell.

$$
\hat{\delta}^{DID} = (\bar{Y}_{11} - \bar{Y}_{10}) - (\bar{Y}_{01} - \bar{Y}_{00}).
$$

## The regression formulation

In practice we almost never compute four means by hand. Instead we run the
equivalent regression, which generalises cleanly to covariates and clustered
standard errors:

$$
Y_{it} = \beta_0 + \beta_1 D_i + \beta_2 T_t + \delta\,(D_i \times T_t) + \varepsilon_{it}.
$$

Each coefficient maps directly onto a cell mean, and the interaction term is the
DID estimate:

- $\beta_0 = \mu_{00}$ — control group, before.
- $\beta_1 = \mu_{10} - \mu_{00}$ — fixed treated-vs-control gap.
- $\beta_2 = \mu_{01} - \mu_{00}$ — common time trend.
- $\delta = (\mu_{11} - \mu_{10}) - (\mu_{01} - \mu_{00})$ — **the treatment
  effect**.

The coefficient on the interaction, $\delta$, is exactly the
difference-in-differences. This is why DID is sometimes described as "the
coefficient on the interaction of treatment and post." Adding control variables
$\mathbf{X}_{it}$ is straightforward,

$$
Y_{it} = \beta_0 + \beta_1 D_i + \beta_2 T_t + \delta\,(D_i \times T_t)
+ \boldsymbol{\gamma}^{\top}\mathbf{X}_{it} + \varepsilon_{it},
$$

which can help make parallel trends more plausible by conditioning on observable
differences in how groups evolve.

## A worked numerical example

The classic application is **Card and Krueger's (1994)** study of the minimum
wage. In April 1992 New Jersey raised its minimum wage from \$4.25 to \$5.05 an
hour, while neighbouring Pennsylvania did not. Standard competitive theory
predicts employment should fall. Card and Krueger surveyed fast-food restaurants
in both states before and after the increase. Suppose we observe the following
average full-time-equivalent employment per restaurant (numbers illustrative,
in the spirit of their findings):

| Group | Before (Feb) | After (Nov) | Change |
|---|---|---|---|
| **New Jersey** (treated) | 20.4 | 21.0 | $+0.6$ |
| **Pennsylvania** (control) | 23.3 | 21.2 | $-2.1$ |

Plugging into the formula:

$$
\hat{\delta}^{DID} = (21.0 - 20.4) - (21.2 - 23.3) = (+0.6) - (-2.1) = +2.7.
$$

The DID estimate is **+2.7 employees per restaurant**. Strikingly, raising the
minimum wage is associated with a *small increase* (or at least no decrease) in
employment relative to the control. Notice how the design works: New Jersey's
own before-after change was only $+0.6$, which in isolation tells us little; but
Pennsylvania fell by $2.1$ over the same window — a regional downturn that, by
parallel trends, we attribute to New Jersey as well. Netting it out reveals that
New Jersey did *better* than its counterfactual by 2.7 jobs.

This result was influential precisely because it contradicted the textbook
prediction and launched a large literature. It also illustrates why the control
group matters so much: had we looked only at New Jersey, we would have concluded
"essentially no change." The control reveals that the true counterfactual was a
decline.

## Reading the example as a regression

It is worth connecting the table above to the regression coefficients, because
that is what Stata will report. With the four cell means
$\mu_{00} = 23.3$, $\mu_{10} = 20.4$, $\mu_{01} = 21.2$, $\mu_{11} = 21.0$, the
interaction regression $Y = \beta_0 + \beta_1 D + \beta_2 T + \delta (D\times T)$
returns

$$
\beta_0 = 23.3, \quad
\beta_1 = 20.4 - 23.3 = -2.9, \quad
\beta_2 = 21.2 - 23.3 = -2.1, \quad
\delta = +2.7.
$$

Each number tells a story. The intercept $\beta_0 = 23.3$ is simply
Pennsylvania's pre-period employment. The treatment-group dummy
$\beta_1 = -2.9$ says New Jersey restaurants were *smaller* on average to begin
with — exactly the kind of fixed level difference DID is allowed to ignore. The
post dummy $\beta_2 = -2.1$ captures the regional downturn common to both
states. And the interaction $\delta = +2.7$ is the causal estimate. Seeing the
decomposition makes clear that DID is not magic: it is bookkeeping that isolates
the one piece of variation — being treated *and* in the post period — that
identifies the effect.

## Threats to validity

Parallel trends is the headline assumption, but several other things can break a
DID design. Each has a standard diagnostic.

**Anticipation effects.** If units change behaviour *before* the policy formally
takes effect — restaurants cutting hiring in anticipation of a known future wage
hike — then the "before" period is already contaminated. The event study reveals
this as nonzero pre-treatment coefficients. A common fix is to treat the
announcement date, not the implementation date, as $t^*$, or to drop the
immediately-pre period.

**The Ashenfelter dip.** In job-training evaluations, people often enrol right
after a temporary drop in earnings. That transitory dip means the pre-period is
unusually low, mechanically inflating the apparent treatment effect. The lesson
generalises: selection into treatment that is correlated with *transitory*
shocks to the outcome violates parallel trends.

**Compositional changes.** DID assumes we are following a stable population. If
the *composition* of the treated or control group changes between periods — say,
high-wage restaurants exit New Jersey after the policy — then the before and
after means describe different populations, and the difference is not a causal
effect on a fixed group. With genuine panel data you can hold the sample fixed;
with repeated cross-sections you must argue composition is stable.

**Spillovers (SUTVA).** DID assumes the control group is genuinely untreated.
Pennsylvania restaurants near the New Jersey border might raise their own wages
to compete for workers, or unemployed New Jersey workers might cross state lines.
Any such spillover contaminates the control and biases the estimate toward zero.
Choosing controls that are *similar but insulated* from the treated group is part
of good design.

## Robustness and placebo tests

Because parallel trends cannot be proven, credible DID papers stack up
supporting evidence. The standard battery includes:

- **Pre-trend / event-study plots** — the single most persuasive check, showing
  flat leads before $t^*$.
- **Placebo (falsification) tests in time** — pretend the policy happened in a
  year when it did not, using only pre-period data; a significant "effect" is a
  red flag.
- **Placebo tests in groups** — assign treatment to a group that was never
  actually treated; you should find nothing.
- **Alternative control groups** — show the estimate is stable whether you use
  Pennsylvania, a synthetic control, or a pool of other states.
- **Sensitivity to functional form** — levels versus logs, with and without
  covariates.

When the headline estimate survives all of these, readers are far more willing to
believe the parallel-trends story.

## Inference: standard errors

Estimating $\hat{\delta}$ is only half the job; we need a standard error to know
whether $+2.7$ is statistically distinguishable from zero. Two issues dominate.

**Clustering.** Observations within the same group-period (or the same unit over
time) are correlated. Bertrand, Duflo, and Mullainathan (2004) showed that
ignoring **serial correlation** in DID dramatically *understates* standard
errors and *overstates* significance. The standard remedy is to **cluster the
standard errors at the level of treatment assignment** — typically the state (or
whatever unit the policy varies across):

$$
\widehat{\operatorname{Var}}_{\text{cluster}}(\hat{\delta})
= (\mathbf{X}^{\top}\mathbf{X})^{-1}
\left( \sum_{g} \mathbf{X}_g^{\top} \hat{\varepsilon}_g \hat{\varepsilon}_g^{\top} \mathbf{X}_g \right)
(\mathbf{X}^{\top}\mathbf{X})^{-1}.
$$

**Few clusters.** When the number of clusters is small (the canonical case has
just two states!), cluster-robust standard errors are themselves unreliable, and
researchers turn to wild-cluster bootstrap or randomization inference. With only
two clusters, credible inference essentially requires more groups — a reason
modern DID studies exploit many states and many time periods.

## Multiple periods, two-way fixed effects, and event studies

Real applications rarely have just two periods and two groups. The natural
generalisation is the **two-way fixed effects (TWFE)** model, with unit fixed
effects $\alpha_i$ and time fixed effects $\lambda_t$:

$$
Y_{it} = \alpha_i + \lambda_t + \delta\, \text{Treat}_{it} + \varepsilon_{it},
$$

where $\text{Treat}_{it}$ equals one when unit $i$ is actively treated in period
$t$. The unit effects absorb all time-invariant differences across units; the
time effects absorb shocks common to all units in a period. In the simple 2×2
case this reduces exactly to the interaction regression above.

To *probe* parallel trends and trace out dynamics, we estimate an **event-study
(dynamic) specification**. Replacing the single treatment dummy with a full set
of leads and lags relative to the treatment date $t^*$:

$$
Y_{it} = \alpha_i + \lambda_t + \sum_{k \neq -1} \delta_k \, \mathbf{1}\{t - t^* = k\} + \varepsilon_{it}.
$$

Here each $\delta_k$ measures the treatment effect $k$ periods after adoption,
with the period just before treatment ($k = -1$) omitted as the baseline. Two
things to read off the estimated $\delta_k$:

- **Pre-trends.** The coefficients for $k < -1$ should be near zero. If outcomes
  were already diverging *before* the policy, parallel trends is suspect.
- **Dynamics.** The coefficients for $k \ge 0$ show how the effect builds or
  fades over time.

## A caution about staggered adoption

Much recent econometric work (around 2018–2021) has shown that when units adopt
treatment at **different times** ("staggered" rollout), the simple TWFE estimator
can be badly biased. The reason, formalised by **Goodman-Bacon (2021)**, is that
TWFE is a weighted average of *all possible* 2×2 comparisons — including
"forbidden" comparisons that use **already-treated units as controls** for
later-treated ones. When treatment effects change over time, these comparisons
get negative weights and the headline coefficient can even flip sign.

Modern estimators fix this by being explicit about which clean comparisons to
use. The leading alternatives include **Callaway and Sant'Anna (2021)**, which
estimates group-time average treatment effects $ATT(g,t)$ and aggregates them
with sensible weights; **Sun and Abraham (2021)** for event studies; and **de
Chaisemartin and D'Haultfœuille (2020)**. The practical advice: with a single
treatment date, plain TWFE is fine; with staggered timing and dynamic effects,
use one of these robust estimators.

## Triple differences (DDD)

Sometimes even the control group is suspect because *something else* changed in
the treated state at the same time as the policy. The **triple-differences**
(DDD) design adds a third dimension — typically a within-state group that the
policy should *not* affect — to net out such confounds. The estimator
differences the DID of the affected subgroup against the DID of the unaffected
subgroup:

$$
\delta^{DDD} = \Big[(\bar{Y}^{A}_{11} - \bar{Y}^{A}_{10}) - (\bar{Y}^{A}_{01} - \bar{Y}^{A}_{00})\Big]
- \Big[(\bar{Y}^{B}_{11} - \bar{Y}^{B}_{10}) - (\bar{Y}^{B}_{01} - \bar{Y}^{B}_{00})\Big],
$$

where superscript $A$ is the group the policy targets and $B$ is a comparison
group in the same state and time. In regression form this is a model with all
three two-way interactions plus the **triple interaction**, whose coefficient is
$\delta^{DDD}$. The classic application is Gruber's (1994) study of
state-mandated maternity benefits, which used men and older women within the same
states as the unaffected comparison group to difference out state-by-time shocks.

## Stata example

Below is a complete, self-contained Stata workflow. The first block shows the
canonical DID regression, the rest covers clustered inference, panel
fixed-effects, an event study, and a staggered-design estimator.

```stata
* ---------------------------------------------------------------
* 1. A simple 2x2 difference-in-differences
* ---------------------------------------------------------------
* Variables: treat (1 = New Jersey), post (1 = after policy),
*            emp (employment outcome), id (restaurant), state

* The DID estimate is the coefficient on the treat#post interaction.
* Using factor-variable notation (i.treat##i.post) is preferred:
regress emp i.treat##i.post, vce(cluster state)

* The interaction term 1.treat#1.post is your DID estimate (delta).

* ---------------------------------------------------------------
* 2. Equivalent, with a manually created interaction
* ---------------------------------------------------------------
gen did = treat * post
regress emp treat post did, vce(cluster state)
* coefficient on `did' = the difference-in-differences

* ---------------------------------------------------------------
* 3. Panel DID with two-way fixed effects (many units/periods)
* ---------------------------------------------------------------
xtset id year                 // declare the panel structure

* `treated' = 1 when a unit is actively under treatment in that year
xtreg emp i.treated i.year, fe vce(cluster state)
* the coefficient on 1.treated is the DID/ATT

* ---------------------------------------------------------------
* 4. Event-study (dynamic) specification to check pre-trends
* ---------------------------------------------------------------
* rel = year - adoption_year ; baseline = -1
gen rel = year - adopt_year
* ib(-1) omits the period just before treatment as the reference
reghdfe emp ib(-1).rel, absorb(id year) vce(cluster state)
coefplot, vertical yline(0) xline(0)   // visualise leads & lags

* ---------------------------------------------------------------
* 5. Staggered adoption: a robust modern estimator
* ---------------------------------------------------------------
* Callaway & Sant'Anna (install once):  ssc install csdid
* `first_treat' = the year each unit was first treated (0 if never)
csdid emp, ivar(id) time(year) gvar(first_treat) method(dripw)
estat event                  // aggregated event-study effects
csdid_plot                   // plot the dynamic ATTs
```

A few practical notes on the Stata code. The factor-variable syntax
`i.treat##i.post` automatically creates both main effects and the interaction,
and `vce(cluster state)` delivers the cluster-robust standard errors discussed
above. For panels, `reghdfe` (install with `ssc install reghdfe`) is far faster
than `xtreg` when you have many fixed effects, and it handles multi-way
clustering. For staggered designs, `csdid` implements Callaway and Sant'Anna
directly and sidesteps the negative-weighting problem of plain TWFE.

## How DID relates to other designs

It helps to place DID among its neighbours. Compared with a simple
**before-after** comparison, DID adds a control group to absorb common shocks.
Compared with a pure **cross-sectional** treated-vs-control comparison, it adds a
time dimension to absorb fixed group differences. In that sense DID *is* a
special case of **panel fixed-effects** estimation: the two-way fixed effects
model with unit and time dummies reduces to the DID interaction in the canonical
2×2 case, which is why the two literatures share so much machinery.

DID also sits close to two other modern designs. The **synthetic control method**
can be seen as a data-driven way of constructing the control group: instead of
picking one comparison state, it builds a weighted combination of many untreated
units that best matches the treated unit's pre-treatment path — useful precisely
when no single control satisfies parallel trends. **Event-study designs** are
simply DID with a full set of time-relative coefficients, as we saw above. The
common thread across all of them is the same counterfactual logic: estimate what
*would* have happened, and attribute the gap to the treatment.

## Checklist and takeaways

Difference-in-differences turns a policy change into a natural experiment by
double-differencing away fixed group effects and common time shocks. To use it
credibly:

1. **Identify a clean control** that plausibly shares the treated group's
   trend.
2. **Defend parallel trends** — argue it institutionally and show flat
   pre-trends in an event study.
3. **Run the interaction regression** and read $\delta$ off the
   treatment-by-post term.
4. **Cluster your standard errors** at the level of treatment, and worry about
   inference when clusters are few.
5. **Beware staggered timing** — switch to Callaway–Sant'Anna or similar when
   units adopt at different dates and effects evolve.

Done well, DID is among the most transparent and persuasive tools in the
empirical economist's kit: the entire identifying assumption can be summarised in
a single picture of two trend lines moving in parallel until one of them is
nudged by a policy.
