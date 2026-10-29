---
name: clustering-parameter-sweep-evaluation
description: Use when you need to choose or characterize a weighting parameter for
  hybrid clone/transcriptome clustering by measuring chance-corrected agreement (Cohen's
  kappa) between clusters and clonal barcodes across a parameter range.
when_to_use_negative:
- Data lacks clone barcode assignments, so there is no clone membership against which
  to compute kappa (e.g., transcriptome-only Seurat/scanpy clustering)
- The goal is marker identification or gene set overrepresentation of hybrid clusters
  rather than parameter selection/concordance characterization
- Input is full-lineage tracing data (CRISPR-Cas9 genome editing or imaging-based
  family trees), which ClonoCluster was not designed for and would require phylogenetic-distance
  weighting
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: ClonoCluster
  role: Generates hybrid cluster assignments across the alpha sweep, linearly combining
    transcriptome and clone edge weights
  repo: https://github.com/leeprichman/ClonoCluster
- name: cast_confusion
  role: Cross-tabulates cluster assignment versus clone barcode membership at each
    alpha and supports kappa concordance computation
  repo: https://github.com/leeprichman/ClonoCluster
- name: R (version >= 3.6)
  role: Runtime environment; installed via devtools::install_github('leeprichman/ClonoCluster')
  repo: https://github.com/leeprichman/ClonoCluster
- name: data.table
  role: R package dependency for handling assignment tables and CSV output in the
    paper workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:04db7589f937e67d284aa70adb26efe4783a8692a66e7830399b8987ce0daebd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - R (version >= 3.6)
  - R (version >=)
  - cast_confusion
  - data.table
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# clustering-parameter-sweep-evaluation

## Summary

Sweeps a clustering weighting parameter (alpha from 0 to 1 by 0.1 in ClonoCluster) and quantifies how cluster assignments transition toward clone identity using Cohen's kappa against barcode membership, revealing dataset-specific transition sharpness.

## When to use

Use when you need to choose or characterize a weighting parameter for hybrid clone/transcriptome clustering by measuring chance-corrected agreement (Cohen's kappa) between clusters and clonal barcodes across a parameter range.

## When NOT to use

- Data lacks clone barcode assignments, so there is no clone membership against which to compute kappa (e.g., transcriptome-only Seurat/scanpy clustering)
- The goal is marker identification or gene set overrepresentation of hybrid clusters rather than parameter selection/concordance characterization
- Input is full-lineage tracing data (CRISPR-Cas9 genome editing or imaging-based family trees), which ClonoCluster was not designed for and would require phylogenetic-distance weighting

## Inputs

- PCA matrix of transcriptome data (e.g., 25 principal components from a normalized/scaled count matrix via irlba_wrap)
- Two-column barcode assignment table with columns 'rn' (cell ID) and 'Barcode'
- Alpha sweep vector seq(0, 1, by = 0.1) with beta = 0.1 and clustering resolution res = 1

## Outputs

- Per-dataset table of Cohen's kappa versus alpha (11 alpha values from 0 to 1 by 0.1), written as CSV
- Confusion tables (cluster assignment x clone barcode) at each alpha level from cast_confusion
- Kappa-vs-alpha trajectory used to identify gradual versus abrupt (step-like) transitions per dataset

## How to apply

For each dataset, run clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1) on the PCA matrix (e.g., 25 PCs via irlba_wrap) and the two-column barcode table (columns 'rn' and 'Barcode'). At each of the 11 alpha values, cross-tabulate hybrid cluster assignment versus clone barcode membership with the cast_confusion function and compute Cohen's kappa as the chance-corrected agreement statistic from each confusion table. Assemble kappa versus alpha into a per-dataset table (CSV) and inspect the trajectory: alpha = 0 gives purely transcriptome weights (kappa ~ 0) and alpha = 1 purely clone weights (kappa approaching 1). Rationale: a graded rise indicates meaningful intermediate hybrid clusters, whereas an abrupt jump (as seen for the cardiomyocyte-directed iPSC dataset from Jiang et al.) indicates few meaningful intermediate states, informing the choice of a tolerable alpha.

## Related tools

- **ClonoCluster** (Generates hybrid cluster assignments across the alpha sweep, linearly combining transcriptome and clone edge weights) — https://github.com/leeprichman/ClonoCluster
- **cast_confusion** (Cross-tabulates cluster assignment versus clone barcode membership at each alpha and supports kappa concordance computation) — https://github.com/leeprichman/ClonoCluster
- **R (version >= 3.6)** (Runtime environment; installed via devtools::install_github('leeprichman/ClonoCluster')) — https://github.com/leeprichman/ClonoCluster
- **data.table** (R package dependency for handling assignment tables and CSV output in the paper workflow)

## Examples

```
clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)
```

## Evaluation signals

- Each per-dataset kappa-vs-alpha table contains exactly 11 alpha values (0 to 1 by 0.1)
- Kappa approaches perfect agreement (near 1) as alpha approaches 1 (purely clone weights) and reflects ~0 agreement at alpha = 0
- Most datasets show a gradual kappa rise across alpha, while the cardiomyocyte-directed iPSC (CJ) dataset shows an abrupt low-to-high kappa switch
- Each confusion table from cast_confusion cross-tabulates every cell's hybrid cluster assignment against its clone barcode membership without missing cells

## Limitations

- No reliable method was found to identify which experimental systems have intrinsic determinants of cell fate, so the kappa transition shape must be interpreted per dataset
- The appropriate relative weighting of transcriptome versus clone information may differ between endpoint and transitional systems and requires case-by-case evaluation
- ClonoCluster does not natively handle full-lineage tracing data (phylogenetic relationships), only same-vs-different barcode clone weights

## Evidence

- [methods] clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1): "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [methods] Barcode to cluster concordance plots with cast_confusion: "* Barcode to cluster concordance plots with `cast_confusion`"
- [intro] We linearly combined transcriptome and clone weights using alpha such that it returned purely transcriptome weights for alpha = 0 and purely clone weights for alpha = 1: "We linearly combined transcriptome and clone weights using α such that it returned purely transcriptome weights for α = 0 and purely clone weights for α = 1"
- [discussion] in the case of cardiomyocyte-directed iPSCs from Jiang et al., in which we know that cell fate is largely determined by extrinsic factors: "in the case of cardiomyocyte-directed iPSCs from Jiang et al.,18 in which we know that cell fate is largely determined by extrinsic factors"
- [readme] Confusion_stats.R - generate boxplots of alpha vs cohen's kappa, as in Figure S2B: [evidence span withheld — anchor status: external_doc]
