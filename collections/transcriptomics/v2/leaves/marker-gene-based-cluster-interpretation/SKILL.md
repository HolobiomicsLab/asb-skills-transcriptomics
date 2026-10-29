---
name: marker-gene-based-cluster-interpretation
description: Use when a re-clustering or clustering partition of scRNA-seq data (e.g.
  low-purity fibroblast clusters split by ROGUE) needs biological labels, and marker
  genes distinguishing subtypes such as myCAFs or antigen-presenting CAFs can be evaluated
  from the expression matrix.
when_to_use_negative:
- Clusters whose ROGUE purity is below the data-appropriate threshold and have no
  coherent marker signature — re-cluster further before annotating
- Low-quality or continuous datasets where the 0.9 purity threshold does not apply
  and thresholds must be set from global ROGUE values
- Integrated populations from different protocols/platforms, which ROGUE does not
  yet assess for purity
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ROGUE
  role: Computes cluster purity (CalculateRogue) and guides re-clustering so that
    marker-gene labels are applied to purer subtypes; R package installed via devtools
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: R dependency required before installing ROGUE, used for data manipulation
    of the annotation table
- name: devtools
  role: Used to install the ROGUE package from GitHub (devtools::install_github)
- name: R
  role: Runtime environment in which ROGUE, tidyverse and the annotation workflow
    execute
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:11d1f79eb5236d8f28fd981d2a06075e45c78b3e87329fa66c1b8429fa9ce812
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - R
  - ROGUE
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# marker-gene-based-cluster-interpretation

## Summary

Annotate single-cell clusters by their signature marker genes after ROGUE-guided re-clustering, e.g. labeling lung-tumor CAF subtypes as myCAF (RGS5/ACTA2/GPX3) or apCAF (CD74/MHC-II) to translate a purer clustering partition into biological subtypes.

## When to use

Use when a re-clustering or clustering partition of scRNA-seq data (e.g. low-purity fibroblast clusters split by ROGUE) needs biological labels, and marker genes distinguishing subtypes such as myCAFs or antigen-presenting CAFs can be evaluated from the expression matrix.

## When NOT to use

- Clusters whose ROGUE purity is below the data-appropriate threshold and have no coherent marker signature — re-cluster further before annotating
- Low-quality or continuous datasets where the 0.9 purity threshold does not apply and thresholds must be set from global ROGUE values
- Integrated populations from different protocols/platforms, which ROGUE does not yet assess for purity

## Inputs

- scRNA-seq count matrix (gene x cell, e.g. Kim et al. lung-tumor stromal data) with cluster/subtype assignments from re-clustering
- Normalized expression or Seurat/negative-binomial-regression object with batch-corrected embedding for marker expression checks
- Per-cluster ROGUE values computed with ROGUE's CalculateRogue on the candidate partition

## Outputs

- Annotated cluster/subtype table mapping cluster IDs to marker-defined labels (e.g. BF_C01_RGS5 -> myCAF, apCAF via CD74/MHC-II), saved as caf_rogue_subtypes.csv
- Per-cluster ROGUE purity values for the annotated partition
- Marker expression evidence (e.g. CD74 and MHC class-II expression) supporting each annotation

## How to apply

After ROGUE-guided re-clustering yields a higher-purity partition (e.g. 7 low-ROGUE lung-tumor fibroblast clusters re-clustered into 11 subtypes with higher average ROGUE), compute per-cluster expression of canonical markers and assign subtype labels by co-expression patterns: myCAFs by RGS5/ACTA2/GPX3, antigen-presenting CAFs by high CD74 and MHC class-II gene expression. Validate each label against the ROGUE values computed with CalculateRogue on the new partition — e.g. the original myCAF cluster AF_C02_COL4A1 (ROGUE = 0.81) split into BF_C01_RGS5 (0.84), BF_C02_ACTA2 (0.87), and BF_C03_GPX3 (0.94), so each labeled subtype should hold a purity near or above the recommended threshold or the global ROGUE level. Save the cluster-to-marker-to-ROGUE annotation table (e.g. caf_rogue_subtypes.csv) so labels are traceable to quantitative purity evidence rather than manual signature-gene inspection alone, which the paper calls arbitrary and imprecise.

## Related tools

- **ROGUE** (Computes cluster purity (CalculateRogue) and guides re-clustering so that marker-gene labels are applied to purer subtypes; R package installed via devtools) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (R dependency required before installing ROGUE, used for data manipulation of the annotation table)
- **devtools** (Used to install the ROGUE package from GitHub (devtools::install_github))
- **R** (Runtime environment in which ROGUE, tidyverse and the annotation workflow execute)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- Re-clustering yields 11 subtypes from the original 7 lung-tumor fibroblast clusters with a higher average ROGUE
- The original myCAF cluster AF_C02_COL4A1 (ROGUE = 0.81) splits into BF_C01_RGS5 (~0.84), BF_C02_ACTA2 (~0.87), and BF_C03_GPX3 (~0.94)
- An apCAF cluster is identified by high CD74 and MHC class-II gene expression
- The output table caf_rogue_subtypes.csv contains one marker-defined label plus a ROGUE value per subtype
- Annotation is consistent between the marker set (RGS5/ACTA2/GPX3 for myCAF; CD74/MHC-II for apCAF) and the cluster naming

## Limitations

- Cluster identification by manually checking signature genes is arbitrary and imprecise on its own, so annotations should be paired with quantitative ROGUE purity values
- The recommended 0.9 purity threshold may not suit low-quality or continuous data; thresholds must be set from global ROGUE values
- ROGUE does not yet assess purity of integrated cell populations from different protocols and platforms
- The role of apCAFs in the lung-cancer microenvironment remains unclear and requires further investigation

## Evidence

- [abstract] Applying ROGUE to fibroblast, B cell and brain data identified additional subtypes: "Applying this metric to fibroblast, B cell and brain data, we identify additional subtypes"
- [discussion] Re-clustering low-purity clusters to discover pure subtypes: "By quantifying cluster purity with ROGUE before and after clustering or re-clustering, we were able to detect low-purity clusters and perform further analysis to discover pure subtypes"
- [other] Marker annotation validated by ROGUE purity: myCAF split into RGS5/ACTA2/GPX3 subtypes and apCAF by CD74/MHC-II: [evidence span withheld — anchor status: not_found]
- [intro] Manual signature-gene checking is arbitrary, motivating quantitative purity assessment: "motivating the need to accurately assess the purity or quality of identified clusters"
- [readme] ROGUE examines whether a given cluster is a pure cell population: "we proposed the concept of cluster purity and introduced a conceptually novel statistic, named ROGUE, to examine whether a given cluster is a pure cell population"
