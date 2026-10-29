---
name: cross-modality-common-gene-selection
description: 'Use when preparing unpaired scRNA-seq and scATAC-seq inputs for scBridge:
  the target gene activity matrix and the source count matrix must be reduced to their
  common gene set before encoding.'
when_to_use_negative:
- The two modalities already share an identical gene feature space and no feature
  reconciliation is needed
- The input is peak-by-cell scATAC-seq data that has not yet been converted to a gene
  activity matrix (conversion via tools such as Signac must happen first)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scBridge
  role: Downstream integrator that requires source and target matrices restricted
    to common genes before training its encoder and classifier
  repo: https://github.com/XLearning-SCU/scBridge
- name: Python
  role: Implementation language used with the anndata library to load h5ad files and
    intersect gene features
- name: Signac
  role: R tool used upstream to transform scATAC-seq peak data into the gene activity
    matrix that this skill subsets
  repo: https://github.com/stuart-lab/signac
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:7195e591dd0d8c4cbe376a3c45d2886544b679b47fc1e666e2150b0fea4fcb7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - Python
  - Signac
  - scBridge
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Cross-modality common gene selection

## Summary

Intersects the gene features of a scATAC-seq gene activity matrix (h5ad) with the gene features of the scRNA-seq count matrix so that only shared genes are retained, a prerequisite before scBridge ingests both modalities.

## When to use

Use when preparing unpaired scRNA-seq and scATAC-seq inputs for scBridge: the target gene activity matrix and the source count matrix must be reduced to their common gene set before encoding.

## When NOT to use

- The two modalities already share an identical gene feature space and no feature reconciliation is needed
- The input is peak-by-cell scATAC-seq data that has not yet been converted to a gene activity matrix (conversion via tools such as Signac must happen first)

## Inputs

- Raw scATAC-seq gene activity matrix in AnnData h5ad format (e.g., derived from peak data via Signac)
- scRNA-seq gene count matrix in AnnData h5ad format with cell type annotations in obs['CellType']

## Outputs

- Subsetted scATAC-seq gene activity matrix restricted to the common gene set (h5ad)
- Subsetted scRNA-seq count matrix restricted to the common gene set (h5ad)

## How to apply

Load the raw scATAC-seq gene activity matrix from its h5ad file with the anndata Python library and the scRNA-seq count matrix from its h5ad source file. Compute the intersection of gene feature names across the two modalities and subset both AnnData objects to retain only the common genes, since scBridge requires a shared feature space between encoder inputs. In the mouse atlas analysis this step yielded 18 common cell types and 12,689 common genes. After subsetting, the target matrix can proceed to TF-IDF preprocessing (target_preprocess='TFIDF', the default) followed by standardization, and be saved in AnnData h5ad format.

## Related tools

- **scBridge** (Downstream integrator that requires source and target matrices restricted to common genes before training its encoder and classifier) — https://github.com/XLearning-SCU/scBridge
- **Python** (Implementation language used with the anndata library to load h5ad files and intersect gene features)
- **Signac** (R tool used upstream to transform scATAC-seq peak data into the gene activity matrix that this skill subsets) — https://github.com/stuart-lab/signac

## Examples

```
import anndata as ad; a_rna, a_atac = ad.read_h5ad('rna.h5ad'), ad.read_h5ad('atac_activity.h5ad'); common = a_rna.var_names.intersection(a_atac.var_names); a_atac[:, common].write('target_common_genes.h5ad')
```

## Evaluation signals

- Output h5ad is readable by anndata.read_h5ad without error
- Feature dimension of both subsetted matrices equals the number of genes in the intersection of the two original var_names sets
- Gene name sets of the subsetted source and target matrices are exactly identical
- Obs (cell) dimension of each matrix is unchanged relative to its input
- Downstream scBridge run accepts both matrices without a gene-dimension mismatch error

## Limitations

- scBridge requires common genes across modalities, so genes present in only one omics are permanently discarded and cannot be recovered downstream
- The scATAC-seq gene activity matrix must first be generated from peak data using tools such as Signac; this skill does not perform that transformation

## Evidence

- [readme] Common genes requirement: "scBridge accepts AnnData (https://anndata.readthedocs.io/en/latest/index.html) as inputs, including the source (scRNA-seq) and target (scATAC-seq) h5ad files."
- [readme] Gene activity conversion prerequisite: "For scATAC-seq peak data, it needs to be first transformed into a scATAC-seq activity matrix using tools such as Signac"
- [discussion] Mouse atlas common-gene example: "We reserved cells of 18 common types and 12,689 common genes for the analysis"
- [methods] Target preprocessing default after gene selection: "`--target_preprocess=["Standard", "TFIDF"]` (Default="TFIDF") the preprocessing strategy for target data"
