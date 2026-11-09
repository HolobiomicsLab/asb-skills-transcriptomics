---
name: poisson-gamma-negative-binomial-statistics
description: Use when you need to quantify the purity/homogeneity of an scRNA-seq
  cell cluster by fitting a per-gene entropy model on a gene-by-cell UMI count matrix
  and computing a ROGUE score.
when_to_use_negative:
- You need to assess purity of integrated cell populations from different protocols/platforms
  — the paper notes this is not yet supported and proposed as future work
- The data are low-quality or continuous such that the recommended 0.9 purity threshold
  does not apply without recalibrating against global ROGUE values
- The goal is rare cell-type detection/clustering rather than purity assessment —
  tools like GiniClust or RaceID3 address clustering, not entropy-based purity scoring
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: ROGUE
  role: Open-source R package implementing Entropy, SE_fun, entropy_fit, DetermineK,
    matr.filter, and CalculateRogue for computing the ROGUE purity score
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: Required dependency to install before ROGUE
- name: devtools
  role: Used to install ROGUE from GitHub via devtools::install_github("PaulingLiu/ROGUE")
- name: R
  role: Runtime environment in which the entropy model and ROGUE statistic are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:974c9703d541800305e790fc87e0422fa86ef55782822185109f89224f6bb321
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
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# poisson-gamma-negative-binomial-statistics

## Summary

Implements the S–E (shifted entropy) model of the ROGUE framework: single-gene sample entropy is computed under a Poisson-Gamma/negative-binomial model (S = ln E(X) + a), a LOESS fit of ln E(X) yields per-gene entropy-reduction residuals (ds), and these aggregate into the purity statistic ROGUE = 1 - sum(ds)/(sum(ds) + K).

## When to use

Use when you need to quantify the purity/homogeneity of an scRNA-seq cell cluster by fitting a per-gene entropy model on a gene-by-cell UMI count matrix and computing a ROGUE score.

## When NOT to use

- You need to assess purity of integrated cell populations from different protocols/platforms — the paper notes this is not yet supported and proposed as future work
- The data are low-quality or continuous such that the recommended 0.9 purity threshold does not apply without recalibrating against global ROGUE values
- The goal is rare cell-type detection/clustering rather than purity assessment — tools like GiniClust or RaceID3 address clustering, not entropy-based purity scoring

## Inputs

- Gene-by-cell raw UMI count expression matrix (scRNA-seq), e.g. csv format with genes as rows and cell IDs as columns
- Cell labels / cluster assignments for the population being scored
- Sample IDs and platform annotation ('droplet' or 'full-length')
- LOESS span, multiple-testing method (mt.method), and adjustment flag (if.adj) parameters

## Outputs

- Per-gene entropy-reduction statistic ds (residuals from the LOESS entropy fit)
- Estimated K constant (e.g. 45 for droplet, 500 for full-length platforms)
- ROGUE purity score per cell population/cluster in [0, 1], with 1 indicating a completely pure population

## How to apply

Start with a raw gene-by-cell expression (UMI count) matrix and optionally filter it with matr.filter(expr, min.cells, min.genes). Implement Entropy(expr, r) so that, under the Poisson-Gamma/negative-binomial model, the shifted entropy of each gene is S = ln E(X) + a. Then run SE_fun/entropy_fit(.x, span, mt.method, if.adj), which fits a tunable-span LOESS regression of ln E(X) against average-expression-derived entropy and returns each gene's residual entropy reduction (ds), with multiple-testing adjustment controlled by mt.method/if.adj. Estimate the platform-specific constant with DetermineK(expr, span, r, mt.method, if.adj) — use K=45 for droplet data and K=500 for full-length data — and aggregate the per-gene ds values with CalculateRogue(.x, platform, cutoff, k, features) using ROGUE = 1 - sum(ds)/(sum(ds) + K), honoring the cutoff on retained genes/features. A population with no significant ds for any gene receives ROGUE = 1, indicating a completely pure subtype or state; lower values indicate impurity (a 0.9 threshold is suggested, but must be chosen from global ROGUE values for low-quality or continuous data).

## Related tools

- **ROGUE** (Open-source R package implementing Entropy, SE_fun, entropy_fit, DetermineK, matr.filter, and CalculateRogue for computing the ROGUE purity score) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (Required dependency to install before ROGUE)
- **devtools** (Used to install ROGUE from GitHub via devtools::install_github("PaulingLiu/ROGUE"))
- **R** (Runtime environment in which the entropy model and ROGUE statistic are implemented)

## Examples

```
install.packages("tidyverse"); devtools::install_github("PaulingLiu/ROGUE"); CalculateRogue(expr, platform="droplet", cutoff=0.9)
```

## Evaluation signals

- A population with no significant ds for all genes must return ROGUE = 1 (completely pure subtype/state)
- ROGUE values computed by the module should numerically match the reference ROGUE R package output on the same expression matrix
- DetermineK should recover the platform-appropriate constant (K=45 for droplet, K=500 for full-length data)
- ROGUE should decrease when applied to known impure/mixed clusters and increase after re-clustering into pure subtypes
- matr.filter should reduce the matrix to genes/cells passing min.cells/min.genes before entropy fitting

## Limitations

- The 0.9 purity threshold may not suit low-quality or continuous data; the threshold should be determined from global ROGUE values
- ROGUE does not yet assess purity of integrated cell populations from different protocols and platforms
- Platform choice matters: K defaults to 45 for droplet and 500 for full-length protocols, so mis-annotating the platform skews scores

## Evidence

- [methods] ROGUE = 1 - sum(ds)/(sum(ds) + K): [evidence span withheld — anchor status: not_found]
- [results] Within the S–E model, a cell population with no significant ds for all genes receives a ROGUE value of 1, indicating it is a completely pure subtype or state.: [evidence span withheld — anchor status: not_found]
- [abstract] we present an entropy-based statistic, ROGUE, to accurately quantify the purity of identified cell clusters.: "Here we present an entropy-based statistic, ROGUE, to accurately quantify the purity of identified cell clusters."
- [methods] devtools::install_github("PaulingLiu/ROGUE"): "To install ROGUE, run:
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools")
devtools::install_github("PaulingLiu/ROGUE")"
- [discussion] But for low-quality data or continuous data, the threshold could be determined by considering the global ROGUE values: "But for low-quality data or continuous data, the threshold could be determined by considering the global ROGUE values"
