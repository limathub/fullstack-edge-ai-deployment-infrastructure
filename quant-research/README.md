[← Back to overview](../README.md) · **Language:** English | [한국어](./README.ko.md)

# Quantitative Research — A Return-Forecasting Feature and Structural Changes to a Boosted Model

> **Scope note.** Employer, team and product names, live performance figures, and internal code
> are excluded. What follows is the problem definition, the methodology, and the reasoning behind
> the choices I made. The presentation this is drawn from is not included in the repository.

---

## Context

The work was improving a model that forecasts equity returns. It split into two tracks.

- **Feature engineering** — building new explanatory variables, or finding more out of the
  several hundred that already existed.
- **Algorithm modification** — the train/validate/test design, the hyperparameters, and the
  structure of the model itself.

Forecasting is not the whole job. Turning a forecast into actual orders brings its own
constraints back in — liquidity, and the effect your own bidding has on the market. What follows
is the forecasting side: the two pieces I owned.

---

## 1. A Trend-Consistency Feature

**The problem.** The momentum features already in use — RSI, MFI, CCI and others — mostly measure
*how much* a price moved. But two names that rose by the same amount should not be treated the
same if one climbed steadily and the other lurched. Nothing measured that difference directly.

**Definition.** Fit a trend line to the price path over a window, and use the **R² of that fit**
as the consistency of the trend. A high value means the price stayed close to its own trend line
over that window.

One thing to watch: the value is **direction-blind**. A name that fell steadily scores as high as
one that rose steadily. So it is not used on its own — it gets a sign by being multiplied with the
past return, or it is used to group and rank names.

**Design axes.** One definition, several layers of choice. Three axes, so the number of variants
is their product.

| Axis | Choices |
| :--- | :--- |
| Fit | Ordinary linear regression / low-degree polynomial (when the trend curves) |
| Price series | Raw price / moving average (noise removed) / adjusted price (principal components removed) |
| Lookback | Short to long |

**Validation.** A new feature does not go into use because it exists. It gets a time-series
analysis first.

- **Rolling correlation against the existing features** — to check it is not an existing feature
  rebuilt under a new name.
- **Rolling correlation against future returns** — read as a heatmap and a histogram. A
  full-period average hides a sign that flips from one regime to the next.

*Rolling* is the point in both. A correlation measured once, at one point in time, can be an
accident.

**Implementation.** An R script taking a year as its argument: pull two years of data over SQL,
define internal functions for the various R² values, then apply them over moving one-year windows
with a fast-rolling function. A Python equivalent as well (pandas with sklearn). Years are
independent, so a shell script parallelizes across them.

---

## 2. Structural Changes to Gradient Boosting

### Train / validate / test on a time series

The usual split partitions the data at random. On a time series that amounts to seeing the future
and then predicting the past. The design is **walk-forward** instead: slide a
`[train | validate | test]` window forward along the time axis, pick the best model on each
validation segment, and test it on the segment that follows.

- **Training.** Find `f` with `f(X_train) ≈ y_train`, where `X` is the features and `y` is the
  future return. XGBoost builds trees in sequence, each one correcting the error accumulated so
  far. The knobs are the number of trees, the maximum depth of each, the loss function.
- **Validation.** This is where overfitting is held off. Rather than use the full set of trees,
  stop early — the same role the number of epochs plays in a feedforward network. And the
  **choice of metric here is the definition of "better."** L² distance, L¹ distance, correlation
  `f(X)·y / |f(X)||y|`, sign accuracy — each selects a different model. L² if avoiding a large
  loss matters most; sign accuracy if only the direction has to be right.
- **Test.** Take the `f₁, f₂, …` chosen under the different metrics, combine them linearly, and
  measure the combination's error on the test segment.

### Structural modification

Changing the model itself. Two things.

**Grouping names and handling the groups separately.** Group by a performance characteristic —
past return, or the trend consistency above. Three designs, compared against each other; each one
down the list has less freedom and less room to overfit.

1. Train a separate XGBoost per group — the most freedom, and the most overfitting risk.
2. Train separately, but hold the number of trees equal across groups.
3. Train together, and vary only the number of trees each group uses.

**Using trees selectively.** In boosting, the early trees carry the broad structure and the later
ones fit the residual. Taking a slice rather than the whole set (say trees 80 through 100), or
weighting trees differently, is another knob.

**Implementation.** Modifying the XGBoost code in Python directly: extending the yaml
configuration with new parameters such as the group name, then finding and changing the relevant
loops in a codebase several thousand lines long. Locating the place to cut took longer than the
cut.

---

## 3. Telling an Improvement from an Overfit

Algorithms get updated as time passes — hyperparameters change, structure changes. The difficulty
is that **a reduction in error is not by itself evidence of an improvement.** The changes were
made while looking at the data accumulated so far, so some part of that reduction is just a fit
to the past.

So the check is this: take the proposed change X and **apply it retroactively to an older version
of the algorithm.** Had X been in place at that point, would performance over the period that
followed actually have been better? That evaluates the change on the terms available then, not on
the data it was tuned against.

With that check in place, "this change reduced the error" and "this change holds up when you move
the point in time" become visibly different claims.

---

## What Carried Forward

| From this period | Where it went |
| :--- | :--- |
| Choosing the evaluation metric *is* defining "better" | [01 — Model Selection & Evaluation Methodology](../wildfire-uav/01-model-selection.md) |
| Validation design comes before the model | [01 — Model Selection & Evaluation Methodology](../wildfire-uav/01-model-selection.md) |
| Splitting data so it does not leak | [02 — Data Pipeline & Annotation](../wildfire-uav/02-data-pipeline.md) |

---

[← Back to overview](../README.md)
