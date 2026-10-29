---
name: rogue-guided-cell-reclustering
description: Use when a scRNA-seq cluster's purity is questionable (per-cluster ROGUE
  < ~0.8) and pure subtypes must be resolved by re-clustering that subpopulation,
  or when validating cluster homogeneity against a 0.9 ROGUE purity threshold.
when_to_use_negative:
- Data are low-quality or continuous-trajectory in nature, where the recommended 0.9
  purity threshold is unsuitable and thresholds must instead be derived from global
  ROGUE values
- The goal is purity assessment of integrated cell populations from different protocols/platforms,
  which ROGUE does not yet support (proposed as future work)
- The aim is de novo rare cell-type discovery rather than purity assessment/re-clustering
  of known clusters, where methods like GiniClust or RaceID3 are the appropriate alternatives
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: ROGUE
  role: Entropy-based statistic and R package implementing matr.filter, entropy_fit,
    DetermineK, CalculateRogue and the rogue() entry point for quantifying cluster
    purity and guiding re-clustering
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: R dependency that must be installed before installing ROGUE
- name: devtools
  role: R package used to install ROGUE from GitHub via devtools::install_github('PaulingLiu/ROGUE')
- name: R
  role: Runtime environment for the ROGUE package and reproduction scripts in ./scripts
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:127d838d9bf6cfc4307181ba5068f10d6ded7b094396db07501010418feaa1ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - R
  - ROGUE
  - ROGUE (R package)
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# ROGUE-guided cell re-clustering

## Summary

A workflow for detecting low-purity cell clusters with the entropy-based ROGUE statistic (Ratio of Global Unshifted Entropy) and re-clustering those subpopulations to resolve homogeneous subtypes, e.g. re-clustering 3401 oligodendrocytes into ten subtypes with ROGUE ~0.9–1 or B cells into seven clusters.

## When to use

Use when a scRNA-seq cluster's purity is questionable (per-cluster ROGUE < ~0.8) and pure subtypes must be resolved by re-clustering that subpopulation, or when validating cluster homogeneity against a 0.9 ROGUE purity threshold.

## When NOT to use

- Data are low-quality or continuous-trajectory in nature, where the recommended 0.9 purity threshold is unsuitable and thresholds must instead be derived from global ROGUE values
- The goal is purity assessment of integrated cell populations from different protocols/platforms, which ROGUE does not yet support (proposed as future work)
- The aim is de novo rare cell-type discovery rather than purity assessment/re-clustering of known clusters, where methods like GiniClust or RaceID3 are the appropriate alternatives

## Inputs

- Raw or filtered scRNA-seq count matrix (gene x cell), e.g. Tabula Muris brain from GEO GSE109774 or tumor-infiltrating B cells from GEO GSE131907
- Cell type/cluster labels per cell and sample/batch identifiers
- Platform designation (e.g. UMI vs non-UMI) passed to rogue()/CalculateRogue()

## Outputs

- Per-cluster and per-subtype ROGUE purity values (e.g. nine of ten oligodendrocyte subtypes at ROGUE ~0.9–1; five of seven B-cell clusters > 0.92)
- Re-clustered subpopulation assignments and per-cluster marker gene lists (e.g. rogue_values.csv)
- Entropy model fit (entropy_fit) and optimal cluster number k (DetermineK)

## How to apply

First quantify purity of every cluster with the ROGUE R package: install tidyverse and devtools::install_github('PaulingLiu/ROGUE'), preprocess the expression matrix with matr.filter(expr, min.cells, min.genes), fit the mean-variance entropy model with entropy_fit(.x, span, mt.method), select the optimal cluster number with DetermineK(expr, span, r, mt.method, if.adj), and compute per-cluster ROGUE via rogue(expr, labels, samples, platform, ...) or CalculateRogue(.x, platform, cutoff, k, features). Decision point: clusters with ROGUE below ~0.8–0.9 (e.g. oligodendrocytes and neurons in the Tabula Muris brain data while other classes reach ~0.9–1) are candidates for re-clustering. Extract the low-purity subpopulation (e.g. the 3401 oligodendrocytes from GSE109774, or 4291 tumor-infiltrating B cells from GSE131907), repeat the entropy-fit/DetermineK/CalculateRogue pipeline on this subset to re-cluster it, and check expected marker signatures (e.g. JUNB/FOS, TXNIP, ACTB, FCER2/HVCN1, MX1/IFI6/IFI44L for B-cell clusters; CD3D for doublet-like, LRMP/RGS13 for germinal center). Collect per-subtype ROGUE values into a results table and compare against the repository's ./scripts reference outputs to validate the reproduction.

## Related tools

- **ROGUE** (Entropy-based statistic and R package implementing matr.filter, entropy_fit, DetermineK, CalculateRogue and the rogue() entry point for quantifying cluster purity and guiding re-clustering) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (R dependency that must be installed before installing ROGUE)
- **devtools** (R package used to install ROGUE from GitHub via devtools::install_github('PaulingLiu/ROGUE'))
- **R** (Runtime environment for the ROGUE package and reproduction scripts in ./scripts)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE"); rogue(expr, labels, samples, platform, k, min.cell.n, remove.outlier.n, span, r, mt.method, filter, min.cells, min.genes)
```

## Evaluation signals

- Per-class ROGUE pattern reproduced: oligodendrocytes and neurons ROGUE < 0.8 while other Tabula Muris brain classes reach ~0.9–1
- After re-clustering the 3401 oligodendrocytes, ten subtypes are recovered and nine of ten (all except cluster 6) attain ROGUE ~0.9–1
- For B cells, five of seven re-clustered subpopulations show ROGUE > 0.92 while DEs/doublet-like (CD3D+) and germinal center (LRMP/RGS13+) clusters show low ROGUE
- Computed ROGUE values match the reference quantitative results generated by the repository's ./scripts manuscript scripts
- Marker signatures appear in the expected clusters (JUNB/FOS, TXNIP, ACTB, FCER2/HVCN1, MX1/IFI6/IFI44L for the pure B-cell clusters)

## Limitations

- The recommended 0.9 purity threshold may not be suitable for low-quality or continuous data; the threshold should be determined from global ROGUE values in those cases
- Current ROGUE does not assess purity of integrated cell populations from different protocols and platforms (proposed as future work)
- Not every re-clustered subpopulation reaches purity — e.g. oligodendrocyte cluster 6 and the B-cell DEs/doublet-like and germinal center clusters retain low ROGUE

## Evidence

- [results] ROGUE-guided re-clustering of the 3401 oligodendrocytes identified ten refined subtypes, and except cluster 6, all nine other clusters attained ROGUE values of ~0.9–1, indicating purity.: [evidence span withheld — anchor status: not_found]
- [discussion] Quantify cluster purity with ROGUE before and after clustering or re-clustering to detect low-purity clusters and discover pure subtypes: "By quantifying cluster purity with ROGUE before and after clustering or re-clustering, we were able to detect low-purity clusters and perform further analysis to discover pure subtypes"
- [methods] devtools::install_github("PaulingLiu/ROGUE"): "devtools::install_github("PaulingLiu/ROGUE")"
- [methods] DetermineK(expr, span, r, mt.method, if.adj): "DetermineK(expr, span, r, mt.method, if.adj)"
- [discussion] For low-quality data or continuous data, the threshold could be determined by considering the global ROGUE values: "But for low-quality data or continuous data, the threshold could be determined by considering the global ROGUE values"
- [readme] The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts): "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
