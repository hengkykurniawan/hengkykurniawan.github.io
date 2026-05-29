---
layout: post
title:  "Convexity of a Graph: Definitions and Proofs"
#categories: mathematics
---

Convexity is one of the most useful structural properties a function can have.
In economics it guarantees that a cost curve bends the right way, that
indifference sets are well behaved, and — most importantly — that a local
optimum is a global one. This post builds convexity up carefully from its
definition and proves the key results, including the first- and second-order
characterisations and why convexity makes optimisation easy.

## Convex sets

Before talking about the graph of a function, we need convex sets. A set
$C \subseteq \mathbb{R}^n$ is **convex** if, for any two points in it, the entire
line segment joining them also lies in the set:

$$
x, y \in C \;\Longrightarrow\; \lambda x + (1-\lambda) y \in C
\quad \text{for all } \lambda \in [0, 1].
$$

The point $\lambda x + (1-\lambda)y$ is a **convex combination** of $x$ and $y$;
as $\lambda$ runs from $0$ to $1$ it traces the chord between them. Intuitively,
a convex set has no dents or holes.

## Convex functions and the epigraph

A function $f : C \to \mathbb{R}$ defined on a convex set $C$ is **convex** if

$$
f\big(\lambda x + (1-\lambda) y\big) \;\le\; \lambda f(x) + (1-\lambda) f(y)
\qquad \forall\, x, y \in C,\; \lambda \in [0,1].
$$

The geometric reading is the heart of the matter: the **graph of $f$ lies on or
below the chord** connecting any two of its points. If the inequality is strict
for $x \neq y$ and $\lambda \in (0,1)$, the function is **strictly convex**.

There is an elegant link between the two notions of convexity above. Define the
**epigraph** of $f$ — everything on or above the graph:

$$
\operatorname{epi} f = \{ (x, t) \in C \times \mathbb{R} : t \ge f(x) \}.
$$

**Claim.** $f$ is convex *if and only if* $\operatorname{epi} f$ is a convex set.

*Proof.* Suppose $f$ is convex and take two points
$(x, s), (y, t) \in \operatorname{epi} f$, so $s \ge f(x)$ and $t \ge f(y)$. For
$\lambda \in [0,1]$, convexity of $f$ gives

$$
f(\lambda x + (1-\lambda) y) \le \lambda f(x) + (1-\lambda) f(y)
\le \lambda s + (1-\lambda) t.
$$

Hence $\big(\lambda x + (1-\lambda)y,\; \lambda s + (1-\lambda)t\big) \in
\operatorname{epi} f$, so the epigraph is convex. Conversely, if
$\operatorname{epi} f$ is convex, apply the definition to the boundary points
$(x, f(x))$ and $(y, f(y))$; the resulting inclusion is exactly the convexity
inequality for $f$. $\blacksquare$

This equivalence is why "convexity of a graph" and "convexity of a function"
mean the same thing.

## First-order characterisation

For differentiable functions there is a cleaner test that says the graph never
falls below any of its tangent lines.

**Theorem.** Let $f$ be differentiable on a convex set $C$. Then $f$ is convex
if and only if

$$
f(y) \;\ge\; f(x) + \nabla f(x)^{\top}(y - x)
\qquad \forall\, x, y \in C.
$$

*Proof ($\Rightarrow$).* Assume $f$ is convex. For $\lambda \in (0,1]$,

$$
f\big(x + \lambda(y - x)\big) = f(\lambda y + (1-\lambda)x)
\le \lambda f(y) + (1-\lambda) f(x).
$$

Rearranging,

$$
\frac{f(x + \lambda(y - x)) - f(x)}{\lambda} \le f(y) - f(x).
$$

Letting $\lambda \to 0^+$, the left side converges to the directional derivative
$\nabla f(x)^{\top}(y-x)$, which yields the tangent inequality.

*Proof ($\Leftarrow$).* Suppose the tangent inequality holds everywhere. Fix
$x, y$ and let $z = \lambda x + (1-\lambda) y$. Applying the inequality from $z$
to $x$ and from $z$ to $y$:

$$
f(x) \ge f(z) + \nabla f(z)^{\top}(x - z), \qquad
f(y) \ge f(z) + \nabla f(z)^{\top}(y - z).
$$

Multiply the first by $\lambda$, the second by $1-\lambda$, and add. The
gradient terms cancel because $\lambda(x - z) + (1-\lambda)(y - z) = 0$, leaving

$$
\lambda f(x) + (1-\lambda) f(y) \ge f(z) = f(\lambda x + (1-\lambda)y),
$$

which is convexity. $\blacksquare$

## Second-order characterisation

When $f$ is twice differentiable, convexity reduces to a condition on curvature.

**Theorem.** A twice-differentiable $f$ on an open convex set is convex if and
only if its **Hessian is positive semidefinite** everywhere:

$$
\nabla^2 f(x) \succeq 0 \quad \Longleftrightarrow \quad
v^{\top} \nabla^2 f(x)\, v \ge 0 \;\; \forall v \in \mathbb{R}^n.
$$

*Proof sketch.* Restrict $f$ to the line $g(t) = f(x + t v)$. By the chain rule
$g''(t) = v^{\top}\nabla^2 f(x + tv)\, v$. A one-dimensional function is convex
iff its second derivative is nonnegative, and $f$ is convex iff every such
restriction $g$ is convex. Combining the two statements gives the Hessian
condition. $\blacksquare$

In one dimension this is the familiar rule $f''(x) \ge 0$. For example,
$f(x) = x^2$ has $f''(x) = 2 > 0$, so it is (strictly) convex; $f(x) = e^x$ has
$f''(x) = e^x > 0$, convex everywhere; and $f(x) = \log x$ has
$f''(x) = -1/x^2 < 0$, so it is concave, not convex.

## Why convexity matters: global optimality

The payoff of all this structure is a clean optimisation result.

**Theorem.** If $f$ is convex, then any local minimum is a global minimum.

*Proof.* Let $x^{*}$ be a local minimum and suppose, for contradiction, that some
$y$ has $f(y) < f(x^{*})$. By convexity, for $\lambda \in (0,1)$,

$$
f\big(\lambda y + (1-\lambda) x^{*}\big)
\le \lambda f(y) + (1-\lambda) f(x^{*}) < f(x^{*}).
$$

As $\lambda \to 0^+$, the point $\lambda y + (1-\lambda)x^{*}$ stays arbitrarily
close to $x^{*}$ yet has a strictly smaller value — contradicting the assumption
that $x^{*}$ is a local minimum. Hence no such $y$ exists. $\blacksquare$

For a differentiable convex function this means the first-order condition
$\nabla f(x^{*}) = 0$ is not just necessary but **sufficient** for a global
minimum — the reason convex programs are so tractable.

## Takeaways

Convexity ties together geometry and analysis: the chord definition, the
convexity of the epigraph, the tangent-line (first-order) inequality, and the
positive-semidefinite Hessian (second-order) condition are all equivalent ways
of saying the same thing — the graph curves upward. The reward is decisive: for
convex functions, local optimality implies global optimality, which is why
convexity sits at the foundation of optimisation and economic theory alike.
