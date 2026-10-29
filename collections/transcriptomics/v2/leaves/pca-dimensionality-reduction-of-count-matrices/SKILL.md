---
name: pca-dimensionality-reduction-of-count-matrices
description: Use when a normalized or scaled cell-by-gene count matrix of barcoded
  cells needs a compact PC representation (e.g., npc = 25) to feed nearest-neighbor
  graph construction for ClonoCluster hybrid clustering.
when_to_use_negative:
- The count matrix is raw and not yet normalized/scaled — ClonoCluster expects 'a
  normalized or scaled count matrix of your barcoded cells'
- You only need clone-based structure with no transcriptome contribution (α = 1 uses
  only the size-normalized clone weight graph, so the PC step adds nothing)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ClonoCluster
  role: Provides dt2m for converting the data.table count matrix to a cell-by-gene
    matrix and irlba_wrap for truncated PCA of the transcriptome matrix
  repo: https://github.com/leeprichman/ClonoCluster
- name: irlba
  role: Truncated-SVD solver wrapped by irlba_wrap to compute the 25-principal-component
    representation
- name: data.table
  role: fread is used to load the count matrix and barcode TSV files before conversion
    with dt2m
- name: Seurat
  role: Consumes the PC representation downstream to build the shared-nearest-neighbor
    jaccard-weighted transcriptome graph
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:b2d291a575384e8fe080fd1a8f557c76958597f566e41e4a5ea2b2c671ff9e8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - R (version >=)
  - Seurat
  - data.table
  - irlba
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# pca-dimensionality-reduction-of-count-matrices

## Summary

Reduce a normalized/scaled single-cell count matrix to a low-dimensional principal component representation (25 PCs via irlba in the ClonoCluster workflow) that serves as the transcriptome-space input for building the Seurat shared-nearest-neighbor jaccard-weighted graph prior to clone-transcriptome integration.

## When to use

Use when a normalized or scaled cell-by-gene count matrix of barcoded cells needs a compact PC representation (e.g., npc = 25) to feed nearest-neighbor graph construction for ClonoCluster hybrid clustering.

## When NOT to use

- The count matrix is raw and not yet normalized/scaled — ClonoCluster expects 'a normalized or scaled count matrix of your barcoded cells'
- You only need clone-based structure with no transcriptome contribution (α = 1 uses only the size-normalized clone weight graph, so the PC step adds nothing)

## Inputs

- Normalized or scaled count matrix of barcoded cells (TSV, gene-by-cell format, loadable with data.table::fread)
- Two-column barcode table with columns 'rn' (cell IDs) and 'Barcode' (TSV)

## Outputs

- Cell-by-25 principal component matrix (output of irlba_wrap(cm, npc = 25)) suitable as input to clonocluster() and Seurat SNN graph construction

## How to apply

Load the count matrix from the deposited TSV with data.table::fread and convert the gene-by-cell table to a cell-by-gene matrix using ClonoCluster's dt2m (cm %<>% dt2m). The input must already be normalized or scaled, as specified in the package README. Compute the PC representation with ClonoCluster's irlba_wrap wrapper around the irlba truncated-SVD solver, using npc = 25 principal components as in the paper's workflow (pca <- irlba_wrap(cm, npc = 25)). The resulting PC matrix is then used to build the Seurat shared-nearest-neighbor jaccard-weighted transcriptome graph whose edge weights are later combined with clone weights via W = αβ(lin − J) + J. Rationale: the PC embedding preserves transcriptome similarity structure in a compact form, making the jaccard graph construction tractable and consistent with the tutorial's default parameters.

## Related tools

- **ClonoCluster** (Provides dt2m for converting the data.table count matrix to a cell-by-gene matrix and irlba_wrap for truncated PCA of the transcriptome matrix) — https://github.com/leeprichman/ClonoCluster
- **irlba** (Truncated-SVD solver wrapped by irlba_wrap to compute the 25-principal-component representation)
- **data.table** (fread is used to load the count matrix and barcode TSV files before conversion with dt2m)
- **Seurat** (Consumes the PC representation downstream to build the shared-nearest-neighbor jaccard-weighted transcriptome graph)

## Examples

```
cm %<>% dt2m; pca <- irlba_wrap(cm, npc = 25)
```

## Evaluation signals

- The output PC matrix has exactly npc = 25 columns and one row per cell matching the barcode table cell IDs
- Downstream, the Seurat SNN jaccard graph built on the PCs yields transcriptome edge weights that at α = 0 reproduce the transcriptome-only combined matrix
- Combined edge weights W = αβ(lin − J) + J (β = 0.1) stay in [0, 1] for every α in seq(0, 1, by = 0.1), indicating the PC-derived jaccard weights are well-behaved
- Conversion with dt2m yields a cell-by-gene orientation (not gene-by-cell), confirming correct matrix transposition before PCA

## Limitations

- Only R version >= 3.6 is stated; version numbers for other software (irlba, Seurat, data.table) are not reported
- The paper's workflow is tuned to npc = 25; no guidance is given for choosing the number of PCs per dataset
- The study did not address full-lineage tracing data (CRISPR-Cas9 genome editing or imaging-based family trees), so the transcriptome-only PC graph component's suitability in those settings is untested

## Evidence

- [methods] irlba_wrap with npc=25: "pca <- irlba_wrap(cm, npc = 25)"
- [intro] modified SNN graph edge weights with alpha: "the transcriptome nearest neighbor network graph edge weights are modified to incorporate clone clusters with a tunable free parameter, α"
- [readme] input requirements from README: "A normalized or scaled count matrix of your barcoded cells"
