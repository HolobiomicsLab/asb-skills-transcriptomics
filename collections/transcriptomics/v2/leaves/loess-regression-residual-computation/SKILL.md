---
name: loess-regression-residual-computation
description: Use when computing per-gene entropy-reduction statistics (ds) from a
  gene-by-cell scRNA-seq count matrix to feed the ROGUE cluster-purity calculation,
  or when modeling the expected mean-expression vs. sample-entropy relationship.
when_to_use_negative:
- When the goal is clustering or rare-cell-type detection rather than purity assessment
  — tools like GiniClust, GiniClust2 or RaceID3 are designed for that; ROGUE quantifies
  homogeneity of an already-defined cluster.
- When assessing purity of cell populations integrated across different protocols/platforms
  — the paper states current ROGUE does not yet handle integrated populations (proposed
  future work).
- 'Input is not raw count data (e.g., log-transformed expression): GiniClust''s README
  explicitly warns log2-transformed RNA-seq data may not work for related count-based
  models; the S–E entropy model likewise expects counts under the Poisson-Gamma framework.'
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: ROGUE
  role: R package providing Entropy(), SE_fun(), entropy_fit(), DetermineK(), CalculateRogue()
    implementing the LOESS residual ds computation and ROGUE aggregation
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: required dependency package installed before ROGUE
- name: devtools
  role: used to install ROGUE from GitHub via devtools::install_github()
- name: R
  role: runtime environment for the ROGUE implementation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:bcb8ad856a37e30240420a5b5f3d4a1931f4c345a2b05ef75dfa2e570a6566f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - R
  - ROGUE
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# loess-regression-residual-computation

## Summary

Fit a LOESS regression of ln E(X) against average expression within the S–E entropy model and take per-gene residuals as the entropy-reduction statistic (ds), which feeds the ROGUE purity score. This is the core inferential step of the ROGUE (Ratio of Global Unshifted Entropy) method for quantifying single-cell cluster purity.

## When to use

Use when computing per-gene entropy-reduction statistics (ds) from a gene-by-cell scRNA-seq count matrix to feed the ROGUE cluster-purity calculation, or when modeling the expected mean-expression vs. sample-entropy relationship.

## When NOT to use

- When the goal is clustering or rare-cell-type detection rather than purity assessment — tools like GiniClust, GiniClust2 or RaceID3 are designed for that; ROGUE quantifies homogeneity of an already-defined cluster.
- When assessing purity of cell populations integrated across different protocols/platforms — the paper states current ROGUE does not yet handle integrated populations (proposed future work).
- Input is not raw count data (e.g., log-transformed expression): GiniClust's README explicitly warns log2-transformed RNA-seq data may not work for related count-based models; the S–E entropy model likewise expects counts under the Poisson-Gamma framework.

## Inputs

- gene-by-cell raw UMI count matrix (scRNA-seq expression matrix) from droplet or full-length platform
- cell label / cluster assignment vector (to compute ds within a given cell population)
- platform annotation ('droplet' or 'full-length')
- LOESS span parameter, r parameter for the entropy model, mt.method for multiple-testing adjustment, if.adj flag, and gene cutoff

## Outputs

- per-gene entropy-reduction statistic ds (LOESS residuals of ln E(X) vs. mean-expression entropy)
- fitted LOESS mean-entropy curve (entropy_fit output)
- estimated K constant from DetermineK
- scalar ROGUE = 1 - sum(ds)/(sum(ds) + K) purity score per cell population/cluster

## How to apply

First compute single-gene sample entropy with Entropy(expr, r) under the Poisson-Gamma (negative-binomial) model so that the shifted entropy is S = ln E(X) + a. Then, for each gene, fit a LOESS regression (SE_fun / entropy_fit with a tunable span) of ln E(X) against the average-expression-derived entropy across the mean-entropy curve; the per-gene residual of this fit is the ds statistic quantifying how much observed entropy falls below the expectation for that expression level. Control false discovery with multiple-testing adjustment via mt.method and if.adj, and retain genes passing the cutoff before aggregation. Finally, DetermineK(expr, span, r, mt.method, if.adj) estimates the platform-specific K constant (K=45 for 'droplet', K=500 for 'full-length'), and CalculateRogue(.x, platform, cutoff, k, features) combines the retained ds values as ROGUE = 1 - sum(ds)/(sum(ds) + K). The rationale is that a pure population shows no significant entropy reduction beyond what mean expression predicts, so residuals (ds) near zero for all genes yield ROGUE ≈ 1.

## Related tools

- **ROGUE** (R package providing Entropy(), SE_fun(), entropy_fit(), DetermineK(), CalculateRogue() implementing the LOESS residual ds computation and ROGUE aggregation) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (required dependency package installed before ROGUE)
- **devtools** (used to install ROGUE from GitHub via devtools::install_github())
- **R** (runtime environment for the ROGUE implementation)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- A completely pure population should yield no significant ds for all genes and ROGUE = 1 (per the S–E model finding).
- Validate the implementation by running it on a test scRNA-seq matrix with cell labels, sample ids, and platform annotation and verifying outputs numerically match the reference ROGUE R package on the same data.
- Check that K takes the documented platform-specific values: K=45 when platform='droplet' and K=500 when platform='full-length'.
- Verify the multiple-testing adjustment respects mt.method/if.adj settings and that the gene cutoff correctly filters retained features before aggregation.
- Compare computed ds values against the fitted LOESS curve: residuals should reflect genuine entropy reduction, not fitting artifacts (tune span if needed).

## Limitations

- The 0.9 purity threshold may not be suitable for low-quality or continuous data; the threshold should be determined by considering global ROGUE values in those cases.
- Current ROGUE does not assess purity of integrated cell populations from different protocols and platforms; this is left as future work.
- Results depend on the LOESS span and the K constant matching the platform (droplet vs. full-length); mis-specification distorts the residual ds statistic.
- Existing purity approaches (silhouette, DendroSplit, distance ratio) are not comparable among datasets — ROGUE aims to fix this, but cross-method comparisons should not be treated as equivalent.

## Evidence

- [methods] introduced a conceptually novel statistic, named ROGUE, to examine whether a given cluster is a pure cell population: "introduced a conceptually novel statistic, named ROGUE, to examine whether a given cluster is a pure cell population"
- [methods] entropy_fit function signature: "entropy_fit(.x, span, mt.method)"
- [methods] DetermineK function signature: "DetermineK(expr, span, r, mt.method, if.adj)"
- [methods] CalculateRogue function signature: "CalculateRogue(.x, platform, cutoff, k, features)"
- [intro] we propose the Ratio of Global Unshifted Entropy (ROGUE) statistic to quantify the purity or homogeneity of a given single-cell population while accounting for other technical factors: "we propose the Ratio of Global Unshifted Entropy (ROGUE) statistic to quantify the purity or homogeneity of a given single-cell population while accounting for other technical factors"
- [discussion] recommend 0.9 threshold caveat: "But for low-quality data or continuous data, the threshold could be determined by considering the global ROGUE values"
- [discussion] future work on integrated populations: "additional extensions could enable enhanced performance, for example, assessing the purity of integrated cell populations from different protocols and platforms"
- [readme] installation instructions: "devtools::install_github("PaulingLiu/ROGUE")"
