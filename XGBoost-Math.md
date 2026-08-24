# XGBoost: The Full Math, Built From Scratch

Every formula derived, every variable defined next to what it *means*, and a worked numerical example that shows the machine running.

---

## 0. The core idea in one line

Train a model by **minimizing (total error) + (penalty for complexity)**, and build it up one tree at a time, where each new tree corrects the leftover error of the trees so far.

Every decision in the model is the same shape: **a cost is weighed against a benefit, and the action happens only if the benefit wins.** Splitting a leaf costs a toll ($\gamma$) and buys a loss reduction; the split survives iff the reduction beats the toll. Keep that "cost weighed against benefit" template in mind — every boxed formula below is an instance of it.

---

## 1. Variables — the complete dictionary

| Symbol | Name | Math meaning | Intuition |
|---|---|---|---|
| $x_i$ | feature vector | input for example $i$ | the row of data (qualifying gap, rain flag, etc.) |
| $y_i$ | label | true target for example $i$ | what actually happened |
| $\hat{y}_i$ | prediction | model's current output for $i$ | current best guess |
| $\hat{y}_i^{(t)}$ | prediction at round $t$ | guess after $t$ trees | guess improves as rounds stack |
| $\ell(y_i, \hat{y}_i)$ | **loss** | per-example wrongness | "how wrong is this one prediction" |
| $\mathcal{L}$ | **objective** | total quantity minimized | loss over all data $+$ complexity penalty |
| $f_t$ | a tree | the $t$-th tree, a function $x \mapsto$ value | one round's correction |
| $T$ | leaf count | number of leaves in a tree | how many distinct outputs the tree can give |
| $j$ | leaf index | which leaf, $j = 1 \dots T$ | a specific bucket of examples |
| $w_j$ | leaf weight | the value leaf $j$ outputs | the correction applied to everything in that bucket |
| $I_j$ | instance set | $\{i : \text{example } i \text{ lands in leaf } j\}$ | which examples fall into that bucket |
| $g_i$ | gradient | $\partial \ell / \partial \hat{y}_i$ | slope of loss — which way / how hard to push |
| $h_i$ | hessian | $\partial^2 \ell / \partial \hat{y}_i^2$ | curvature — how confident the push should be |
| $G_j$ | leaf gradient sum | $\sum_{i \in I_j} g_i$ | total push demanded by that bucket |
| $H_j$ | leaf hessian sum | $\sum_{i \in I_j} h_i$ | total curvature in that bucket |
| $\lambda$ | L2 penalty | weight-shrinkage knob | how loud each leaf is allowed to speak |
| $\gamma$ | complexity penalty | per-leaf toll | how many leaves are allowed to exist |
| $\eta$ | learning rate | scale on each tree's output | how big a step each round takes |

> **Naming trap:** $\ell$ (little ell) is the **loss** on one prediction. $\mathcal{L}$ (script L) is the **objective** — the whole thing being minimized. They are not the same symbol and not the same object.

---

## 2. The objective

$$\mathcal{L} = \underbrace{\sum_{i=1}^{n} \ell(y_i, \hat{y}_i)}_{\text{benefit side: fit the data}} \;+\; \underbrace{\Omega(f)}_{\text{cost side: pay for complexity}}$$

**The relationship in words:** this is the same "cost weighed against benefit" template as the split, one level up. Fitting the data lowers the loss term (the benefit of complexity); every bit of complexity raises $\Omega$ (its cost). The model settles where one more unit of fit is no longer worth the complexity it costs.

- **Loss term** $\sum_i \ell$ — pushes the model to fit the data.
- **Regularization term** $\Omega(f)$ — pushes back, punishing complexity.

**Why regularization exists:** a tree can memorize. Enough leaves and training error hits zero — one leaf per data point. That's memorization, not learning. If you minimize *only* $\sum_i \ell$, the optimizer's best move is always "add more leaves." $\Omega$ is the brake: adding a leaf lowers the loss term but raises the penalty term, so a leaf is only added when it earns its keep.

Complexity has two natural meanings, so $\Omega$ has two terms:

$$\Omega(f) = \gamma T + \frac{1}{2}\lambda \sum_{j=1}^{T} w_j^2$$

| Term | Charges for | Bigger knob → |
|---|---|---|
| $\gamma T$ | **number** of leaves | fewer splits, smaller tree |
| $\tfrac12 \lambda \sum_j w_j^2$ | **magnitude** of each leaf output | weights shrink toward 0 |

Neither term looks at the data — only at the *shape* of the tree. That's what makes it regularization and not loss.

---

## 3. Additive training — one tree at a time

XGBoost doesn't build the whole model at once. At round $t$ it freezes everything so far and adds one tree:

$$\hat{y}_i^{(t)} = \hat{y}_i^{(t-1)} + f_t(x_i)$$

