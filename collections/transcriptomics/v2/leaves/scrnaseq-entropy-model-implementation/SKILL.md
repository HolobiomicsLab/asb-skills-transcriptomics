---
name: scrnaseq-entropy-model-implementation
description: Use when implementing or validating an entropy-based cluster purity score
  for scRNA-seq clusters, i.e. computing single-gene sample entropy, LOESS-derived
  ds residuals, and aggregating them into ROGUE.
when_to_use_negative:
- Assessing purity of integrated cell populations from different protocols and platforms
  — the article states current ROGUE does not yet support this (proposed future work)
- Data are log2-transformed or otherwise not raw counts — the model assumes count-level
  expression for the Poisson-Gamma entropy computation
- Low-quality or continuous data where the default 0.9 purity threshold must instead
  be determined from global ROGUE values per the discussion
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ROGUE
  role: Reference R package whose functions (Entropy, entropy_fit, DetermineK, CalculateRogue)
    this implementation re-creates and against which outputs are validated
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: Dependency R package that must be installed before ROGUE
- name: devtools
  role: Used to install the ROGUE reference package from GitHub for validation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:b0b758391db4f89d2122942ca1904e30d93d3414c2c127184c02107b49bf10c1
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5263157894736842
n_claims: 19
---

# scrnaseq-entropy-model-implementation

## Summary

Implementation of the S-E (expression entropy) model underlying the ROGUE statistic, converting per-gene entropy reduction (ds) into a quantitative cluster-purity score for scRNA-seq cell populations (ROGUE = 1 - sum(ds)/(sum(ds) + K)).

## When to use

Use when implementing or validating an entropy-based cluster purity score for scRNA-seq clusters, i.e. computing single-gene sample entropy, LOESS-derived ds residuals, and aggregating them into ROGUE.

## When NOT to use

- Assessing purity of integrated cell populations from different protocols and platforms — the article states current ROGUE does not yet support this (proposed future work)
- Data are log2-transformed or otherwise not raw counts — the model assumes count-level expression for the Poisson-Gamma entropy computation
- Low-quality or continuous data where the default 0.9 purity threshold must instead be determined from global ROGUE values per the discussion

## Inputs

- Gene-by-cell scRNA-seq expression matrix (raw UMI counts) with cell labels and sample ids
- Platform annotation ('droplet' or 'full-length')
- Parameters: span, r, mt.method, if.adj, cutoff, features

## Outputs

- Per-gene entropy-reduction statistic ds (from entropy_fit/SE_fun)
- Estimated K constant (from DetermineK)
- ROGUE purity score per cell population (1 = completely pure subtype; from CalculateRogue)

## How to apply

Implement Entropy(expr, r) in R to compute single-gene sample entropy under the Poisson-Gamma/negative-binomial model such that the shifted entropy is S = ln E(X) + a. Implement SE_fun(expr, span, r, mt.method, if.adj) with entropy_fit(.x, span, mt.method), fitting a LOESS regression of ln E(X) against average-expression-derived entropy and returning the residual entropy-reduction statistic ds per gene, with multiple-testing adjustment controlled by mt.method/if.adj. Implement DetermineK(expr, span, r, mt.method, if.adj) to estimate the K constant, then CalculateRogue(.x, platform, cutoff, k, features) aggregating per-gene ds into ROGUE = 1 - sum(ds)/(sum(ds) + K), using K=45 for platform='droplet' and K=500 for platform='full-length'. Validate the module by running it on a test expression matrix (with cell labels, sample ids, platform annotation) and confirming outputs numerically match the reference ROGUE R package. Filter inputs first with matr.filter(expr, min.cells, min.genes).

## Related tools

- **ROGUE** (Reference R package whose functions (Entropy, entropy_fit, DetermineK, CalculateRogue) this implementation re-creates and against which outputs are validated) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (Dependency R package that must be installed before ROGUE)
- **devtools** (Used to install the ROGUE reference package from GitHub for validation)

## Examples

```
if (!requireNamespace("devtools", quietly=TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE"); CalculateRogue(expr, platform="droplet")
```

## Evaluation signals

- ROGUE output numerically matches the reference ROGUE R package (devtools::install_github('PaulingLiu/ROGUE')) on the same test expression matrix
- A population with no significant ds for all genes yields ROGUE = 1 (pure subtype invariant)
- platform='droplet' yields K=45 and platform='full-length' yields K=500 in the ROGUE formula
- entropy_fit returns a per-gene ds residual for every gene, with multiple-testing adjustment consistent with the supplied mt.method/if.adj
- matr.filter(expr, min.cells, min.genes) prunes the matrix before entropy computation

## Limitations

- ROGUE does not yet assess purity of integrated cell populations from different protocols and platforms (future work)
- The recommended 0.9 purity threshold may not suit low-quality or continuous data; it must be determined from global ROGUE values
- Purity score interpretation depends on honoring the gene cutoff and platform-specific K constant

## Evidence

- [methods] CalculateRogue aggregates per-gene ds: "CalculateRogue(.x, platform, cutoff, k, features)"
- [intro] ROGUE is open-source R package on GitHub: "is implemented in an open-source R package ROGUE"
- [readme] Installation requires tidyverse first: "Before installing ROGUE, the “tidyverse” package should be installed first"
- [abstract] A pure population gets ROGUE of 1: "Here we present an entropy-based statistic, ROGUE, to accurately quantify the purity of identified cell clusters."
