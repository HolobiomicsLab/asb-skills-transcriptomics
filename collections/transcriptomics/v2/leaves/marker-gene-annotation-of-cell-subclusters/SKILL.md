---
name: marker-gene-annotation-of-cell-subclusters
description: Use when re-clustered scRNA-seq subpopulations must be biologically labeled
  by signature genes and their purity quantified with a quantitative statistic such
  as ROGUE before being trusted as subtypes.
when_to_use_negative:
- When the goal is to detect rare cell types within a large population rather than
  verify purity of existing subclusters — GiniClust/GiniClust2 or RaceID3 are designed
  for rare-type detection, not purity assessment
- When clusters contain continuously varying trajectories or low-quality data where
  the fixed 0.9 threshold is inappropriate — the authors note the threshold must be
  determined from global ROGUE values in such cases
- When measuring purity of integrated cell populations from different protocols and
  platforms — the authors state current ROGUE does not yet support this
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ROGUE
  role: Entropy-based statistic (Ratio of Global Unshifted Entropy) used to quantify
    the purity/homogeneity of each identified subcluster via CalculateRogue
  repo: https://github.com/PaulingLiu/ROGUE
- name: ROGUE (R package)
  role: Open-source R implementation providing matr.filter, entropy_fit, DetermineK,
    and CalculateRogue functions for the ROGUE-guided workflow
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: R dependency package that must be installed before ROGUE; used for data manipulation
    in the annotation workflow
- name: devtools
  role: R package used to install ROGUE from GitHub via install_github
- name: R
  role: Runtime environment for the ROGUE package and marker-gene/ROGUE computation
    scripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:e9191492286add52f27a5f94781060cb9b40fbb5670b21697a4ce43474d1ec72
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# marker-gene-annotation-of-cell-subclusters

## Summary

Annotates re-clustered single-cell subpopulations by checking per-cluster marker gene signatures, then verifies each cluster is a pure subtype using the ROGUE entropy statistic. In the ROGUE-guided B-cell analysis, five of seven subclusters passed the ROGUE > 0.92 purity bar while doublet-like and germinal-center clusters failed.

## When to use

Use when re-clustered scRNA-seq subpopulations must be biologically labeled by signature genes and their purity quantified with a quantitative statistic such as ROGUE before being trusted as subtypes.

## When NOT to use

- When the goal is to detect rare cell types within a large population rather than verify purity of existing subclusters — GiniClust/GiniClust2 or RaceID3 are designed for rare-type detection, not purity assessment
- When clusters contain continuously varying trajectories or low-quality data where the fixed 0.9 threshold is inappropriate — the authors note the threshold must be determined from global ROGUE values in such cases
- When measuring purity of integrated cell populations from different protocols and platforms — the authors state current ROGUE does not yet support this

## Inputs

- scRNA-seq gene-by-cell expression count matrix (e.g., 4291 tumor-infiltrating B cells from GEO accession GSE131907)
- Cluster assignments from ROGUE-guided re-clustering (e.g., seven B-cell subpopulations)
- Prior marker gene signature lists used to identify the cell class (e.g., B-cell markers)

## Outputs

- Per-cluster marker gene lists (e.g., JUNB/FOS, TXNIP, ACTB, FCER2/HVCN1, MX1/IFI6/IFI44L, CD3D, LRMP/RGS13)
- Per-cluster ROGUE purity values (five pure clusters with ROGUE > 0.92; low values for DEs/doublet-like and germinal center clusters)
- Cluster assignment table with marker genes and ROGUE values (e.g., rogue_values.csv)

## How to apply

After ROGUE-guided re-clustering (e.g., using DetermineK to fix the cluster number), compute per-cluster marker genes and check for expected signature sets: JUNB/FOS, TXNIP, ACTB, FCER2/HVCN1, MX1/IFI6/IFI44L for pure B-cell subtypes, CD3D for DEs/doublet-like contamination, and LRMP/RGS13 for germinal center B cells. Quantify each subcluster's homogeneity with CalculateRogue(.x, platform, cutoff, k, features), selecting the platform setting appropriate to the dataset. Accept clusters with ROGUE > 0.92 (the recommended purity threshold) as highly homogeneous subtypes, and treat clusters with low ROGUE — such as the DEs/doublet-like and germinal center clusters — as impure candidates requiring further re-clustering rather than as definitive subtypes. Write cluster assignments, marker gene lists, and ROGUE values to an output file such as rogue_values.csv.

## Related tools

- **ROGUE** (Entropy-based statistic (Ratio of Global Unshifted Entropy) used to quantify the purity/homogeneity of each identified subcluster via CalculateRogue) — https://github.com/PaulingLiu/ROGUE
- **ROGUE (R package)** (Open-source R implementation providing matr.filter, entropy_fit, DetermineK, and CalculateRogue functions for the ROGUE-guided workflow) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (R dependency package that must be installed before ROGUE; used for data manipulation in the annotation workflow)
- **devtools** (R package used to install ROGUE from GitHub via install_github)
- **R** (Runtime environment for the ROGUE package and marker-gene/ROGUE computation scripts)

## Examples

```
devtools::install_github("PaulingLiu/ROGUE"); rogue <- CalculateRogue(expr, platform = "UMI", k = 7)
```

## Evaluation signals

- Five of seven B-cell subclusters show ROGUE values > 0.92, confirming high homogeneity
- DEs/doublet-like clusters show CD3D signature and germinal center clusters show LRMP/RGS13; both have low ROGUE values as expected for impure populations
- Pre-clustering expression matrix passes matr.filter(expr, min.cells, min.genes) retaining adequately expressed genes before annotation
- DetermineK selects the optimal cluster number (k = 7 for the GSE131907 B-cell data) before re-clustering and marker annotation
- Output rogue_values.csv contains cluster assignments, marker gene lists, and ROGUE values consistent with the expected seven-cluster partition

## Limitations

- The recommended 0.9 purity threshold may not be suitable for low-quality or continuous data, where it must be determined from global ROGUE values
- Current ROGUE does not yet assess purity of integrated cell populations from different protocols and platforms
- Roles of specific annotated subclusters (e.g., B_C02_ACTB cells in immune suppression) can remain biologically undefined even after successful annotation and purity quantification

## Evidence

- [intro] ROGUE statistic quantifies purity: "we propose the Ratio of Global Unshifted Entropy (ROGUE) statistic to quantify the purity or homogeneity of a given single-cell population while accounting for other technical factors"
- [methods] CalculateRogue signature: "CalculateRogue(.x, platform, cutoff, k, features)"
- [discussion] Re-clustering rationale: "By quantifying cluster purity with ROGUE before and after clustering or re-clustering, we were able to detect low-purity clusters and perform further analysis to discover pure subtypes"
- [readme] ROGUE concept from README: "Here, we proposed the concept of cluster purity and introduced a conceptually novel statistic, named ROGUE, to examine whether a given cluster is a pure cell population."
- [methods] Filtering step: "matr.filter(expr, min.cells, min.genes)"
