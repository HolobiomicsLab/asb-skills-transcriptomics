---
name: tabula-muris-dataset-reproduction
description: Use when reproducing or validating cluster-purity benchmarks on the Tabula
  Muris brain dataset, or when testing whether ROGUE-guided re-clustering resolves
  impure oligodendrocyte/neuron clusters into pure subtypes.
when_to_use_negative:
- Input data is low-quality or continuous, where the recommended 0.9 purity threshold
  is not appropriate and the cutoff must instead be determined from global ROGUE values
- Goal is to assess purity of cell populations integrated across different protocols
  or platforms, which ROGUE does not yet support (proposed future work)
- Data is not a UMI-based scRNA-seq count matrix with cell/sample metadata (e.g.,
  qPCR or pre-normalized feature tables), which the entropy-fit pipeline is not designed
  for
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: ROGUE
  role: Computes the entropy-based ROGUE (Ratio of Global Unshifted Entropy) cluster
    purity statistic; provides matr.filter, entropy_fit, DetermineK, Entropy, CalculateRogue,
    and the rogue() entry point used for per-class and per-subtype purity evaluation
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: R dependency package that must be installed before ROGUE and used for data
    manipulation of the Tabula Muris count matrix
- name: devtools
  role: Installs the ROGUE R package from GitHub via install_github
- name: R
  role: Runtime environment in which the reproduction scripts and ROGUE pipeline are
    executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:d5c5acf8c1cc54f5a48cda705b759be42478d852751256ae4430ae7b963f2282
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - R
  - ROGUE
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tabula-muris-dataset-reproduction

## Summary

Reproduce the ROGUE purity analysis of the Tabula Muris brain scRNA-seq dataset (GSE109774), showing oligodendrocytes and neurons have ROGUE < 0.8 while other brain classes reach ~0.9-1, and perform ROGUE-guided re-clustering of the 3401 oligodendrocytes into ten pure subtypes.

## When to use

Use when reproducing or validating cluster-purity benchmarks on the Tabula Muris brain dataset, or when testing whether ROGUE-guided re-clustering resolves impure oligodendrocyte/neuron clusters into pure subtypes.

## When NOT to use

- Input data is low-quality or continuous, where the recommended 0.9 purity threshold is not appropriate and the cutoff must instead be determined from global ROGUE values
- Goal is to assess purity of cell populations integrated across different protocols or platforms, which ROGUE does not yet support (proposed future work)
- Data is not a UMI-based scRNA-seq count matrix with cell/sample metadata (e.g., qPCR or pre-normalized feature tables), which the entropy-fit pipeline is not designed for

## Inputs

- Tabula Muris brain scRNA-seq count matrix (GEO accession GSE109774)
- Per-cell class/cluster labels for the brain dataset
- Sample identifiers and platform annotation (e.g., 10X or inDrop) for the rogue() call

## Outputs

- Per-cell-class ROGUE values showing oligodendrocyte and neuron ROGUE < 0.8 and other classes ~0.9-1
- Ten re-clustered oligodendrocyte subtypes with per-subtype ROGUE values, nine of which attain ROGUE ~0.9-1 (cluster 6 excepted)
- Results table reproducing Fig. 5a-d patterns

## How to apply

Download the Tabula Muris brain scRNA-seq count matrix from GEO (GSE109774) and load it into R. Install tidyverse and ROGUE from GitHub with devtools::install_github("PaulingLiu/ROGUE"). Preprocess with matr.filter(expr, min.cells, min.genes), fit the expression entropy model with entropy_fit and DetermineK, then call rogue(expr, labels, samples, platform, k, min.cell.n, remove.outlier.n, span, r, mt.method, filter, min.cells, min.genes) per brain cell class to confirm oligodendrocytes and neurons score < 0.8 while other classes reach ~0.9-1. Extract the 3401 oligodendrocytes, re-cluster them, and recompute ROGUE per subtype with the same entropy-fit/CalculateRogue pipeline. Assemble per-class and per-subtype ROGUE tables and compare against the repository's manuscript scripts (./scripts) to validate the reproduction, expecting nine of ten oligodendrocyte subtypes at ROGUE ~0.9-1 (Fig. 5a-d).

## Related tools

- **ROGUE** (Computes the entropy-based ROGUE (Ratio of Global Unshifted Entropy) cluster purity statistic; provides matr.filter, entropy_fit, DetermineK, Entropy, CalculateRogue, and the rogue() entry point used for per-class and per-subtype purity evaluation) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (R dependency package that must be installed before ROGUE and used for data manipulation of the Tabula Muris count matrix)
- **devtools** (Installs the ROGUE R package from GitHub via install_github)
- **R** (Runtime environment in which the reproduction scripts and ROGUE pipeline are executed)

## Examples

```
devtools::install_github("PaulingLiu/ROGUE"); rogue(expr, labels, samples, platform, k, min.cell.n, remove.outlier.n, span, r, mt.method, filter, min.cells, min.genes)
```

## Evaluation signals

- Oligodendrocyte and neuron classes return ROGUE < 0.8 while other brain cell classes reach ROGUE ~0.9-1
- Re-clustering of the 3401 oligodendrocytes yields exactly ten refined subtypes
- Nine of the ten oligodendrocyte subtypes (all except cluster 6) attain ROGUE ~0.9-1
- Computed ROGUE values match the reference quantitative results produced by the repository's ./scripts manuscript scripts
- DetermineK completes entropy-model fitting without errors for each cluster before ROGUE is computed

## Limitations

- Cluster 6 of the re-clustered oligodendrocytes does not reach ROGUE ~0.9-1, so not every subtype resolves to purity
- The recommended 0.9 purity threshold may be unsuitable for low-quality or continuous data; the threshold should be determined by considering global ROGUE values in such cases
- ROGUE currently does not assess purity of integrated cell populations generated under different protocols and platforms (noted as future work)

## Evidence

- [other] ROGUE-guided re-clustering of the 3401 oligodendrocytes identified ten refined subtypes, and except cluster 6, all nine other clusters attained ROGUE values of ~0.9-1: [evidence span withheld — anchor status: not_found]
- [methods] introduced a conceptually novel statistic, named ROGUE, to examine whether a given cluster is a pure cell population: "introduced a conceptually novel statistic, named ROGUE, to examine whether a given cluster is a pure cell population"
- [methods] devtools::install_github("PaulingLiu/ROGUE"): "devtools::install_github("PaulingLiu/ROGUE")"
- [readme] Before installing ROGUE, the “tidyverse” package should be installed first: "Before installing ROGUE, the “tidyverse” package should be installed first"
- [readme] The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts): "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)"
