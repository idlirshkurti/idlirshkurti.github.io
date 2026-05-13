---
layout: page
title: Causal Inference
parent: Projects
nav_order: 10
description: Bayesian Causal AI - From Correlation to Counterfactuals with Uncertainty Quantification
tags: [causal inference, bayesian, pymc, causalpy, counterfactuals, uncertainty quantification, python, statistics]
math: mathjax3
---

# Bayesian Causal AI
{:.no_toc}

*From Correlation to Counterfactuals with Uncertainty Quantification*

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 1. Executive Summary

Traditional machine learning excels at **prediction** (mapping \\( X \to Y \\) based on correlations). However, it often fails at **intervention** (predicting what happens to \\( Y \\) if we actively change \\( X \\)).

**Bayesian Causal AI** combines three powerful paradigms:

- **Causal Inference**: To identify true mechanisms and remove confounding bias.
- **Bayesian Statistics**: To quantify uncertainty and incorporate domain knowledge (priors).
- **Generative AI/Modeling**: To simulate counterfactual realities ("What would have happened if...?").

This document outlines the theory, the workflow, and a practical implementation using the Python ecosystem (`PyMC`, `CausalPy`).

---

## 2. The Conceptual Shift

### 2.1 The Ladder of Causation

To understand where Bayesian Causal AI fits, we must revisit Pearl's three levels of cognitive ability:

- **L1: Association (Seeing)**: "What does a symptom tell me about a disease?" *(Standard ML/Bayesian Networks)*
- **L2: Intervention (Doing)**: "If I take aspirin, will my headache go away?" *(A/B Testing, Causal Inference)*
- **L3: Counterfactuals (Imagining)**: "My headache is gone, but would it have gone away had I not taken aspirin?" *(Causal AI)*

> **Key Insight for Data Scientists**: Most supervised learning models stay at L1. They learn \\( P(Y \mid X) \\). Causal AI aims to learn \\( P(Y \mid do(X)) \\), where \\( do(X) \\) represents an active intervention that breaks the dependence of \\( X \\) on its parents (confounders).

### 2.2 Why Bayesian?

In causal inference, we often deal with small sample sizes, observational data, or unobserved confounders. A frequentist point estimate of an Average Treatment Effect (ATE) might mislead us about risk.

- **Uncertainty Quantification**: We get a full posterior distribution of the causal effect, not just a single number.
- **Priors as Domain Knowledge**: We can encode constraints (e.g., *"Price elasticity cannot be positive"*) directly into the model structure.

---

## 3. The Architecture: DAGs and Structural Models

### 3.1 The Directed Acyclic Graph (DAG)

The DAG is your **map**. It represents your assumptions about the data-generating process.

| Node Type | Definition | Why it Matters |
|-----------|-----------|----------------|
| **Confounder (Z)** | Causes both Treatment (T) and Outcome (Y) | Must be controlled for to remove bias |
| **Collider** | Caused by both T and Y | Controlling for it *opens* a non-causal path (selection bias) |
| **Mediator** | A step in the chain \\( T \to M \to Y \\) | Controlling for it removes the effect you are trying to measure |

### 3.2 Bayesian Structural Equation Models (BSEM)

Instead of just drawing arrows, we define the mathematical relationship for each node.

For a simple structure \\( Z \to T \to Y \\) and \\( Z \to Y \\):

$$
Z \sim \text{Normal}(0, 1)
$$

$$
T = \alpha_Z Z + \epsilon_T
$$

$$
Y = \beta_T T + \beta_Z Z + \epsilon_Y
$$

In a Bayesian framework, \\( \alpha \\) and \\( \beta \\) are **distributions**, not fixed scalars.

---

## 4. Practical Implementation: Synthetic Control with CausalPy

Let's define a scenario common in industry: **Marketing Campaign Impact**.

- **Scenario**: We launched a marketing campaign in a specific region at time \\( t = 100 \\).
- **Goal**: Estimate the causal impact of the campaign on Sales.
- **Challenge**: We cannot A/B test regions easily. We must use the pre-intervention period to learn the relationship between our region and a control region (or other covariates).

### 4.1 The Code

We will use `CausalPy`, a library built on top of `PyMC` that abstracts standard causal designs.

