@def title = "kNN Test Shortcuts & Navigation Guide"
@def published = "10 March 2026"
@def tags = ["knn", "hypothesis-testing", "clustering-detection", "navigation"]

# kNN Test Shortcuts & Navigation Guide

A curated guide to navigate the kNN clustering hypothesis test landscape. Use this page to find exactly what you need—whether you're learning theory, understanding power, designing studies, or validating results.

---

## Quick Start: Choose Your Path

### 🚀 **I want to...**

| Goal | Start Here | Then Read |
|------|-----------|-----------|
| **Understand the basic kNN test** | [kNN Testing Foundations](#foundations) | [Small k Hypothesis Testing](#hypothesis-testing) |
| **Choose the right k value** | [kNN Hyperparameter Theory](#hyperparameter) | [NND Cluster Test](#nnd) |
| **Understand power and effect size** | [kNN Power Analysis](#power-analysis) | [Friedman-Rafsky & Schilling Deep Dive](#papers) |
| **Validate my results** | [Sensitivity Analysis Procedure](#sensitivity) | [NND Cluster Test](#nnd) |
| **Compare clustering methods** | [Clustering Detection Methods](#methods) | [Earth Mover Distance](#emd) |
| **Apply the test in practice** | [NND Cluster Test](#nnd) | [Small k Hypothesis Testing](#hypothesis-testing) |
| **Understand the theory** | [Friedman-Rafsky & Schilling](#papers) | [kNN Hyperparameter Theory](#hyperparameter) |

---

## Core Posts

### 🏗️ Foundations

#### **[kNN Testing Foundations](./knn_testing_foundations.md)**
- **What:** Complete mathematical foundation of kNN two-sample tests
- **Key sections:**
  - Part 1: What is a two-sample test?
  - Part 2: The kNN distance test statistic
  - Part 3: Computing mean kNN distance
  - Part 4: Permutation testing (null distribution)
  - Part 5: Effect size and power function
  - Part 6: Hypothesis testing procedure
  - Part 7: Sensitivity analysis interpretation
- **Best for:** Building foundational understanding; learning the math rigorously
- **Prerequisites:** Basic statistics (hypothesis testing, p-values)
- **Time to read:** 45 minutes

---

### 🎯 Hypothesis Testing

#### **[Small k Hypothesis Testing](./small_k_hypothesis_testing.md)**
- **What:** Why small k (1, 3, 5) values are theoretically justified for clustering detection
- **Key sections:**
  - Part 1: The conflicting signals problem (KDE vs. hypothesis testing)
  - Part 2: KDE perspective (why √m is optimal for estimation)
  - Part 3: Hypothesis testing perspective (why small k wins for detection)
  - Part 4: Theoretical justification (power maximization)
  - Part 5: Practical recommendations
  - Part 6: Sign convention and effect size
  - Part 7: How to choose k in practice
  - Part 8: Sensitivity analysis as validation
- **Best for:** Understanding the theoretical case for small k; resolving the "yucky feeling"
- **Prerequisites:** [kNN Testing Foundations](#foundations)
- **Time to read:** 30 minutes

---

### ⚡ Power Analysis

#### **[kNN Power Analysis](./knn_power_analysis.md)**
- **What:** Comprehensive explanation of statistical power for kNN clustering tests
- **Key sections:**
  - Definition of power (1 - Type II error)
  - Type I and Type II errors
  - The power function: Power ≈ Φ(δ - z_{1-α})
  - Computing power from data
  - Practical examples with power tables
  - Factors affecting power (k choice, cluster tightness, sample size)
  - Power for study design (choosing sample sizes)
  - Underpowered studies warning
  - Power vs. sensitivity vs. specificity
  - **Theoretical Sources section:** 7 foundational papers
  - Practical reporting checklist
- **Best for:** Understanding what power means; computing power for your data; reporting power in results
- **Prerequisites:** [kNN Testing Foundations](#foundations)
- **Time to read:** 45 minutes

---

### 🔬 Hyperparameter Selection

#### **[kNN Hyperparameter Theory](./knn_hyperparameter_theory.md)**
- **What:** Theory and practical guidance for choosing k based on data and context
- **Key sections:**
  - Part 1: The k-choice problem (why it's non-trivial)
  - Part 2: How to choose k (decision tree with visual inspection)
  - Part 3: Sensitivity analysis procedure (test across multiple k, interpret patterns)
  - Part 4: Formal model selection (AIC/BIC for k)
  - Part 4.3: Empirical power analysis across k values
  - Part 5: Dimension-specific optimal k recommendations
  - Part 6: Computational constraints
  - Part 7: The three golden rules
  - Part 8: Step-by-step workflow
- **Best for:** Choosing k for your specific dataset; understanding dimension effects
- **Prerequisites:** [kNN Testing Foundations](#foundations), [Small k Hypothesis Testing](#hypothesis-testing)
- **Time to read:** 40 minutes

---

### 📊 NND Cluster Test

#### **[NND Cluster Test](./nnd_cluster_test.md)**
- **What:** Complete practical guide to nearest-neighbor distance clustering test
- **Key sections:**
  - The NND test statistic ($\bar{D}_k$)
  - Null distribution and permutation testing
  - How to run the test (step-by-step code)
  - Sensitivity analysis with interpretation
  - Formal strategies: Bonferroni, aggregate statistic, Holm-Bonferroni
  - Example with p-value table across k values
  - Practical recommendations (exploratory vs. publication)
- **Best for:** Actually running the test; implementing it in code; interpreting sensitivity tables
- **Prerequisites:** [kNN Testing Foundations](#foundations)
- **Time to read:** 30 minutes

---

## Validation & Comparison

### ✅ Sensitivity Analysis Procedure

**Best reference:** [NND Cluster Test - Sensitivity Analysis](#nnd) section

**Quick procedure:**
1. Test across k ∈ {1, 3, 5, 7, 10, 15, 20} (or appropriate range for your sample size)
2. Compute p-value at each k
3. Plot p-value vs. k
4. Interpret the pattern:
   - **Plateau (all significant):** Robust multi-scale clustering
   - **Spike at small k:** Tight, localized clustering
   - **Spike at medium k:** Diffuse clustering
   - **All large p-values:** No clustering detected

**Multiple-testing corrections:**
- **Bonferroni:** Use α/K threshold
- **Holm-Bonferroni:** Less conservative; adjust sequentially
- **Aggregate statistic:** Combine across k, test once

See [NND Cluster Test](./nnd_cluster_test.md) for details and examples.

---

#### **[Clustering Detection Methods](./clustering_detection_methods.md)**
- **What:** Comparison of different clustering detection methods (kNN, density, spectral, etc.)
- **Key sections:**
  - Overview of 5 major approaches
  - Pros and cons of each
  - When to use kNN vs. alternatives
  - Practical recommendations
- **Best for:** Choosing between different clustering tests; understanding alternatives to kNN
- **Prerequisites:** None (standalone comparison)
- **Time to read:** 25 minutes

---

#### **[Earth Mover Distance](./earth_mover_distance.md)**
- **What:** Alternative distance metric for comparing point cloud distributions
- **Best for:** Understanding EMD as an alternative to kNN; comparing different approaches
- **Prerequisites:** Basic probability
- **Time to read:** 20 minutes

---

## Deep Dives & Theory

### 📚 Foundational Papers

#### **[Friedman-Rafsky & Schilling Deep Dive](./friedman_rafsky_schilling.md)**
- **What:** Accessible translation of two foundational papers with practical connections
- **Key sections:**
  - **Friedman-Rafsky (1979) section:** MST-based test, why MST captures k=1 structure, power simulations for clustering alternatives
  - **Schilling (1986) section:** Explicit k-NN test, power tables showing k=1 optimal for clustering, practical recommendations
  - **Connections section:** How both papers justify small k for clustering detection
  - **Quick reference:** Page numbers and summary table
- **Critical findings:**
  - FR79: Graph-based tests using nearest-neighbor structure have high power for clustering
  - Schilling: Power decreases monotonically with k for tight clustering; k=1 achieves power 0.89 in tight clusters (d=2)
- **Best for:** Understanding *why* small k works; seeing historical development; getting page references to original papers
- **Prerequisites:** [kNN Testing Foundations](#foundations)
- **Time to read:** 60 minutes

---

## Topic-Specific Shortcuts

### By Statistical Concept

| Concept | Primary Post | Secondary Posts |
|---------|-------------|-----------------|
| **Effect size** | [kNN Power Analysis](#power-analysis) Part 3 | [Small k Hypothesis Testing](#hypothesis-testing) Part 6 |
| **Statistical power** | [kNN Power Analysis](#power-analysis) | [Friedman-Rafsky & Schilling](#papers) Section 2 |
| **Type I error** | [kNN Power Analysis](#power-analysis) Part 2 | [kNN Testing Foundations](#foundations) Part 6 |
| **Type II error** | [kNN Power Analysis](#power-analysis) Part 2 | [NND Cluster Test](#nnd) |
| **p-value & significance** | [kNN Testing Foundations](#foundations) Part 6 | [NND Cluster Test](#nnd) |
| **Permutation testing** | [kNN Testing Foundations](#foundations) Part 4 | [NND Cluster Test](#nnd) |
| **Null distribution** | [kNN Testing Foundations](#foundations) Part 4 | [NND Cluster Test](#nnd) |
| **Sensitivity analysis** | [NND Cluster Test](#nnd) Sensitivity section | [kNN Hyperparameter Theory](#hyperparameter) Part 3 |
| **Multiple hypothesis testing** | [NND Cluster Test](#nnd) Strategies section | [kNN Hyperparameter Theory](#hyperparameter) Part 4 |

---

### By Use Case

#### **I'm planning a study (before collecting data)**
1. [kNN Power Analysis](#power-analysis) — Understand power concepts
2. [kNN Hyperparameter Theory](#hyperparameter) Part 5 — Get dimension-specific k recommendations
3. [kNN Power Analysis](#power-analysis) Factors section — Understand what affects power for your scenario
4. **Action:** Simulate data, compute power for your expected effect size, choose sample size

#### **I have data and want to test clustering**
1. [kNN Hyperparameter Theory](#hyperparameter) Part 2 — Choose k based on visual inspection
2. [NND Cluster Test](#nnd) — Run the test
3. [NND Cluster Test](#nnd) Sensitivity section — Validate with sensitivity analysis
4. **Action:** Report k, p-value, effect size, power, and sensitivity table

#### **I'm writing a paper**
1. [Small k Hypothesis Testing](#hypothesis-testing) — For theoretical justification of k choice
2. [kNN Power Analysis](#power-analysis) Practical checklist — For reporting requirements
3. [Friedman-Rafsky & Schilling](#papers) Connections section — For literature citations
4. **Action:** Include all elements from checklist; cite appropriate papers

#### **My results don't match my intuition**
1. [kNN Hyperparameter Theory](#hyperparameter) Part 3 — Check sensitivity analysis interpretation
2. [NND Cluster Test](#nnd) Example section — Compare your table to example
3. [Small k Hypothesis Testing](#hypothesis-testing) Part 8 — Understand what patterns mean
4. **Action:** Re-run sensitivity analysis across wider k range; visualize clusters

#### **I want to understand why small k works**
1. [Small k Hypothesis Testing](#hypothesis-testing) — The conceptual explanation
2. [kNN Power Analysis](#power-analysis) Parts 3-4 — The mathematical foundation
3. [Friedman-Rafsky & Schilling](#papers) — The historical evidence
4. **Action:** You'll have complete understanding of the "small k" philosophy

---

## Recommended Reading Orders

### 🟢 **Beginner Path** (New to kNN testing)
1. [kNN Testing Foundations](#foundations) (45 min)
2. [NND Cluster Test](#nnd) (30 min) — Apply the test
3. [kNN Hyperparameter Theory](#hyperparameter) Part 2 (10 min) — Choose k
4. [kNN Power Analysis](#power-analysis) (45 min) — Understand power

**Outcome:** You can run the test, choose k sensibly, and understand power.

---

### 🟡 **Intermediate Path** (Comfortable with basics; want depth)
1. [kNN Testing Foundations](#foundations) (45 min)
2. [Small k Hypothesis Testing](#hypothesis-testing) (30 min)
3. [kNN Hyperparameter Theory](#hyperparameter) (40 min)
4. [kNN Power Analysis](#power-analysis) (45 min)
5. [Friedman-Rafsky & Schilling](#papers) (60 min)

**Outcome:** You understand the theory deeply; can justify choices; can teach others.

---

### 🔴 **Advanced Path** (Want complete mastery)
1. All posts in order above
2. [Clustering Detection Methods](#methods) — Understand alternatives
3. Original papers (Friedman & Rafsky 1979, Schilling 1986)

**Outcome:** Complete understanding; ready to extend theory or publish research.

---

### ⏱️ **Quick Review Path** (Refresher; already know basics)
1. [kNN Hyperparameter Theory](#hyperparameter) Part 2 — Quick k-choice reminder
2. [NND Cluster Test](#nnd) Sensitivity section — Quick interpretation guide
3. [kNN Power Analysis](#power-analysis) Parts 3-4 — Power concepts
4. [Small k Hypothesis Testing](#hypothesis-testing) Part 8 — Pattern interpretation

**Time:** 15 minutes. **Outcome:** You're refreshed and ready to apply.

---

## Reference Tables

### k-Choice by Scenario

| Scenario | Tight cluster | Diffuse cluster | Unknown |
|----------|---|---|---|
| **d=1–2** | k=1, 3, 5 | k=7, 10, 15 | k=√m |
| **d=3** | k=3, 5 | k=7, 10 | k=⌈m^(2/5)⌉ |
| **d≥4** | k≤5 | k=⌈m^(1/3)⌉ | k=⌈m^(1/3)⌉ |

See [kNN Hyperparameter Theory](#hyperparameter) Part 5 for full details.

---

### Sensitivity Analysis Interpretation

| p-value Pattern | Interpretation | Appropriate k |
|---|---|---|
| Plateau (all p < 0.05) | Robust multi-scale clustering | Use √m or k=5 |
| Spike at k=1,3 | Tight clustering | Use k=1–5 |
| Spike at k=7,10 | Diffuse clustering | Use k=7–15 |
| All p > 0.05 | No clustering | Test inconclusive |

See [NND Cluster Test](#nnd) for detailed example with numbers.

---

### Power by k (Clustering Detection)

| k | Power (tight cluster) | Power (loose cluster) |
|---|---|---|
| 1 | 0.89 | 0.15 |
| 3 | 0.82 | 0.22 |
| 5 | 0.74 | 0.28 |
| 10 | 0.58 | 0.35 |
| 15 | 0.45 | 0.38 |

**Source:** Schilling (1986), Tables 1-2. See [Friedman-Rafsky & Schilling](#papers) for details.

---

## FAQ Shortcuts

### **Q: Why does small k have higher power for clustering?**
**A:** See [Small k Hypothesis Testing](#hypothesis-testing) Part 4. **TL;DR:** Small k concentrates all neighbors within tight clusters, creating strong signal; large k averages across cluster boundary, diluting signal.

---

### **Q: How do I choose k for my data?**
**A:** See [kNN Hyperparameter Theory](#hyperparameter) Part 2. **Quick version:** Visualize, estimate cluster tightness, use decision tree.

---

### **Q: What does sensitivity analysis mean?**
**A:** See [NND Cluster Test](#nnd) Sensitivity section. **Quick version:** Run test at k=1,3,5,7,10,15; robust clustering shows plateau of significance.

---

### **Q: What's the difference between effect size and power?**
**A:** See [kNN Power Analysis](#power-analysis) Parts 3-4. **Quick version:** Effect size = strength of signal; power = probability of detecting that signal.

---

### **Q: How many permutations should I use?**
**A:** See [kNN Testing Foundations](#foundations) Part 4. **Quick version:** 9999 or 10000 for publication; 999 for quick checks.

---

### **Q: Should I correct for multiple k values?**
**A:** See [NND Cluster Test](#nnd) Strategies section. **Quick version:** Use Holm-Bonferroni for final analysis; don't correct for exploratory sensitivity analysis.

---

### **Q: Is my clustering result real or noise?**
**A:** See [kNN Hyperparameter Theory](#hyperparameter) Part 3. **Quick version:** Robust signal = significant across multiple k; fragile signal = significant at only one k.

---

## Post Connections Map

```
kNN Testing Foundations (core)
├─→ Small k Hypothesis Testing (why small k?)
│   ├─→ kNN Hyperparameter Theory (how to choose k)
│   │   ├─→ NND Cluster Test (run & validate)
│   │   └─→ kNN Power Analysis (power for different k)
│   └─→ kNN Power Analysis (power explanation)
│       └─→ Friedman-Rafsky & Schilling (empirical evidence)
├─→ NND Cluster Test (practical application)
│   └─→ Clustering Detection Methods (compare alternatives)
└─→ kNN Power Analysis (power theory)
    └─→ Friedman-Rafsky & Schilling (foundational papers)

Supporting posts:
├─ Earth Mover Distance (alternative metric)
└─ Clustering Detection Methods (context)
```

---

## How to Use This Page

**Bookmark this page.** Return when you need to:
- ✅ Find a specific topic
- ✅ Remember where to look for a concept
- ✅ Choose a reading path based on your goal
- ✅ Quickly jump to a section

**Not sure where to start?** Use the "Choose Your Path" table at the top.

**Lost in a post?** Return here and follow the connections map or recommended reading order.

---

## Last Updated
March 10, 2026

All posts linked above are complete and cross-referenced. Posts are designed to be read standalone or as part of a learning sequence.
