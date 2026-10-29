---
name: differential-gene-expression-analysis
description: Use when rare cells detected from a large scRNA-seq dataset (e.g. 4238
  FiRE-detected PBMC cells) have been clustered and specific clusters must be biologically
  characterized and assigned to known reference sub-types.
when_to_use_negative:
- Input expression profiles have not been through FiRE rareness scoring and dropClust
  clustering — without FiRE-detected rare cells there are no rare clusters to characterize.
- The goal is de novo discovery of cell sub-types rather than assigning clusters to
  an existing reference signature set such as the Villani dendritic sub-types.
- The target population is an abundant cell type; FiRE scoring is designed for minor
  subpopulations that require profiling several thousands of cells to capture.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: FiRE
  role: Upstream algorithm whose IQR-thresholded rareness scores define the 4238 rare
    PBMC cells that are clustered and differentially characterized
  repo: https://github.com/princethewinner/FiRE
- name: dropClust
  role: Clusters the FiRE-detected rare cells into 13 clusters (R1–R13) with no special
    treatment to outliers, providing the partitions that are then differentially expressed
- name: Python
  role: Environment in which FiRE was installed and run to score rareness and produce
    the rare-cell set used for differential analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:83eacd9d13f66c96a465ddcfe4a187836abc58ca78fc468e81808fa6b2eeac2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princethewinner/FiRE
  tools:
  - FiRE
  - Python
  - dropClust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# differential-gene-expression-analysis

## Summary

Characterizes FiRE-detected rare dendritic cell clusters from ~68k human PBMC single-cell expression profiles by differential gene expression, overlaying cluster marker genes with Villani dendritic sub-type reference signatures to resolve DC1, DC3, DC4 and DC6 sub-types.

## When to use

Use when rare cells detected from a large scRNA-seq dataset (e.g. 4238 FiRE-detected PBMC cells) have been clustered and specific clusters must be biologically characterized and assigned to known reference sub-types.

## When NOT to use

- Input expression profiles have not been through FiRE rareness scoring and dropClust clustering — without FiRE-detected rare cells there are no rare clusters to characterize.
- The goal is de novo discovery of cell sub-types rather than assigning clusters to an existing reference signature set such as the Villani dendritic sub-types.
- The target population is an abundant cell type; FiRE scoring is designed for minor subpopulations that require profiling several thousands of cells to capture.

## Inputs

- FiRE-detected rare cell set (4238 rare PBMC cells with rareness scores above the IQR threshold q3 + 1.5*iqr)
- ~68k human PBMC single-cell expression profiles from the upstream FiRE rareness-scoring step
- dropClust cluster labels R1–R13 for the rare cells
- Villani dendritic cell sub-type reference gene signatures (six reported sub-types)

## Outputs

- List of dendritic-exclusive clusters (R4, R8, R9, R13)
- Differential gene expression results per dendritic-exclusive cluster
- Cluster-to-sub-type assignment table written to dendritic_subtype_assignments.csv (resolves DC1, DC3, DC4, DC6)

## How to apply

Start from the FiRE-detected rare cells (4238 cells from the ~68k PBMC scRNA-seq profiles) clustered with dropClust into 13 clusters labeled R1–R13; note that dropClust administers no special treatment to outliers, so outlier cells may be submerged into minor cell clusters. Inspect cluster composition to identify the clusters populated exclusively by dendritic cells (R4, R8, R9 and R13). Run differential gene expression analysis between the dendritic-exclusive clusters and the Villani dendritic sub-type reference signatures. Overlay the cluster-specific differential genes with the reported Villani genes and assign each cluster to its best-matching Villani sub-type following the Fig. 7 mapping (DC1, DC3, DC4 or DC6). Write the cluster-to-sub-type assignment table to dendritic_subtype_assignments.csv.

## Related tools

- **FiRE** (Upstream algorithm whose IQR-thresholded rareness scores define the 4238 rare PBMC cells that are clustered and differentially characterized) — https://github.com/princethewinner/FiRE
- **dropClust** (Clusters the FiRE-detected rare cells into 13 clusters (R1–R13) with no special treatment to outliers, providing the partitions that are then differentially expressed)
- **Python** (Environment in which FiRE was installed and run to score rareness and produce the rare-cell set used for differential analysis)

## Evaluation signals

- Exactly four clusters out of the dropClust partition (R4, R8, R9, R13) are populated exclusively by dendritic cells.
- Differential genes of the dendritic-exclusive clusters overlap the Villani reference signatures sufficiently to confidently resolve at least 4 of the 6 reported sub-types (DC1, DC3, DC4, DC6).
- Each cluster-to-sub-type assignment is consistent with the Fig. 7 mapping between FiRE/dropClust clusters and Villani sub-types.
- The output dendritic_subtype_assignments.csv contains one sub-type assignment for each of the four dendritic-exclusive clusters.
- Upstream rare-cell selection follows the FiRE IQR criterion (score >= q3 + 1.5*iqr), yielding ~4238 rare cells from the ~68k profiles.

## Limitations

- FiRE does not discriminate between an outlier and cells representing minor cell types, and since dropClust administers no special treatment to outliers, outlier cells may get submerged into the minor cell clusters and confound cluster composition.
- Only 4 of the 6 Villani-reported dendritic sub-types could be confidently resolved; two sub-types remain unresolvable with this rare-cell clustering.
- Doublet rate may vary between 1% and 30%, which does not necessarily comply with the notion of cell rareness, limiting FiRE's application to doublet detection and thus the purity of the rare-cell input.

## Evidence

- [intro] We also demonstrated the efficacy of FiRE in delineating human blood dendritic cell sub-types using ~68 k single-cell expression profiles of human blood cells.: "We also demonstrated the efficacy of FiRE in delineating human blood dendritic cell sub-types using ~68 k single-cell expression profiles of human blood cells."
- [discussion] For clustering FiRE detected rare cells, in all our analyses, we adhered to dropClust24 that does not administer any special treatment to outliers.: "For clustering FiRE detected rare cells, in all our analyses, we adhered to dropClust24 that does not administer any special treatment to outliers."
- [discussion] FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types.: "FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types."
- [readme] Higher values of FiRE score represent rare cells.: [evidence span withheld — anchor status: external_doc]