So the objective at round $t$ is:

$$\mathcal{L}^{(t)} = \sum_{i=1}^{n} \ell\!\left(y_i,\; \hat{y}_i^{(t-1)} + f_t(x_i)\right) + \Omega(f_t)$$

Because $\hat{y}_i^{(t-1)}$ is already decided, it is a **known constant**. The only unknown is the new tree $f_t$. This is the key that unlocks the whole derivation: we're optimizing one tree with everything else fixed.

---

## 4. Second-order Taylor expansion — where $g_i$ and $h_i$ come from

$\ell$ can be messy (log-loss, etc.), so approximate it. Treat $f_t(x_i)$ as a small step added to the known point $\hat{y}_i^{(t-1)}$, and Taylor-expand to 2nd order:

$$\ell\!\left(y_i,\; \hat{y}_i^{(t-1)} + f_t(x_i)\right) \approx \underbrace{\ell\!\left(y_i, \hat{y}_i^{(t-1)}\right)}_{\text{constant}} + g_i\, f_t(x_i) + \tfrac12 h_i\, f_t(x_i)^2$$

with

$$g_i = \frac{\partial\, \ell(y_i, \hat{y}_i^{(t-1)})}{\partial\, \hat{y}_i^{(t-1)}}, \qquad h_i = \frac{\partial^2\, \ell(y_i, \hat{y}_i^{(t-1)})}{\partial\, (\hat{y}_i^{(t-1)})^2}$$

The constant term has no $f_t$, so it can't affect the minimization — drop it:

$$\tilde{\mathcal{L}}^{(t)} = \sum_{i=1}^{n} \left[ g_i\, f_t(x_i) + \tfrac12 h_i\, f_t(x_i)^2 \right] + \Omega(f_t)$$

> **Meaning:** near the current prediction, any loss looks like a parabola. $g_i$ is its slope, $h_i$ its curvature. From here on, the tree only sees these two numbers per example — it has forgotten the raw $y_i$.

---

## 5. Rewrite by leaf — how a *tree* becomes an *equation*

This is the conceptual leap. A tree is a **piecewise-constant** function: every example in the same leaf gets the **same output**. If $x_i$ lands in leaf $j$, then

$$f_t(x_i) = w_j$$

So group the sum-over-examples into a sum-over-leaves. All examples in leaf $j$ share $w_j$, so pull it out:

$$\tilde{\mathcal{L}}^{(t)} = \sum_{j=1}^{T} \left[ \Big(\sum_{i \in I_j} g_i\Big) w_j + \tfrac12 \Big(\sum_{i \in I_j} h_i\Big) w_j^2 \right] + \gamma T + \tfrac12 \lambda \sum_{j=1}^{T} w_j^2$$

**What $\sum_{j=1}^{T}$ means:** add up over the leaves. $j$ is which leaf, $T$ is how many. When a split happens, $T$ grows by 1 and the sum gains one more term. That is *literally* how splits enter the equations.

Define the per-leaf sums:

$$G_j = \sum_{i \in I_j} g_i, \qquad H_j = \sum_{i \in I_j} h_i$$

The $\tfrac12 \lambda w_j^2$ term is also summed over $j$, so fold it into the bracket (it merges with $\tfrac12 H_j w_j^2$):

$$\tilde{\mathcal{L}}^{(t)} = \sum_{j=1}^{T} \left[ G_j\, w_j + \tfrac12 (H_j + \lambda) w_j^2 \right] + \gamma T$$

For one leaf (dropping $\gamma T$, constant in $w_j$):

$$\boxed{\;\text{Obj}(w_j) = G_j\, w_j + \tfrac12 (H_j + \lambda)\, w_j^2\;}$$

> **Meaning:** each leaf is a simple parabola in its own output $w_j$. $\lambda$ just adds to the curvature. The abstract "leaf" is now a variable; the abstract "tree" is now a sum of parabolas.

---

## 6. Optimal leaf weight $w_j^*$

$\text{Obj}(w_j)$ is an upward parabola (coefficient $H_j + \lambda > 0$), so it has one minimum. Differentiate, set to zero:

$$\frac{\partial\, \text{Obj}}{\partial\, w_j} = G_j + (H_j + \lambda)\, w_j = 0$$

$$\boxed{\; w_j^* = -\frac{G_j}{H_j + \lambda} \;}$$

**How $\lambda$ shrinks weights:** it sits in the denominator.

| $\lambda$ | effect on $w_j^*$ |
|---|---|
| $0$ | $w^* = -G_j / H_j$ — full step, largest magnitude |
| large | denominator inflates → $w^*$ shrinks toward 0 |
| $\to \infty$ | $w^* \to 0$ |