```python
import arviz as az
import causalpy as cp
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd

# ---------------------------------------------------------
# 1. Generate Synthetic Data
# ---------------------------------------------------------
np.random.seed(42)
n_time_points = 150
treatment_time = 100

# Predictor variables (e.g., competitors, organic trends)
x1 = np.sin(np.linspace(0, 4 * np.pi, n_time_points)) + np.random.normal(0, 0.1, n_time_points)
x2 = np.cos(np.linspace(0, 4 * np.pi, n_time_points)) + np.random.normal(0, 0.1, n_time_points)

# True outcome (Sales) without treatment
y_pre = 2.5 * x1 + 1.2 * x2 + np.random.normal(0, 0.2, n_time_points)

# Add a Causal Effect (Lift) after t=100
# The effect gradually increases then decays
effect = np.zeros(n_time_points)
effect[treatment_time:] = (
    np.linspace(0, 2, n_time_points - treatment_time)
    * np.exp(-np.linspace(0, 1, n_time_points - treatment_time))
)
y = y_pre + effect

df = pd.DataFrame({'x1': x1, 'x2': x2, 'y': y, 'time': range(n_time_points)})
treatment_idx = df.index >= treatment_time

# ---------------------------------------------------------
# 2. Bayesian Causal Model (Synthetic Control / Interrupted Time Series)
# ---------------------------------------------------------
result = cp.ExperimentalDesign(
    data=df,
    treatment_time=treatment_time,
    formula="y ~ 1 + x1 + x2",
    model=cp.pymc_models.WeightedSumFitter(
        sample_kwargs={"draws": 2000, "target_accept": 0.95}
    ),
)

# ---------------------------------------------------------
# 3. Visualization of Causal Impact
# ---------------------------------------------------------
fig, ax = result.plot()
plt.suptitle("Bayesian Causal Impact Analysis", fontsize=16)
plt.show()

# Extracting the Bayesian Credible Intervals for the cumulative impact
summary = az.summary(result.idata.posterior, var_names=["causal_impact_cumulative"])
print(summary)
```

### 4.2 Explanation of the Plots

Running the code above generates a comprehensive **Causal Impact dashboard**. Here is what the plots represent:

**Observed vs. Counterfactual (Top Panel)**
- *Black Line*: Actual observed data (\\( y \\)).
- *Blue Line*: The counterfactual prediction — what would have happened *without* the campaign. Generated by the Bayesian model trained only on pre-intervention data.
- *Shaded Region*: The Bayesian Uncertainty Interval (usually 94% HDI). If the black line falls outside this blue shaded area, the effect is statistically significant.

**Causal Effect (Middle Panel)**

This plots the difference: \\( y_{\text{observed}} - y_{\text{counterfactual}} \\). It shows the **lift** at every time point — you can see the effect ramp up and potentially decay.

**Cumulative Effect (Bottom Panel)**

Sums the daily causal effects over time. This answers the business question: *"What was the total incremental revenue generated by this campaign?"* with a credible interval (e.g., *"We generated $50k ± $5k"*).

---

## 5. Advanced Topic: The "Do-Operator" in PyMC

For more complex DAGs (not just time series), we explicitly model the intervention using `pm.do`.

**Scenario**: We want to know if raising prices (\\( P \\)) increases profit (\\( Y \\)), but Income (\\( I \\)) confounds both.

**DAG**: \\( I \to P \\), \\( I \to Y \\), \\( P \to Y \\).

```python
import pymc as pm

with pm.Model() as model:
    # Priors (The "Bayesian" part)
    income = pm.Normal("income", 0, 1)

    # Price depends on Income (High income areas tolerate high prices)
    price = pm.Normal("price", mu=0.5 * income, sigma=0.5)

    # Sales depends on Price AND Income (Confounder)
    # True causal effect of price is -1.0
    sales = pm.Normal("sales", mu=2.0 * income - 1.0 * price, sigma=0.5)

    # Inference
    idata = pm.sample()

# ---------------------------------------------------------
# The "Do" Operator (Intervention)
# ---------------------------------------------------------
# We want to simulate P(Sales | do(Price=2.0))
# This breaks the link Income -> Price.

with model:
    # We replace the 'price' variable with a fixed value, breaking its parents
    idata_do = pm.do(model, {"price": 2.0}).sample_posterior_predictive(
        idata, var_names=["sales", "income"]
    )
```

> **Why this matters**: If you simply filtered the dataframe for rows where `Price == 2.0`, you would get a **biased estimate** because those rows usually correspond to high-income customers (who buy more anyway). The `do` operator mathematically simulates the intervention, giving the **true causal effect**.

---

## 6. Conclusion

Bayesian Causal AI moves us from passive observers to active reasoners.

1. **Define the Structure**: Use domain knowledge to draw the DAG.
2. **Model the Process**: Use Bayesian priors to capture relationships and uncertainty.
3. **Intervene**: Use do-calculus or counterfactual forecasting to simulate decisions.

### Recommended Tooling Stack

| Purpose | Libraries |
|---------|-----------|
| **Modelling** | `PyMC`, `NumPyro`, `Stan` |
| **Causal Wrappers** | `CausalPy` (Quasi-experiments), `DoWhy` (identification logic), `CausalNex` (structure learning) |
| **Visualization** | `ArviZ`, `Daft` (DAG rendering) |
