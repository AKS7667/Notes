# A Practical Guide to ML Models

This guide walks through the major families of machine-learning models, from linear regression up through deep networks. For each model you get a formal definition, an intuitive picture, the math that defines it, the scikit-learn call (or the next-best library when scikit-learn doesn't ship it), and the parameters and hyperparameters that actually matter in practice. Throughout, I'll flag when each model is the right tool for the job. A "parameter" here is something the model *learns* from data; a "hyperparameter" is something *you* set before training. Scikit-learn's API blurs this — both are passed to the constructor — so I'll group both under the constructor arguments and label them by role.

<br><br>

# INDEX  
---

**[1. Linear Models](#1-linear-models)**  
*[1.1 Linear Regression](#11-linear-regression-ordinary-least-squares)*  
*[1.2 Ridge Regression (L2 - Regularized OLS)](#12-ridge-regression-l2-regularized-ols)*  
*[1.3 Lasso (L1-regularized OLS)](#13-lasso-l1-regularized-ols)*  
*[1.4 Elastic Net](#14-elastic-net)*  
*[1.5 Logistic Regression](#15-logistic-regression)*  

**[2. Distance Based Models ](#2-distance-based-models)**  
*[2.1 K Nearest Neighbor (KNN) ](#21-k-nearest-neighbors-knn)*
*[]()*
*[]()*
*[]()*


---

## 1. Linear Models

Linear models assume the target is a linear combination of features (possibly after a link function like the sigmoid). They're fast, interpretable, and a strong baseline. If a linear model already does well, more complex models often won't help much.

### 1.1 Linear Regression (Ordinary Least Squares)

**Formal definition.** Given features $X \in \mathbb{R}^{n \times p}$ and target $y \in \mathbb{R}^n$, fit $\hat{y} = X\beta + \beta_0$ by minimizing the sum of squared residuals.

**Intuition.** Draw the line (or hyperplane) that gets as close as possible to every point, where "close" is measured by squared vertical distance.

**Math.**
$$\hat{\beta} = \arg\min_{\beta} \|y - X\beta\|_2^2 \quad\Longrightarrow\quad \hat{\beta} = (X^\top X)^{-1} X^\top y$$

The squared loss has a closed-form solution when $X^\top X$ is invertible. Otherwise sklearn falls back to the pseudo-inverse.

**scikit-learn.** `from sklearn.linear_model import LinearRegression`.

**Constructor arguments.**
- `fit_intercept` (bool, default `True`): whether to learn $\beta_0$. Set to `False` only if your data is already centered.
- `copy_X` (bool): whether to copy `X` before fitting. Leave `True` unless memory is tight.
- `n_jobs` (int): parallelism for the solve.
- `positive` (bool): constrains coefficients to be non-negative — useful when negative effects are physically impossible (e.g. counts, masses).

There are no hyperparameters in the strict sense — no regularization, no knobs — which is why this is your baseline.

**Use it when** you want a fast, interpretable baseline for a continuous target with roughly linear effects, or you genuinely care about coefficient values for inference.

---

### 1.2 Ridge Regression (L2-regularized OLS)

**Formal definition.** OLS with an L2 penalty on the coefficients.

**Intuition.** OLS, but it dislikes large coefficients, so it shrinks them toward zero. This trades a bit of bias for a lot less variance — especially useful when features are correlated.

**Math.**
$$\hat{\beta} = \arg\min_{\beta} \|y - X\beta\|_2^2 + \alpha \|\beta\|_2^2$$

Closed form: $\hat{\beta} = (X^\top X + \alpha I)^{-1} X^\top y$. The $\alpha I$ term is what makes the matrix always invertible — that's where "ridge" gets its name.

**scikit-learn.** `from sklearn.linear_model import Ridge`. Use `RidgeCV` to auto-tune $\alpha$ by cross-validation.

**Hyperparameters.**
- `alpha` (float, default `1.0`): regularization strength. Larger $\alpha$ = more shrinkage = simpler model.
- `solver` (`'auto'`, `'svd'`, `'cholesky'`, `'lsqr'`, `'sparse_cg'`, `'sag'`, `'saga'`): how to solve the optimization. `'auto'` picks for you. Use `'sag'`/`'saga'` for very large datasets.
- `max_iter`, `tol`: convergence controls for iterative solvers.

**Parameters (learned).** `fit_intercept`, `coef_`, `intercept_`.

**Use it when** you have many correlated features, or any time OLS overfits.

---

### 1.3 Lasso (L1-regularized OLS)

**Formal definition.** OLS with an L1 penalty.

**Intuition.** Same idea as ridge, but the L1 penalty has corners at zero, so it actually drives some coefficients to *exactly* zero. This makes Lasso a feature selector.

**Math.**
$$\hat{\beta} = \arg\min_{\beta} \frac{1}{2n}\|y - X\beta\|_2^2 + \alpha \|\beta\|_1$$

No closed form; solved by coordinate descent.

**scikit-learn.** `from sklearn.linear_model import Lasso` (and `LassoCV`).

**Hyperparameters.**
- `alpha`: regularization strength. Higher = more zeros.
- `max_iter`, `tol`: convergence.
- `selection` (`'cyclic'` or `'random'`): coordinate-update order; `'random'` is often faster.

**Use it when** you suspect only a small subset of features matter and you want the model to tell you which.

---

### 1.4 Elastic Net

**Formal definition.** A convex combination of L1 and L2 penalties.

**Intuition.** Lasso struggles with groups of correlated features (it tends to pick one and zero out the rest, somewhat arbitrarily). Elastic Net adds the ridge penalty back, so correlated features are kept together.

**Math.**
$$\hat{\beta} = \arg\min_{\beta} \frac{1}{2n}\|y - X\beta\|_2^2 + \alpha \rho \|\beta\|_1 + \frac{\alpha(1-\rho)}{2}\|\beta\|_2^2$$

**scikit-learn.** `ElasticNet(alpha, l1_ratio)` (`l1_ratio` is $\rho$). `ElasticNetCV` tunes both.

**Hyperparameters.** `alpha`, `l1_ratio` ($\in [0,1]$, with `0` = ridge and `1` = lasso).

**Use it when** you have many correlated features and want the sparsity of Lasso plus the stability of Ridge.

---

### 1.5 Logistic Regression

**Formal definition.** A linear model for binary classification: model $P(y=1 \mid x) = \sigma(\beta^\top x + \beta_0)$ where $\sigma(z) = 1/(1+e^{-z})$.

**Intuition.** Linear regression squashed through an S-curve so the output is a probability.

**Math.** Fit by minimizing the negative log-likelihood (cross-entropy):
$$\mathcal{L}(\beta) = -\sum_{i=1}^{n} \left[ y_i \log \sigma(\beta^\top x_i) + (1-y_i)\log(1-\sigma(\beta^\top x_i)) \right] + \text{regularization}$$

For multi-class, sklearn uses softmax (multinomial) or one-vs-rest.

**scikit-learn.** `from sklearn.linear_model import LogisticRegression`.

**Hyperparameters.**
- `C` (default `1.0`): *inverse* regularization strength. Smaller `C` = stronger regularization (note this is the opposite of `alpha` in Ridge/Lasso — a common gotcha).
- `penalty` (`'l2'`, `'l1'`, `'elasticnet'`, `None`): regularization type.
- `l1_ratio` (only for `'elasticnet'`).
- `solver` (`'lbfgs'`, `'liblinear'`, `'saga'`, `'newton-cg'`, `'sag'`): not all solvers support all penalties — `'saga'` is the most flexible; `'liblinear'` is good for small datasets and supports L1.
- `class_weight` (`'balanced'` or dict): re-weight classes; essential for imbalanced data.
- `max_iter`: increase if you see convergence warnings.
- `multi_class` (`'auto'`, `'ovr'`, `'multinomial'`): how to handle >2 classes.

**Use it when** you need a fast, interpretable baseline classifier — and a calibrated probability is a nice bonus.

---

## 2. Distance-Based Models

### 2.1 K-Nearest Neighbors (KNN)

**Formal definition.** A non-parametric "lazy" learner: to predict for $x$, find the $k$ closest training points and average their labels (regression) or take a majority vote (classification).

**Intuition.** Things that look similar tend to behave similarly. There's no training; the work happens at prediction time.

**Math.** Prediction:
$$\hat{y}(x) = \frac{1}{k}\sum_{i \in N_k(x)} y_i \quad \text{(regression)}$$
$$\hat{y}(x) = \arg\max_c \sum_{i \in N_k(x)} \mathbb{1}[y_i = c] \quad \text{(classification)}$$

where $N_k(x)$ is the set of $k$ nearest training points under a chosen distance metric.

**scikit-learn.** `KNeighborsClassifier`, `KNeighborsRegressor`.

**Hyperparameters.**
- `n_neighbors` (default `5`): the $k$. Small $k$ = wiggly decision boundary (high variance); large $k$ = smooth (high bias).
- `weights` (`'uniform'` or `'distance'`): whether closer neighbors count more.
- `metric` (`'minkowski'`, `'euclidean'`, `'manhattan'`, `'cosine'`, custom callable).
- `p` (with Minkowski): `p=2` is Euclidean, `p=1` is Manhattan.
- `algorithm` (`'auto'`, `'ball_tree'`, `'kd_tree'`, `'brute'`): how neighbors are found. Affects speed, not predictions.

**Use it when** the dataset is small to medium, the decision boundary is irregular, and you have a meaningful distance metric. **Always scale your features first** — KNN is fully driven by distances.

---

## 3. Probabilistic Models

### 3.1 Naive Bayes

**Formal definition.** Apply Bayes' theorem assuming features are conditionally independent given the class:
$$P(y \mid x_1, \ldots, x_p) \propto P(y) \prod_{j=1}^p P(x_j \mid y)$$

**Intuition.** Score each class by combining how common it is with how typical the features look for that class — but pretend features don't interact. The independence assumption is almost always wrong, yet the model often works surprisingly well, especially for text.

**scikit-learn.** Three flavors based on what $P(x_j \mid y)$ looks like:
- `GaussianNB` — continuous features, assumes Gaussian.
- `MultinomialNB` — count features (e.g. word counts).
- `BernoulliNB` — binary features.

**Hyperparameters.**
- `var_smoothing` (Gaussian): tiny constant added to variances for stability.
- `alpha` (Multinomial / Bernoulli): Laplace smoothing for unseen feature/class combos. `alpha=1.0` is the standard default.
- `fit_prior` (bool): whether to learn class priors from data or assume uniform.

**Use it when** you have a text-classification problem (spam, sentiment) or need an extremely fast baseline. Trains in seconds even on millions of documents.

---

### 3.2 Linear / Quadratic Discriminant Analysis (LDA / QDA)

**Formal definition.** Assume each class is Gaussian-distributed in feature space. LDA assumes all classes share a covariance matrix (giving linear boundaries); QDA lets each class have its own (giving quadratic boundaries).

**Math.** From Bayes' rule, decision boundaries are determined by
$$\delta_k(x) = -\frac{1}{2}(x - \mu_k)^\top \Sigma^{-1}(x - \mu_k) + \log \pi_k$$

where $\mu_k$, $\Sigma$, and $\pi_k$ are the class mean, covariance, and prior.

**scikit-learn.** `LinearDiscriminantAnalysis`, `QuadraticDiscriminantAnalysis`.

**Hyperparameters.**
- `solver` (LDA): `'svd'` (default, no covariance computation, good for many features), `'lsqr'`, `'eigen'`.
- `shrinkage` (LDA, with `'lsqr'`/`'eigen'`): regularizes the covariance estimate, useful when $p \approx n$.
- `n_components` (LDA): LDA can also be used for supervised dimensionality reduction.
- `reg_param` (QDA): regularization on the per-class covariance matrices.

**Use it when** the Gaussian assumption is plausible. LDA also doubles as a supervised dimensionality reducer.

---

## 4. Support Vector Machines

### 4.1 SVM (Classification and Regression)

**Formal definition.** A *maximum-margin* classifier. Find the hyperplane $w^\top x + b = 0$ that separates classes with the largest margin, allowing some slack for misclassifications.

**Intuition.** Don't just separate the classes — separate them with as much breathing room as possible. The "support vectors" are the points right at the margin; only they determine the boundary.

**Math.** The soft-margin primal is
$$\min_{w, b, \xi}\; \frac{1}{2}\|w\|_2^2 + C \sum_{i=1}^n \xi_i \quad \text{s.t. } y_i(w^\top x_i + b) \ge 1 - \xi_i,\; \xi_i \ge 0$$

The dual form lets you replace $x_i^\top x_j$ with a kernel $K(x_i, x_j)$ — the **kernel trick** — implicitly mapping data into a high-dimensional space without ever computing the mapping. Common kernels:
$$K_{\text{rbf}}(x, z) = \exp(-\gamma \|x - z\|^2), \quad K_{\text{poly}}(x, z) = (\gamma\, x^\top z + r)^d$$

For regression (SVR), the analogous loss is the $\epsilon$-insensitive loss: errors smaller than $\epsilon$ are ignored.

**scikit-learn.** `SVC`, `SVR`, `LinearSVC` (faster for linear kernels and large data), `NuSVC` / `NuSVR` (alternative parameterization).

**Hyperparameters.**
- `C` (default `1.0`): inverse regularization. Large `C` = penalize misclassifications heavily = harder margin = more overfitting.
- `kernel` (`'linear'`, `'rbf'`, `'poly'`, `'sigmoid'`, `'precomputed'`, callable): the similarity function.
- `gamma` (`'scale'`, `'auto'`, or float): RBF/poly width. Large `gamma` = small region of influence per point = wiggly boundary.
- `degree` (poly only): polynomial degree.
- `coef0` (poly/sigmoid): the $r$ in the kernel formula.
- `epsilon` (SVR only): width of the no-penalty tube.
- `class_weight` (`'balanced'` or dict): for imbalanced classification.
- `probability` (bool): enables probability estimates via Platt scaling, but slows training.

**Use it when** the dataset is small-to-medium (≤ ~100k samples), the decision boundary is non-linear, and you don't mind tuning. Always scale features. Past a few hundred thousand rows SVMs become painfully slow — switch to linear models or trees.

---

## 5. Tree-Based Models

### 5.1 Decision Tree

**Formal definition.** A model that recursively partitions feature space along axis-aligned splits. At each node, the algorithm picks the feature and threshold that most reduce an impurity measure.

**Intuition.** Twenty Questions for prediction. Each split asks "is feature X above threshold t?" and routes the sample left or right. Leaves hold the final prediction.

**Math.** For classification, common impurity measures are
$$\text{Gini}(t) = 1 - \sum_{c} p_c^2, \qquad \text{Entropy}(t) = -\sum_{c} p_c \log p_c$$

For regression, the impurity is variance (MSE) within the node. A split's quality is the parent impurity minus the weighted average of child impurities.

**scikit-learn.** `DecisionTreeClassifier`, `DecisionTreeRegressor`.

**Hyperparameters.**
- `criterion` (`'gini'`, `'entropy'`, `'log_loss'` for classifier; `'squared_error'`, `'absolute_error'`, `'friedman_mse'`, `'poisson'` for regressor).
- `max_depth`: hard cap on tree depth. The single most effective overfitting control.
- `min_samples_split`: minimum samples needed to split a node.
- `min_samples_leaf`: minimum samples in each leaf.
- `max_features`: how many features to consider per split.
- `max_leaf_nodes`: cap on total leaves.
- `min_impurity_decrease`: split only if impurity drop exceeds this.
- `ccp_alpha`: cost-complexity pruning parameter (post-prune by penalizing tree size).
- `class_weight` (classifier).
- `random_state`: trees are not unique when ties exist in splits, so set this for reproducibility.

**Use it when** you want maximum interpretability and don't mind sacrificing accuracy. In practice, single trees are mostly used as building blocks for ensembles.

---

### 5.2 Random Forest

**Formal definition.** An ensemble of decision trees, each trained on a bootstrap sample of the data and considering only a random subset of features at each split. Predictions are averaged (regression) or voted (classification).

**Intuition.** One tree overfits. But if you build many trees that are *decorrelated* (because of bootstrap + feature randomness), their errors cancel out and the average becomes accurate and stable. Lots of slightly-wrong opinions beat one confident wrong one.

**Math.** Final prediction:
$$\hat{y}(x) = \frac{1}{B} \sum_{b=1}^{B} T_b(x) \quad \text{(regression)}$$

The variance of the average is
$$\text{Var}\left(\frac{1}{B}\sum T_b\right) = \rho \sigma^2 + \frac{1-\rho}{B}\sigma^2$$

where $\rho$ is the average pairwise correlation between trees. The whole game is to keep $\rho$ low — that's why random feature subsets matter.

**scikit-learn.** `RandomForestClassifier`, `RandomForestRegressor`.

**Hyperparameters.** All of the decision-tree ones, plus:
- `n_estimators`: number of trees. More is always better for performance (but slower); somewhere between 100 and 1000 is typical.
- `bootstrap` (default `True`): bootstrap samples vs. use the whole dataset for each tree.
- `oob_score` (bool): use out-of-bag samples to get a free validation score (only if `bootstrap=True`).
- `max_features` (default `'sqrt'` for classifier, `1.0` for regressor): the key decorrelator. `'sqrt'` of total features per split is the classic recommendation.
- `n_jobs`: parallelism. Use `-1`.

**Use it when** you want a strong, low-tuning baseline for tabular data. Handles mixed feature types, doesn't need scaling, gives feature importances, and is hard to break.

---

### 5.3 Extra Trees (Extremely Randomized Trees)

Like Random Forest, but at each split it picks the threshold *randomly* rather than searching for the optimum. More variance reduction at the cost of more bias per tree. Often slightly faster and slightly less prone to overfit.

**scikit-learn.** `ExtraTreesClassifier`, `ExtraTreesRegressor`. Same hyperparameters as RF.

---

## 6. Boosting Models

Where bagging (Random Forest) builds trees independently in parallel, **boosting** builds them *sequentially*, with each tree correcting the previous ensemble's mistakes.

### 6.1 AdaBoost

**Formal definition.** Iteratively re-weight training samples — increasing the weight of misclassified ones — and combine many weak learners (typically shallow trees) into a weighted vote.

**Math.** At round $m$, train weak learner $h_m$ on weighted data, compute its error $\epsilon_m$, set $\alpha_m = \frac{1}{2}\log\frac{1-\epsilon_m}{\epsilon_m}$, and update sample weights by $w_i \leftarrow w_i \exp(\alpha_m \mathbb{1}[h_m(x_i) \neq y_i])$. Final prediction: $\text{sign}(\sum_m \alpha_m h_m(x))$.

**scikit-learn.** `AdaBoostClassifier`, `AdaBoostRegressor`.

**Hyperparameters.** `n_estimators`, `learning_rate` (shrinks each tree's contribution), `estimator` (the base learner, default a shallow decision tree).

**Use it when** you want a simple boosting baseline; rarely the best choice today since gradient boosting variants dominate.

---

### 6.2 Gradient Boosting (GBM)

**Formal definition.** Each new tree is fit to the *negative gradient* of the loss function with respect to the current ensemble's predictions. AdaBoost is actually a special case (gradient boosting with exponential loss).

**Intuition.** At each step, the ensemble's errors define a "residual" you'd like to reduce. Fit a small tree to predict those residuals, add it to the ensemble (scaled down by a learning rate), and repeat. You're doing gradient descent in function space.

**Math.** With loss $L$, current model $F_{m-1}$, fit $h_m \approx -\partial L / \partial F$ at $F_{m-1}$, then update
$$F_m(x) = F_{m-1}(x) + \nu \cdot h_m(x)$$

where $\nu$ is the learning rate. For squared loss, the negative gradient is just the residual $y - F_{m-1}(x)$.

**scikit-learn.** `GradientBoostingClassifier`, `GradientBoostingRegressor`. For larger data use `HistGradientBoostingClassifier` / `HistGradientBoostingRegressor` — these bin features into histograms (like LightGBM) and are vastly faster.

**Hyperparameters.**
- `n_estimators`: number of boosting rounds. With small `learning_rate`, you want more rounds.
- `learning_rate`: shrinkage. Smaller = more rounds needed but better generalization. There's a strong tradeoff with `n_estimators`.
- `max_depth`: depth of each tree. Boosting trees are usually shallow (3–8); the ensemble does the heavy lifting.
- `subsample`: fraction of samples used per tree (stochastic gradient boosting).
- `min_samples_split`, `min_samples_leaf`, `max_features`: same as in trees.
- `loss`: squared/absolute/huber for regression; log_loss/exponential for classification.
- `validation_fraction` + `n_iter_no_change`: early stopping.

**Use it when** you want top-tier accuracy on tabular data and are willing to tune.

---

### 6.3 XGBoost
**Formal definition.** A gradient boosting library that improves on basic gradient boosting in two ways: (1) it adds an explicit regularization penalty into the objective, and (2) it uses both first-derivative (gradient) and second-derivative (Hessian) information when building trees, rather than just gradients.

**Use it when** you want strong tabular performance with a battle-tested library. Workhorse of Kaggle and a sensible default for structured data problems like the F1 predictor. Pair with `GroupKFold` when rows share natural groupings (multiple drivers per race).


## What It Is
 XGBoost (Extreme Gradient Boosting) is a sequential ensemble learning method. You start with a dumb initial prediction (the mean of the target), then repeatedly build small decision trees, each one trained to correct the errors (residuals) of everything before it.  
After $T$ rounds, the prediction for sample $i$ is:
$$\hat{y}_i = \bar{y} + \eta \cdot f_1(x_i) + \eta \cdot f_2(x_i) + \cdots + \eta \cdot f_T(x_i)$$

> where each $f_t$ is a decision tree and $\eta$ is the learning rate (shrinkage).



## How a Single Tree Works

A decision tree asks sequential yes/no questions to partition data into groups called **leaves**. Each leaf holds a group of training samples and outputs one number — the **leaf weight** $w_j$ — which is the prediction for all samples in that leaf.

```
                All 400 drivers
                (avg position: 10.5)
                    |
            grid_position ≤ 5?
            /              \
        YES                NO
    80 drivers          320 drivers
    (avg pos: 3.2)      (avg pos: 12.3)
        |                    |
    wet = true?        grid_position ≤ 15?
    /      \             /          \
    YES      NO          YES          NO
20 drv    60 drv      200 drv      120 drv
(avg 4.8) (avg 2.7)   (avg 10.1)  (avg 15.9)  ← these are leaves
```

**Depth** is the number of questions from root to leaf. Depth controls interaction order — depth 1 can only use one feature, depth 6 can capture six-way interactions like "grid ≤ 3 AND dry AND constructor = Red Bull AND ..."

---

## How Boosting Works — Step by Step

**Round 0:** Predict the mean for everyone: $\hat{y}_i^{(0)} = 10.5$

| Driver | Actual | Prediction | Residual |
|--------|--------|------------|----------|
| Hamilton | 2 | 10.5 | −8.5 |
| Verstappen | 1 | 10.5 | −9.5 |
| Latifi | 18 | 10.5 | +7.5 |

**Round 1:** Build a tree that predicts the **residuals**, not the original target. The tree learns patterns like "front-starters were underpredicted." Each leaf computes an optimal correction using:

$$w_j^* = -\frac{\sum_{i \in \text{leaf}_j} g_i}{\sum_{i \in \text{leaf}_j} h_i + \lambda}$$

For squared error: $g_i = \hat{y}_i - y_i$ (the residual), $h_i = 1$. So this simplifies to: sum of residuals / (count + λ).

Update predictions: $\hat{y}_i^{(1)} = \hat{y}_i^{(0)} + \eta \cdot f_1(x_i)$

**Round 2, 3, ... T:** Compute new residuals from updated predictions. Build another tree on those residuals. Add it to the running sum. Each round nudges predictions closer to truth.

---

## What "Gradient" and "Hessian" Mean Here

For squared error loss $\ell = \frac{1}{2}(\hat{y}_i - y_i)^2$:

**Gradient** $g_i = \hat{y}_i - y_i$ — the first derivative. It's just the residual: direction and magnitude of the error.

**Hessian** $h_i = 1$ — the second derivative. For squared error this is always 1, which is why the leaf weight formula simplifies to "sum of residuals / (count + λ)."

The Hessian becomes interesting with other loss functions (e.g. logistic loss for classification, where $h_i$ varies per sample). XGBoost uses both gradient and Hessian so it works with any twice-differentiable loss — this is what makes it "second-order" and distinguishes it from plain gradient boosting.

---

## The Role of the Taylor Expansion

The Taylor expansion is **not** part of the objective. The actual objective XGBoost minimizes is:

$$\mathcal{L} = \underbrace{\sum_i \ell(y_i, \hat{y}_i)}_{\text{exact loss}} + \underbrace{\sum_m \Omega(f_m)}_{\text{regularization}}$$

The Taylor expansion is an internal shortcut used *during tree construction*. When XGBoost asks "how much will the loss change if I add this tree?", evaluating the exact answer for every possible tree is intractable. So it approximates the change using a second-order Taylor expansion, which lets it derive the closed-form leaf weight formula above. Once the tree is built and added, the next round uses exact residuals again.

$$\Omega(f) = \gamma T + \frac{1}{2}\lambda \sum_{j=1}^{T} w_j^2$$

where $T$ is the number of leaves in the tree, $w_j$ is the weight (output value) of leaf $j$, $\gamma$ penalizes having more leaves (discourages complex trees), and $\lambda$ penalizes large leaf weights (shrinks extreme corrections toward zero).


**Interview-level answer:** "The leaf weight formula is the closed-form solution you get from minimizing the second-order approximation of the loss plus the regularization penalty." You don't need to derive it.

---

## Regularization — What It Is and Why It Matters

**Overfitting** means the model memorizes noise in the training data. Example: a safety car at one race caused a driver to gain 5 positions — a one-off event. If a tree builds a leaf capturing only that race, it learns a rule it'll incorrectly apply to future races.

**Regularization** is anything that prevents this by making the model pay a cost for complexity. XGBoost has four independent regularization knobs, each fighting overfitting from a different angle: (1) Lambda(`reg_lambda`) shrinks the numerical output of each leaf to avoid overfitting(2) Max number of leaves per tree (`max_depth`) - limits tree complexity from the top, overtly complex trees overfit (3) `min_child_weight` - minimum samples per leaf - limits complexity from the bottom (how many samples justify a new leaf) (4) `subsample` or `colsample_bytree` - randomizing what each tree sees so errors decorrelate and cancel out when averaged. 

---

## Hyperparameters — What Each Does and Why

### `learning_rate` (η) — shrinkage per tree

**What it does:** Multiplies each tree's contribution by a small number (e.g. 0.05). Each tree only gets to make a 5% correction.

**Effect:** Increases training error — each tree undercorrects.

**Why that's good:** Forces the model to need many rounds to converge. Patterns that are real keep getting reinforced across rounds. One-off noise from a single tree gets diluted by hundreds of subsequent trees. Smaller η = better generalization, but need more trees (slower training).

### `n_estimators` — number of trees

**What it does:** Maximum number of boosting rounds (trees).

**Effect:** More trees = more chances to correct residuals = lower training error.

**Why to be careful:** Too many trees and the model starts correcting noise. Set this high and use early stopping to automatically find the right number.

### `max_depth` — tree complexity from the top

**What it does:** Limits how many sequential questions each tree can ask.

**Effect:** Restricts the interaction order the model can learn. Depth 6 = up to 6-way feature interactions.

**Why that's good:** Shallow trees are weak learners that can't memorize specific races. Deeper trees are powerful enough to overfit individual data points. Depth 4–8 is typical.

### `min_child_weight` — tree complexity from the bottom

**What it does:** Minimum Sum of Hessians required for a node split.With objective as `reqressionsqrd` it is simply the number of samples required in a leaf (because $h_i = 1$ for all rows).

**Effect:** Blocks splits that would create a leaf with too little Hessian mass — i.e. too little statistical evidence. For squared error this is a floor on sample count; for other objectives (e.g. logistic) it's a floor on total curvature/uncertainty, since $h_i$ then varies per row.

**Why that's good:** A leaf backed by little Hessian mass is fitting too few informative rows — memorizing rather than generalizing. Requiring more mass per leaf forces each split to be supported by enough signal, filtering out noise-driven splits. Higher value → fewer, larger leaves → more regularization.

**Typical range:** 1–10; try 3–5 as a starting point for noise control.

### `reg_lambda` (λ) — L2 regularization on leaf weights

**What it does:** Inflates the denominator of the leaf weight formula, shrinking every leaf's output toward zero.

Without λ: $w = \text{sum of residuals} / \text{count}$ (the raw mean residual)

With λ: $w = \text{sum of residuals} / (\text{count} + \lambda)$

**Effect:** Deliberately increases training error — every tree undercorrects.Without it, leaves have to fit large corrections and large corrections are usually fitting noise. 

**Why that's good:** Leaves with few samples get shrunk the most (adding λ to 3 is a 33% increase; adding λ to 300 is a 0.3% increase). This disproportionately dampens the corrections most likely to be noise. The subsequent trees then pick up whatever consistent signal remains — but one-off noise doesn't persist across rounds, so it gets filtered out naturally.

**Example with 3 samples summing to residual −24:**

| λ | Leaf weight | Effect |
|---|------------|--------|
| 0 | −24/3 = −8.0 | Full correction, possible overfitting |
| 1 | −24/4 = −6.0 | Moderate shrinkage |
| 10 | −24/13 = −1.85 | Heavy shrinkage, very conservative |

### `reg_alpha` — L1 regularization on leaf weights

**What it does:** Penalizes the absolute value of leaf weights (not squared).

**Effect:** Can push some leaf weights to exactly zero (sparsity).

**Why that's good:** Effectively prunes irrelevant leaves — if a leaf's correction is tiny, L1 kills it entirely.

### `gamma` (γ) — minimum split gain

**What it does:** A split is only allowed if it reduces the loss by at least γ.

**Effect:** Fewer splits = simpler trees.

**Why that's good:** Prevents marginal splits that barely improve training loss but add complexity.

### `subsample` — row subsampling

**What it does:** Each tree sees a random fraction (e.g. 80%) of the training rows.

**Effect:** Different trees overfit to different noise. When you average predictions across hundreds of trees, the noise cancels out and the signal remains.

**Why that's good:** Reduces **variance** (how much predictions change with different training data). Without subsampling, every tree sees the same outlier race and every tree tries to fit it — the ensemble's mistakes are correlated. With subsampling, tree #47 might miss that outlier entirely while tree #112 misses a different one. Decorrelated errors average out.

### `colsample_bytree` — column subsampling

**What it does:** Each tree only gets a random fraction of features to consider.

**Effect:** Prevents dominant features (like `grid_position`) from appearing at the root of every tree.

**Why that's good:** Forces the model to discover signal in weaker features. Creates a more diverse ensemble where different trees rely on different feature subsets.

### `tree_method` — how splits are found

**What it does:** Controls the algorithm for finding the best split point at each node.

- `exact`: evaluates every unique feature value as a candidate — accurate but slow.
- `hist`: bins features into ~256 histogram buckets, only evaluates bin boundaries — 5-10x faster with negligible accuracy loss. Modern default.

### `early_stopping_rounds` — automatic n_estimators selection

**What it does:** Monitors validation loss during training. If it hasn't improved for N rounds, stop.

**Why that's good:** You set `n_estimators` high (e.g. 2000) and let early stopping pick the actual best round. No risk of under- or over-training.

### `verbosity` — logging level

0 = silent, 1 = warnings only, 2 = info, 3 = debug.

### `random_state` — reproducibility

Seeds the RNG. Without it, row/column subsampling and tie-breaking give slightly different results each run.

### `objective` - Loss Function
 loss function that defines $g_i$ and $h_i$. `'reg:squarederror'`, `'binary:logistic'`, `'multi:softprob'`, `'rank:pairwise'`, etc.

---
## XGBoost Loss Functions
**Regression** (predicting a continuous number):
- `reg:squarederror` — default, penalizes large errors heavily. Use for most regression tasks (F1 finishing position, house prices, stock returns).
- `reg:squaredlogerror` — squared error on log-transformed predictions. Use when you care about relative error and targets are positive with large range (e.g. predicting revenue where being off by $100 matters more at $500 than at $50,000).
- `reg:absoluteerror` — MAE loss, penalizes all errors equally. Use when you have outliers you don't want to dominate training (e.g. if DNFs cause extreme residuals).

**Binary classification** (yes/no, 0/1):
- `binary:logistic` — outputs a probability between 0 and 1. Use for most two-class problems (will this driver DNF? win/lose prediction).

**Multiclass classification** (one of N categories):
- `multi:softmax` — outputs the predicted class label directly. Use when you just need the answer.
- `multi:softprob` — outputs probabilities for each class. Use when you need confidence scores (e.g. "60% chance this is a pit strategy A, 30% B, 10% C").

**Ranking** (ordering items relative to each other):
- `rank:pairwise` — learns to rank items correctly relative to each other within a group. This is arguably the most correct framing for F1 — you care about finishing *order* within a race, not the exact position number.
- `rank:ndcg` — optimizes NDCG (normalized discounted cumulative gain), which weights getting the top positions right more heavily than the bottom. Use for search results or recommendations where top-k accuracy matters most.

**Count data:**
- `count:poisson` — for predicting counts (number of pit stops, number of overtakes). Assumes the target follows a Poisson distribution.

**Survival analysis:**
- `survival:cox` — for time-to-event data (how long until engine failure, customer churn).

> XGBoost specifically requires the loss function to be twice differentiable because it needs both gig_i
gi​ and hih_i
hi​. That's why you won't find something like 0-1 loss (correct or wrong, no gradient) in XGBoost — there's no smooth derivative to work with.

---
## The Regularization Mental Model

All four knobs fight overfitting from different angles:

| Knob | Controls | Mechanism |
|------|----------|-----------|
| `max_depth` | Tree complexity from top | Limits interaction order |
| `min_child_weight` | Tree complexity from bottom  Prevents small-sample leaves |
| `reg_lambda` | Leaf weight magnitude | Shrinks corrections, especially in small leaves |
| `subsample` / `colsample_bytree` | What each tree sees | Decorrelates trees so noise cancels out |

Using all at moderate values is almost always better than cranking one to an extreme. They're complementary — each catches a different kind of overfitting.

---

## Why XGBoost Over Alternatives (for tabular data like F1 prediction)

**Linear/Logistic Regression:** Cannot capture non-linear interactions (grid × track × weather) without heavy manual feature engineering. Too rigid.

**Random Forests:** Handle non-linearity but average independent trees. Boosting sequentially corrects residuals, which fits noisy regression better. RF is an acceptable baseline; XGBoost almost always beats it on tabular data.

**Neural Networks (MLP, TabNet):** Need much more data than F1 provides (~20 races/season). Per Shwartz-Ziv & Armon (2022), gradient boosting still wins on small-to-medium tabular datasets.

**LightGBM / CatBoost:** Genuinely competitive. XGBoost is chosen for mature ecosystem, native missing-value handling, and easy SHAP integration. LightGBM is a planned A/B comparison.

---

## Key Insight to Remember

Lambda (and the learning rate) deliberately **increase training error**. Each tree undercorrects. But patterns that are real keep showing up in the residuals round after round, so subsequent trees reinforce them. One-off noise doesn't persist across rounds, so it naturally gets filtered out. That's the core mechanism: regularization + many rounds = noise dies, signal survives.

---

### 6.4 LightGBM

A Microsoft library that's a cousin of XGBoost. Two key differences: it grows trees **leaf-wise** (expanding the leaf with the largest loss reduction) instead of level-wise, and it uses histogram-based splits with two extra tricks — Gradient-based One-Side Sampling (GOSS) and Exclusive Feature Bundling (EFB) — to handle large datasets fast.

**Library.** `lightgbm`: `LGBMClassifier`, `LGBMRegressor`, `LGBMRanker`.

**Hyperparameters.** Mostly mirror XGBoost (`n_estimators`, `learning_rate`, `subsample`, `colsample_bytree`, `reg_alpha`, `reg_lambda`, `min_child_weight`), plus:
- `num_leaves`: cap on leaves per tree. *This is the main complexity knob* — it replaces `max_depth` for leaf-wise growth. A common rule of thumb is `num_leaves < 2^max_depth`.
- `min_data_in_leaf`: prevents overfitting on tiny leaves.
- `boosting_type`: `'gbdt'` (default), `'dart'` (with dropout), `'goss'`.
- `feature_fraction`, `bagging_fraction`, `bagging_freq`: sampling controls.
- `categorical_feature`: pass categorical indices and LightGBM handles them natively (no one-hot).

**Use it when** XGBoost is too slow or you have many categorical features.

---

### 6.5 CatBoost

Yandex's offering. Two distinguishing features: native, sophisticated handling of categorical variables (no preprocessing needed), and **ordered boosting** — a permutation-driven scheme that reduces the prediction shift bias that other gradient boosters suffer from.

**Library.** `catboost`: `CatBoostClassifier`, `CatBoostRegressor`.

**Hyperparameters.** `iterations` (= `n_estimators`), `learning_rate`, `depth` (CatBoost uses *symmetric* trees by default), `l2_leaf_reg`, `cat_features` (list of categorical column indices), `border_count`, `bagging_temperature`.

**Use it when** your dataset is rich in categorical features.

---

## 7. Clustering (Unsupervised)

### 7.1 K-Means

**Formal definition.** Partition $n$ points into $k$ clusters by minimizing within-cluster sum of squares.

**Math.**
$$\min_{\{C_k\}, \{\mu_k\}} \sum_{k=1}^K \sum_{x \in C_k} \|x - \mu_k\|_2^2$$

Lloyd's algorithm alternates: (1) assign each point to its nearest centroid; (2) recompute each centroid as the mean of its assigned points. Repeat until assignments stop changing.

**scikit-learn.** `KMeans`. Use `MiniBatchKMeans` for very large datasets.

**Hyperparameters.**
- `n_clusters`: the $k$. Pick using the elbow method, silhouette score, or domain knowledge.
- `init` (`'k-means++'` or `'random'`): smart initialization is essential — `k-means++` is the default and almost always what you want.
- `n_init`: number of different initializations to try. Default is `'auto'`.
- `max_iter`, `tol`: convergence.
- `algorithm` (`'lloyd'`, `'elkan'`).

**Use it when** you expect roughly spherical, similarly-sized clusters. Always scale features.

---

### 7.2 DBSCAN

**Formal definition.** A density-based cluster forms wherever there are at least `min_samples` points within distance `eps` of each other. Points in low-density regions are labeled noise.

**Intuition.** A cluster is a dense crowd; outliers are loners. Unlike K-Means, you don't specify the number of clusters and shape doesn't have to be spherical.

**scikit-learn.** `DBSCAN`. For larger data: `HDBSCAN` (in sklearn 1.3+) handles varying densities better.

**Hyperparameters.**
- `eps`: neighborhood radius. The hardest hyperparameter — tools like the k-distance plot help.
- `min_samples`: minimum density for a core point. Often set to `2 * n_features`.
- `metric`.

**Use it when** clusters are non-convex, contain outliers, or you don't know $k$.

---

### 7.3 Agglomerative (Hierarchical) Clustering

**Formal definition.** Bottom-up: start with every point as its own cluster, repeatedly merge the closest pair, until you reach the desired number of clusters.

**scikit-learn.** `AgglomerativeClustering`.

**Hyperparameters.**
- `n_clusters` *or* `distance_threshold` (one of the two).
- `linkage` (`'ward'`, `'complete'`, `'average'`, `'single'`): how to measure inter-cluster distance. Ward minimizes within-cluster variance and is the most popular.
- `metric`.

**Use it when** you want a dendrogram (visualizing the merge tree) or hierarchical structure matters substantively.

---

### 7.4 Gaussian Mixture Models (GMM)

**Formal definition.** Model the data as a mixture of $K$ Gaussians: $p(x) = \sum_{k=1}^K \pi_k \mathcal{N}(x \mid \mu_k, \Sigma_k)$. Fit with the Expectation-Maximization algorithm.

**Intuition.** Soft K-Means: each point belongs to each cluster with a probability instead of a hard assignment.

**scikit-learn.** `GaussianMixture` (and `BayesianGaussianMixture` for an automatic component count).

**Hyperparameters.** `n_components`, `covariance_type` (`'full'`, `'tied'`, `'diag'`, `'spherical'`), `max_iter`, `n_init`, `reg_covar`.

**Use it when** clusters overlap or are elliptical, or you want soft assignments.

---

## 8. Dimensionality Reduction

### 8.1 Principal Component Analysis (PCA)

**Formal definition.** Find the orthogonal directions of maximum variance in the data — the eigenvectors of the covariance matrix.

**Math.** With centered data $X$, compute $\Sigma = \frac{1}{n}X^\top X$ and find its top-$k$ eigenvectors $W$. The reduced representation is $Z = XW$. Equivalently via SVD: $X = U \Lambda V^\top$, and $W$ is the top columns of $V$.

**scikit-learn.** `PCA`. Variants: `IncrementalPCA` (streaming), `SparsePCA`, `KernelPCA` (non-linear).

**Hyperparameters.** `n_components` (int, fraction of variance like `0.95`, or `'mle'`), `whiten`, `svd_solver`.

**Use it when** features are correlated and you want a compact, decorrelated representation. Almost always center/scale first.

---

### 8.2 t-SNE

**Formal definition.** A non-linear projection that preserves local neighborhoods. It minimizes the KL divergence between a similarity distribution in the original space and one in 2D/3D.

**scikit-learn.** `TSNE`.

**Hyperparameters.** `n_components` (almost always 2), `perplexity` (effective number of neighbors, typically 5–50), `learning_rate`, `n_iter`, `init` (`'pca'` is more stable than `'random'`).

**Use it when** you want to visualize high-dimensional data. Don't use t-SNE for downstream modeling — distances in t-SNE space are not meaningful.

---

### 8.3 UMAP

A more recent alternative to t-SNE. Faster, scales better, and (compared to t-SNE) preserves more of the global structure.

**Library.** `umap-learn`: `umap.UMAP`.

**Hyperparameters.** `n_components`, `n_neighbors` (local-vs-global tradeoff), `min_dist` (how tightly points cluster), `metric`.

**Use it when** t-SNE feels too slow or you need slightly more faithful global structure.

---

## 9. Neural Networks

Neural networks are stacks of differentiable layers trained by gradient descent. Scikit-learn includes a basic MLP, but for any serious work you should use PyTorch, JAX, or Keras/TensorFlow.

### 9.1 Multilayer Perceptron (MLP)

**Formal definition.** A feed-forward network of fully-connected layers:
$$\hat{y} = f_L(W_L \cdot f_{L-1}(W_{L-1} \cdots f_1(W_1 x + b_1) \cdots) + b_L)$$

where each $f$ is a non-linear activation (ReLU, tanh, sigmoid).

**Intuition.** Stacked logistic regressions with non-linearities in between. Each layer learns a representation that the next layer linearly combines. Universal approximators in theory, but only as good as your data and training.

**Math.** Trained by minimizing a loss (cross-entropy, MSE) via backpropagation — the chain rule applied through the network — combined with stochastic gradient descent or a variant like Adam.

**scikit-learn.** `MLPClassifier`, `MLPRegressor`. Fine for prototyping, **not** for production deep learning.

**Hyperparameters.**
- `hidden_layer_sizes`: tuple, e.g. `(100,)` or `(64, 32)`.
- `activation` (`'relu'`, `'tanh'`, `'logistic'`, `'identity'`).
- `solver` (`'adam'`, `'sgd'`, `'lbfgs'`).
- `alpha`: L2 penalty.
- `learning_rate_init`, `learning_rate` (schedule).
- `batch_size`.
- `max_iter`.
- `early_stopping`, `validation_fraction`, `n_iter_no_change`.

For real work, the equivalent in PyTorch:
```python
import torch.nn as nn
model = nn.Sequential(nn.Linear(d_in, 64), nn.ReLU(), nn.Linear(64, d_out))
```

**Use it when** you have tabular data with strong non-linear interactions and a lot of it. For most tabular problems, gradient-boosted trees still win.

---

### 9.2 Convolutional Neural Networks (CNN)

**Formal definition.** A network whose layers are *convolutions* — small filter kernels slid across the input — plus pooling and non-linearities. Designed to exploit spatial locality and translation equivariance.

**Math.** A 2D convolution: $(I * K)(i, j) = \sum_m \sum_n I(i+m, j+n)\, K(m, n)$.

**Library.** PyTorch (`torch.nn.Conv2d`), Keras (`tf.keras.layers.Conv2D`).

**Key hyperparameters.** `in_channels`, `out_channels`, `kernel_size`, `stride`, `padding`, `dilation`. Beyond a single layer: `n_layers`, learning rate, optimizer choice, batch size, dropout, batch norm, weight decay, augmentation strategy.

**Use it when** the input has spatial or local structure: images, audio spectrograms, certain time series. **Connects to your interests:** processing FPGA or DSP signal data with CNNs is a natural fit when the signal has local patterns.

---

### 9.3 Recurrent Networks: RNN, LSTM, GRU

**Formal definition.** Networks that process sequences by maintaining a hidden state $h_t$ updated at each step: $h_t = f(W_h h_{t-1} + W_x x_t + b)$. LSTMs and GRUs add gating mechanisms to combat vanishing gradients across long sequences.

**LSTM math (compactly).** A cell maintains a state $c_t$ and hidden $h_t$, regulated by *forget*, *input*, and *output* gates:
$$f_t = \sigma(W_f [h_{t-1}, x_t]), \quad i_t = \sigma(W_i [h_{t-1}, x_t]), \quad o_t = \sigma(W_o [h_{t-1}, x_t])$$
$$c_t = f_t \odot c_{t-1} + i_t \odot \tanh(W_c [h_{t-1}, x_t]), \quad h_t = o_t \odot \tanh(c_t)$$

**Library.** PyTorch: `nn.RNN`, `nn.LSTM`, `nn.GRU`.

**Key hyperparameters.** `input_size`, `hidden_size`, `num_layers`, `bidirectional`, `dropout`.

**Use it when** sequence order matters and the sequence isn't enormous. For long sequences and large data, transformers have largely replaced RNNs.

---

### 9.4 Transformers

**Formal definition.** A sequence model built around the **self-attention** mechanism. Instead of recurrence, the model lets every token attend to every other token in parallel.

**Math.** Scaled dot-product attention:
$$\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{Q K^\top}{\sqrt{d_k}}\right) V$$

Multi-head attention runs several attention operations in parallel and concatenates them. A transformer block is a stack of multi-head attention + feed-forward sublayers with residual connections and layer norm.

**Library.** PyTorch (`nn.Transformer`), HuggingFace `transformers` (the standard for pretrained models).

**Key hyperparameters.** `d_model` (hidden size), `nhead` (number of attention heads), `num_encoder_layers` / `num_decoder_layers`, `dim_feedforward`, `dropout`.

**Use it when** you have long-range dependencies in sequences, large datasets, and the compute to train them. Most modern NLP and increasingly vision (ViT) and time series rely on transformers.

---

### 9.5 Autoencoders

**Formal definition.** A network trained to reconstruct its input through a low-dimensional bottleneck: $\hat{x} = g(f(x))$ minimizing $\|x - \hat{x}\|^2$.

**Variants.** Denoising autoencoders, sparse autoencoders, **variational autoencoders (VAEs)** (which impose a probabilistic latent), and contractive autoencoders.

**Use it when** you want learned dimensionality reduction, anomaly detection (high reconstruction error = anomaly), or generative modeling (VAEs).

---

## 10. Ensemble Meta-Methods

These wrap *other* models to combine them.

### 10.1 Bagging

Bootstrap aggregating: fit base models on bootstrap samples and average. Random Forest is a special case where the base model is a tree with random feature subsets.

**scikit-learn.** `BaggingClassifier`, `BaggingRegressor`. Hyperparameters: `estimator`, `n_estimators`, `max_samples`, `max_features`, `bootstrap`, `bootstrap_features`.

### 10.2 Voting

Train several different models, then combine via majority vote (`'hard'`) or by averaging predicted probabilities (`'soft'`).

**scikit-learn.** `VotingClassifier`, `VotingRegressor`. Hyperparameters: `estimators`, `voting`, `weights`.

### 10.3 Stacking

Train several base models and feed their out-of-fold predictions into a *meta-learner* (typically logistic regression or a small tree model) that learns how to combine them.

**scikit-learn.** `StackingClassifier`, `StackingRegressor`. Hyperparameters: `estimators`, `final_estimator`, `cv`, `passthrough`.

**Use these when** you have several reasonably good but diverse models and want to squeeze out a bit more performance.

---

## 11. Choosing a Model: A Rough Guide

For **regression on tabular data**, start with linear regression (with Ridge if features are correlated, Lasso if you need feature selection). If that under-performs, jump to a gradient boosting model (XGBoost, LightGBM, or HistGradientBoosting). Random Forest is a solid intermediate baseline.

For **classification on tabular data**, the same hierarchy applies: logistic regression, then Random Forest, then gradient boosting. Naive Bayes for text. SVMs if you have a small-to-medium dataset and a non-linear pattern.

For **images**, CNNs (often pretrained, fine-tuned). Vision transformers for cutting-edge.

For **text**, transformers (typically pretrained — fine-tune BERT, GPT-style, or use embedding models).

For **time series**, classical statistical models (ARIMA, ETS) for short, well-behaved series; gradient boosting on engineered features (lags, rolling stats) for tabular-style forecasting; LSTMs or transformers for very long or complex sequences. Your F1 race-prediction project sitting in the gradient-boosting bucket is the right call: race outcomes depend on a rich set of engineered features, not raw long sequences.

For **clustering**, K-Means if you expect spherical clusters and know $k$; DBSCAN if you don't and there's noise; GMM for soft assignment.

For **dimensionality reduction**, PCA for linear and downstream-friendly; UMAP/t-SNE for visualization only.

Two general rules: (1) start with the simplest model that could plausibly work, and (2) on tabular data, gradient-boosted trees beat deep learning the vast majority of the time.

---

## 12. Generic, Cross-Cutting Hyperparameters and Utilities

These don't belong to any particular model; they're how you set up the experiment around a model.

### 12.1 Train / Test Splitting

**`train_test_split`** (`sklearn.model_selection`). Random split into train and test sets.
- `test_size`: fraction (e.g. `0.2`) or count.
- `random_state`: reproducibility.
- `stratify`: pass `y` for classification to preserve class proportions in both splits.
- `shuffle`: set `False` for time series.

### 12.2 Cross-Validation

**`KFold`** — basic $k$-fold split. Hyperparameters: `n_splits`, `shuffle`, `random_state`.

**`StratifiedKFold`** — preserves class ratios per fold. Use this by default for classification.

**`GroupKFold`** — keeps all samples from the same group in the same fold. *Critical* whenever rows aren't independent — e.g., multiple laps from the same race, multiple measurements from the same patient. This is what prevents the leakage you've been guarding against in the F1 predictor.

**`StratifiedGroupKFold`** — combines both: stratification *and* group integrity.

**`TimeSeriesSplit`** — expanding-window CV for forecasting; never lets future leak into past.

**`cross_val_score(model, X, y, cv, scoring)`** — quick cross-validated metric.

**`cross_validate`** — same but returns multiple metrics, fit times, and optionally the trained estimators.

### 12.3 Hyperparameter Search

**`GridSearchCV`** — exhaustive search over a parameter grid. Hyperparameters: `estimator`, `param_grid`, `cv`, `scoring`, `n_jobs`, `refit`.

**`RandomizedSearchCV`** — sample from distributions; usually beats grid search on the same compute budget. Hyperparameters: `param_distributions`, `n_iter`, `cv`, `scoring`.

**`HalvingGridSearchCV` / `HalvingRandomSearchCV`** — successive halving: start with many candidates and a small data sample, eliminate weak ones, give survivors more data. Often dramatically faster.

For more sophisticated tuning, look outside sklearn: **Optuna** (Bayesian/TPE search with pruning) and **Hyperopt** are the go-to libraries.

### 12.4 Preprocessing and Scaling

**`StandardScaler`** — subtract mean, divide by std. The default. Hyperparameters: `with_mean`, `with_std`.

**`MinMaxScaler`** — scale to `[0, 1]`. Hyperparameters: `feature_range`.

**`RobustScaler`** — uses median and IQR; resistant to outliers.

**`Normalizer`** — scales each row to unit norm (rarely what you want unless you specifically need it).

**`PowerTransformer`** — Box-Cox or Yeo-Johnson; makes data more Gaussian-like. Hyperparameters: `method`, `standardize`.

**`QuantileTransformer`** — maps to uniform or normal via the empirical CDF.

Scaling matters for: linear models (with regularization), KNN, SVM, neural networks, K-Means, PCA. It does *not* matter for tree-based models.

### 12.5 Categorical Encoding

**`OneHotEncoder`** — creates a binary column per category. Hyperparameters: `drop` (`'first'`, `'if_binary'`), `handle_unknown` (`'ignore'` is safest), `sparse_output`, `min_frequency`, `max_categories`.

**`OrdinalEncoder`** — integer codes; appropriate when categories have order or for tree models.

**`LabelEncoder`** — for the *target*, not features.

**`TargetEncoder`** (sklearn 1.3+) — replaces a category with the mean target value, with internal cross-fitting to prevent leakage. Powerful for high-cardinality features.

### 12.6 Imputation

**`SimpleImputer`** — fill with mean, median, mode, or constant.

**`KNNImputer`** — fill using nearest-neighbor values.

**`IterativeImputer`** — model each feature as a function of the others, iteratively (a.k.a. MICE).

### 12.7 Feature Selection

**`SelectKBest(score_func, k)`**, **`SelectPercentile`** — univariate filters.

**`RFE` / `RFECV`** — recursive feature elimination using a model that exposes coefficients or feature importances.

**`SelectFromModel`** — keep features whose importance exceeds a threshold.

**`VarianceThreshold`** — drop near-constant features.

### 12.8 Pipelines

**`Pipeline`** — chain preprocessing and a model into a single estimator. Crucial for avoiding leakage during cross-validation: the scaler is fit only on the training fold each time.

**`ColumnTransformer`** — apply different preprocessing to different column subsets (e.g. one-hot encode strings, scale numerics).

**`make_pipeline` / `make_column_transformer`** — convenience constructors.

### 12.9 Class Imbalance

- `class_weight='balanced'`: most sklearn classifiers (and XGBoost/LightGBM via `scale_pos_weight`) re-weight the loss inversely to class frequency.
- `sample_weight`: pass per-row weights to `.fit()`.
- **`imbalanced-learn`** (`imblearn`): SMOTE, ADASYN, undersampling, and `imblearn.pipeline.Pipeline` (which understands resamplers).

### 12.10 Common Metrics (`sklearn.metrics`)

Regression: `mean_squared_error`, `mean_absolute_error`, `r2_score`, `mean_absolute_percentage_error`.

Classification: `accuracy_score`, `precision_score`, `recall_score`, `f1_score`, `roc_auc_score`, `log_loss`, `confusion_matrix`, `classification_report`. For imbalanced problems, ROC-AUC and average precision (`average_precision_score`) are more informative than accuracy.

Ranking: `ndcg_score`, `dcg_score` — relevant for race-position-style problems where ordinal accuracy matters more than exact prediction.

Clustering: `silhouette_score`, `adjusted_rand_score`, `normalized_mutual_info_score`.

### 12.11 Calibration and Diagnostics

**`CalibratedClassifierCV`** — wrap a classifier to produce well-calibrated probabilities (Platt or isotonic).

**`learning_curve`**, **`validation_curve`** — plot performance vs. training-set size or a hyperparameter to diagnose underfit vs. overfit.

**`permutation_importance`** — model-agnostic feature importance by shuffling each feature and measuring performance drop. More reliable than tree built-in importances when features have different scales or cardinalities.

### 12.12 Spearman and Pearson COrrelation for Ranking 
Pearson high → strong linear relationship. The points hug a straight line.
Spearman high → strong monotonic relationship. As one goes up the other consistently goes up — but not necessarily along a straight line. Any consistently-increasing curve scores near 1.

---