$\lambda$ never flips the **sign** (that's set by $-G_j$, the loss-reducing direction) — it only **damps the size**.

---

## 7. Best score of a fixed tree structure

Plug $w_j^*$ back into the leaf objective. For any parabola $a w + \tfrac12 b w^2$ with minimizer $w^* = -a/b$, the minimum value is $-\tfrac{a^2}{2b}$:

$$\text{Obj}(w^*) = a\left(-\frac{a}{b}\right) + \tfrac12 b \left(\frac{a}{b}\right)^2 = -\frac{a^2}{b} + \frac{a^2}{2b} = -\frac{1}{2}\frac{a^2}{b}$$

With $a = G_j$, $b = H_j + \lambda$, each leaf's best score is $-\tfrac12 \dfrac{G_j^2}{H_j + \lambda}$. Sum over leaves and restore $\gamma T$:

$$\boxed{\; \tilde{\mathcal{L}}^{(t)}_{\text{best}} = -\frac{1}{2} \sum_{j=1}^{T} \frac{G_j^2}{H_j + \lambda} \;+\; \gamma T \;}$$

Lower is better. The first term (negative) is how much loss the tree removes; $\gamma T$ (positive) charges for the leaves.

---

## 8. The split Gain — derived from "after − before"

The single expression $-\tfrac12 \dfrac{G^2}{H+\lambda}$ is the DNA of the Gain formula. Take one leaf holding instances $I$, split into left $I_L$ and right $I_R$:

$$G_L = \sum_{i \in I_L} g_i,\; G_R = \sum_{i \in I_R} g_i,\; G = G_L + G_R$$
$$H_L = \sum_{i \in I_L} h_i,\; H_R = \sum_{i \in I_R} h_i,\; H = H_L + H_R$$

**Before** (1 leaf, 1 toll):

$$\text{Obj}_{\text{before}} = -\frac12 \frac{G^2}{H+\lambda} + \gamma$$

**After** (2 leaves, 2 tolls):

$$\text{Obj}_{\text{after}} = -\frac12 \frac{G_L^2}{H_L+\lambda} - \frac12 \frac{G_R^2}{H_R+\lambda} + 2\gamma$$

Change from splitting (after − before):

$$\Delta = -\frac12\!\left[\frac{G_L^2}{H_L+\lambda} + \frac{G_R^2}{H_R+\lambda} - \frac{G^2}{H+\lambda}\right] + \underbrace{(2\gamma - \gamma)}_{=\,\gamma}$$

Splitting adds exactly one leaf, so $T \to T+1$ and the penalty rises by one $\gamma$ — that lone $+\gamma$.

$\Delta < 0$ means the split helps (objective drops). Testing "$\Delta < 0$" is awkward, so define **Gain $= -\Delta$** to get the friendlier "$\text{Gain} > 0$". Negate every term:

$$\boxed{\; \text{Gain} = \underbrace{\frac12\!\left[\frac{G_L^2}{H_L+\lambda} + \frac{G_R^2}{H_R+\lambda} - \frac{(G_L+G_R)^2}{H_L+H_R+\lambda}\right]}_{\text{benefit: loss removed by splitting}} \;-\; \underbrace{\gamma}_{\text{toll: cost of one more leaf}} \;}$$

**The relationship in words:** the toll — the penalty for adding a leaf — is weighed against the benefit that leaf provides, which we calculate as the loss reduction in the bracket. The split is kept iff

$$\underbrace{\tfrac12\!\left[\dots\right]}_{\text{benefit (from the data)}} \;>\; \underbrace{\gamma}_{\text{toll (a knob you set)}} \quad\Longleftrightarrow\quad \text{Gain} > 0.$$

Reading each piece:

- $\dfrac{G_L^2}{H_L+\lambda},\; \dfrac{G_R^2}{H_R+\lambda}$ — the two new leaves' scores (the benefit of having two buckets).
- $-\dfrac{(G_L+G_R)^2}{H_L+H_R+\lambda}$ — subtract the old single leaf's score, because you're *replacing* it, not adding to it. The bracket is the *net* improvement.
- $-\gamma$ — the flipped $+\gamma$: one net new leaf, one new toll.

The benefit is **measured from the data** (the bracket is built entirely from $G$ and $H$, i.e. sums of the examples' gradients and hessians), while the toll is **a value you set**. You never tune the benefit; you tune the *bar* it has to clear. So the thing countering the $\gamma$ penalty is the loss reduction *from the data*, not a parameter — a split separating genuinely different examples yields a big bracket, a split on noise yields a tiny one. This is also the entire mechanism by which the model "shrinks": there is no separate shrinking force, only splits *failing to clear the bar* and therefore never being added. The formula isn't designed — it's *forced*: "value of two optimized leaves − value of one optimized leaf − cost of the extra leaf," sign-flipped so bigger = better.

Concretely, with the numbers from §9 the bracket evaluates to $\tfrac12[12+12-0] = 12$ — that $12$ is the data-driven benefit, and only the bar $\gamma$ moves:

| Bar you set | Benefit (data) | Gain | Outcome |
|---|---|---|---|
| $\gamma = 0$ | $12$ | $+12$ | split accepted |
| $\gamma = 15$ | $12$ | $-3$ | split rejected — leaf never added |

---

## 9. Worked example — the machine running on real numbers

**Loss:** squared error $\ell = \tfrac12 (y - \hat{y})^2$, giving clean derivatives:

$$g_i = \hat{y}_i - y_i, \qquad h_i = 1$$

**Data** ($\lambda = 1$, $\gamma = 0$, learning rate $\eta = 1$):

| $i$ | $x$ | $y$ |
|---|---|---|
| 1 | 1 | 1 |
| 2 | 2 | 3 |
| 3 | 6 | 7 |
| 4 | 7 | 9 |

### Step 1 — first prediction
The constant minimizing total loss is the mean:

$$\hat{y}^{(0)} = \frac{1+3+7+9}{4} = 5 \quad \text{for all examples.}$$

### Step 2 — gradients and hessians at $\hat{y} = 5$

$$g_i = 5 - y_i:\quad g_1 = 4,\; g_2 = 2,\; g_3 = -2,\; g_4 = -4, \qquad h_i = 1.$$

The next tree now sees only these numbers.

### Step 3 — root leaf (all 4 examples, $T=1$)

$$G = 4+2-2-4 = 0, \qquad H = 4$$
$$\text{Obj}(w) = 0 \cdot w + \tfrac12 (4+1) w^2 = 2.5\, w^2 \;\Rightarrow\; w^* = -\tfrac{0}{5} = 0,\quad \text{score} = 0.$$

The mean already nailed the average — no correction helps *without splitting*.

### Step 4 — try split $x < 4$: $\{1,2\}$ left, $\{3,4\}$ right ($T: 1 \to 2$)

$$G_L = 6,\; H_L = 2 \qquad G_R = -6,\; H_R = 2$$

One parabola becomes two independent parabolas:

$$\text{Obj}(w_L) = 6 w_L + \tfrac12(3) w_L^2, \qquad \text{Obj}(w_R) = -6 w_R + \tfrac12(3) w_R^2$$
$$w_L^* = -\tfrac{6}{3} = -2, \qquad w_R^* = -\tfrac{-6}{3} = +2$$

### Step 5 — Gain

$$\text{Gain} = \tfrac12\!\left[\frac{6^2}{3} + \frac{(-6)^2}{3} - \frac{0^2}{5}\right] - 0 = \tfrac12[12 + 12 - 0] = 12 > 0$$

Split accepted. (Had $\gamma = 15$: Gain $= 12 - 15 = -3 < 0$, rejected — the tree stays a stump. That's $\gamma$ as the toll, in numbers.)

### Step 6 — update predictions

$$\hat{y}^{(1)}_i = \hat{y}^{(0)}_i + \eta\, f_1(x_i)$$

| $i$ | old $\hat{y}$ | leaf output | new $\hat{y}$ | true $y$ |
|---|---|---|---|---|
| 1 | 5 | $-2$ | 3 | 1 |
| 2 | 5 | $-2$ | 3 | 3 |
| 3 | 5 | $+2$ | 7 | 7 |
| 4 | 5 | $+2$ | 7 | 9 |

Round 2 recomputes fresh $g_i = \hat{y}_i - y_i = \{2, 0, 0, -2\}$ and builds another tree to chip at the residual on examples 1 and 4. Repeat.

---

## 10. The whole loop, no intuition

1. Start with a constant prediction (minimizes loss with zero trees).
2. Compute $g_i, h_i$ at the current prediction.
3. Each candidate leaf is a parabola $G_j w_j + \tfrac12(H_j + \lambda) w_j^2$.
4. Optimal leaf value: $w_j^* = -G_j / (H_j + \lambda)$.
5. A split replaces one parabola with two; accept it iff $\text{Gain} > 0$.
6. Add $\eta \times$ (leaf outputs) to the prediction.
7. Recompute $g_i, h_i$; go to 3.

---

## 11. Two knobs, side by side

| Knob | Lives in | Controls | Bigger → |
|---|---|---|---|
| $\lambda$ | denominator of $w_j^* = -\dfrac{G_j}{H_j + \lambda}$ | magnitude of each leaf output | weights shrink toward 0 |
| $\gamma$ | the $-\gamma$ in Gain | number of leaves (splits allowed) | fewer splits, smaller tree |

**$\lambda$ = how loud each leaf may speak; $\gamma$ = how many leaves may exist.** That's why $\Omega$ has two separate terms.
