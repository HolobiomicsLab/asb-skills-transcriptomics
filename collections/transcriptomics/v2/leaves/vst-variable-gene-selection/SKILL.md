---
name: vst-variable-gene-selection
description: Use when building a Symphony reference from scRNA-seq count/expression
  matrices (e.g., pbmcs_exprs_small) and you need a shared variable-gene panel before
  scaling, PCA, and Harmony integration.
when_to_use_negative:
- Input is already a feature-reduced table (e.g., a cell x embedding or precomputed
  PCA matrix) rather than gene expression counts
- You are doing de novo query-only analysis for novel cell type discovery, where Symphony
  reference compression including its variable-gene panel is not appropriate
- Reference and query were analyzed with non-standardized feature sets (e.g., scATAC-seq
  peaks), where the paper notes it is 'not immediately clear how to optimally select
  the best peak features'
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Symphony
  role: Provides the buildReference pipeline whose first step is vst variable gene
    selection feeding into scaling, PCA, and Harmony integration
  repo: github.com/immunogenomics/symphony
- name: singlecellmethods
  role: Supplies vargenes_vst and scaleDataWithStats used to select and normalize
    the topn = 1000 per-donor variable genes
  repo: github.com/immunogenomics/singlecellmethods
- name: Harmony
  role: Consumes the VST-selected variable gene space downstream to integrate reference
    batches (return_object = TRUE) before reference compression
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:be2718510214be3607d4ab8a98c7181db88f8d785140a84752fcbfd48a823d28
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - Symphony
  - dplyr
  - irlba
  - singlecellmethods
  - uwot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# vst-variable-gene-selection

## Summary

Select variable genes with variance-stabilizing transformation (vst) for a Symphony reference build, grouping cells per donor (e.g., per 10x PBMC technology) and retaining topn = 1000 genes so downstream scaling, SVD-based PCA, and Harmony integration operate on informative features.

## When to use

Use when building a Symphony reference from scRNA-seq count/expression matrices (e.g., pbmcs_exprs_small) and you need a shared variable-gene panel before scaling, PCA, and Harmony integration.

## When NOT to use

- Input is already a feature-reduced table (e.g., a cell x embedding or precomputed PCA matrix) rather than gene expression counts
- You are doing de novo query-only analysis for novel cell type discovery, where Symphony reference compression including its variable-gene panel is not appropriate
- Reference and query were analyzed with non-standardized feature sets (e.g., scATAC-seq peaks), where the paper notes it is 'not immediately clear how to optimally select the best peak features'

## Inputs

- Reference gene-by-cell expression matrix (e.g., pbmcs_exprs_small.rda from 10x PBMC 3'v1, 3'v2, and 5' datasets)
- Cell-level metadata with a donor/technology covariate (e.g., pbmcs_meta_small.rda with 'donor' column)

## Outputs

- Data frame of variable genes with per-group means and standard deviations (vargenes_means_sds), topn = 1000 per donor group
- Reference expression matrix subset to the selected variable genes, ready for scaleDataWithStats and SVD PCA

## How to apply

Apply vargenes_vst to the full reference expression matrix with the groups argument set to the donor/technology covariate (as.character(ref_metadata[['donor']])) and topn = 1000, so that 1000 variable genes are selected per group and unioned. Subset the reference expression to these genes, then scale them with scaleDataWithStats using the vargenes_means_sds means and standard deviations before running SVD PCA (20 PCs) and Harmony (K = 100, vars = 'donor'). Selecting genes per donor/technology rather than globally guards against one batch dominating the variable-feature set, which supports Symphony's goal of removing confounding study-design signals. In the pbmcs_exprs_small/pbmcs_meta_small leave-one-technology-out workflow this step precedes buildReference (with do_normalize = FALSE) and directly determines the feature space on which mapping fidelity (LISI ~2.12 vs 2.15, k-NN correlation > 0.4 for 87.7% of cells) is achieved.

## Related tools

- **Symphony** (Provides the buildReference pipeline whose first step is vst variable gene selection feeding into scaling, PCA, and Harmony integration) — github.com/immunogenomics/symphony
- **singlecellmethods** (Supplies vargenes_vst and scaleDataWithStats used to select and normalize the topn = 1000 per-donor variable genes) — github.com/immunogenomics/singlecellmethods
- **Harmony** (Consumes the VST-selected variable gene space downstream to integrate reference batches (return_object = TRUE) before reference compression)

## Examples

```
var_genes = vargenes_vst(ref_exp_full, groups = as.character(ref_metadata[['donor']]), topn = 1000)
```

## Evaluation signals

- The selected gene set is the union of topn = 1000 vst-selected genes computed within each donor group, not a single global topn list
- Downstream scaling uses the exact means and standard deviations stored in vargenes_means_sds for the selected genes
- In the PBMC benchmark, the full pipeline built on these genes yields technology LISI ~2.12 (vs 2.15 de novo Harmony) and 87.7% of query cells with k-NN correlation > 0.4 at k = 500
- Label transfer built on this feature space reaches ~97.5% 5-NN classification accuracy with median F1 ~0.97

## Limitations

- Identification of novel cell types that have failed to map is an unsolved direction; the variable-gene panel optimizes reference-defined states, not discovery of query-specific populations
- Mapping metrics may be less informative when a novel query population closely resembles an existing reference population, regardless of feature selection
- Equivalence with de novo Harmony depends on comprehensive cell type coverage in the reference (Condition I), which the vst-selected feature space cannot compensate for

## Evidence

- [methods] vargenes_vst call with donor groups and topn=1000: "var_genes = vargenes_vst(ref_exp_full, groups = as.character(ref_metadata[['donor']]), topn = 1000)"
- [methods] Variable gene selection precedes subsetting of reference expression: "Select variable genes and subset reference expression by variable genes"
- [methods] Scaling uses calculated gene means and standard deviations: "Scale data using calculated gene means and standard deviations"
- [readme] Symphony builds large-scale integrated reference atlases: "We present Symphony ( https://github.com/immunogenomics/symphony ), an algorithm for building large-scale, integrated reference atlases"
- [intro] Symphony must remove confounding study-design signals, motivating per-donor gene selection: "it must be able to remove confounding signals due to complex study design in both the reference and query"
