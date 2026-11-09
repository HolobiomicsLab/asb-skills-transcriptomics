---
name: single-cell-rna-seq-cell-type-subsetting
description: Use when building or benchmarking GRN inference methods on snRNA-seq
  data where a specific cell type must be isolated from an annotated single-cell object
  and split into train/test sets to assess network generalization.
when_to_use_negative:
- Input is whole-tissue or spatial data without cell-type annotations — the README
  notes SCING can 'use whole visium data' directly, so cell-type subsetting is not
  applicable
- The dataset lacks reliable cell-type labels, since subsetting on wrong annotations
  would confound the GRN covariance structure
- The goal is batch-effect correction or module scoring (AUCell/leiden subnetworks)
  on an already-built GRN rather than constructing train/test GRN inputs
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCING
  role: Consumes the cell-type-subsetted, train-split expression data via BuildNetwork.ipynb
    (supercells, subsample GRNs, merge, prune edges) for GRN inference
  repo: https://github.com/XiaYangLabOrg/SCING
- name: ppcor
  role: Comparator GRN method run on the same subsetted train split for benchmarking
- name: PIDC
  role: Comparator GRN method run on the same subsetted train split for benchmarking
- name: GRNBOOST2
  role: Comparator gradient-boosting GRN method run on the same subsetted train split
    for benchmarking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:d15e9a7a6df71ea91f572325c6be0868a78a67d69a832c1dc84565cb1bd214b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - ppcor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# single-cell-rna-seq-cell-type-subsetting

## Summary

Subsets annotated cell populations (e.g., oligodendrocytes, astrocytes, microglia) from human prefrontal cortex snRNA-seq data and partitions them into 50/50 train/test splits with random gene subsamples, providing the controlled input regime that the SCING paper uses for GRN construction and held-out overfitting evaluation.

## When to use

Use when building or benchmarking GRN inference methods on snRNA-seq data where a specific cell type must be isolated from an annotated single-cell object and split into train/test sets to assess network generalization.

## When NOT to use

- Input is whole-tissue or spatial data without cell-type annotations — the README notes SCING can 'use whole visium data' directly, so cell-type subsetting is not applicable
- The dataset lacks reliable cell-type labels, since subsetting on wrong annotations would confound the GRN covariance structure
- The goal is batch-effect correction or module scoring (AUCell/leiden subnetworks) on an already-built GRN rather than constructing train/test GRN inputs

## Inputs

- Annotated human prefrontal cortex snRNA-seq dataset (e.g., Morabito control patients) with cell-type labels for oligodendrocytes, astrocytes, and microglia
- Cell-type annotation metadata (obs-level categorical labels)
- Gene subsample sizes (1,000 / 3,000 / 5,000 genes)
- Train/test split specification (50/50 per cell type)

## Outputs

- Per-cell-type single-cell expression subsets (train and test splits)
- Gene-subsampled expression matrices (1,000 / 3,000 / 5,000 genes per split)
- GRN training inputs ready for SCING BuildNetwork.ipynb or ppcor/PIDC/GRNBOOST2
- Predicted vs. true expression per gene on train and test splits for cosine-similarity benchmarking (grn_cosine_similarity_benchmark.csv)

## How to apply

Load the annotated single-cell (snRNA-seq) dataset and select the cells whose cell-type label matches the target populations — in the Morabito control human prefrontal cortex dataset these are oligodendrocytes, astrocytes, and microglia. Split each cell type's cells 50/50 into train and test sets so the network is built on the train split and evaluated on the held-out test split, mirroring the paper's gene-prediction validation design. For each cell type, draw random gene subsamples of 1,000, 3,000, and 5,000 genes to control network size and computational tractability. Feed each training subset into SCING's BuildNetwork workflow (supercell construction, GRN inference on subsamples, merging, and edge pruning) or into comparator methods (ppcor, PIDC, GRNBOOST2). The rationale is that GRN quality depends strongly on which cells define the covariance structure, so subsetting by cell type plus held-out evaluation isolates overfitting: cosine similarity between predicted and true expression is computed separately on train and test splits and aggregated as a test/train ratio.

## Related tools

- **SCING** (Consumes the cell-type-subsetted, train-split expression data via BuildNetwork.ipynb (supercells, subsample GRNs, merge, prune edges) for GRN inference) — https://github.com/XiaYangLabOrg/SCING
- **ppcor** (Comparator GRN method run on the same subsetted train split for benchmarking)
- **PIDC** (Comparator GRN method run on the same subsetted train split for benchmarking)
- **GRNBOOST2** (Comparator gradient-boosting GRN method run on the same subsetted train split for benchmarking)

## Examples

```
adata_sub = adata[adata.obs["cell_type"].isin(["oligodendrocyte", "astrocyte", "microglia"])]
```

## Evaluation signals

- Each cell type yields exactly two disjoint splits with a 50/50 cell ratio and no shared cells between train and test
- Gene subsample matrices contain exactly 1,000 / 3,000 / 5,000 genes per cell type and subsample draw
- SCING GRN construction on each train split follows the documented sequence: build supercells, build GRNs on subsamples, merge to a final GRN, then prune edges
- Per-gene GBRs trained on the train split predict held-out gene expression on the test split with cosine similarity computed against true expression for both splits
- The aggregated benchmark table (grn_cosine_similarity_benchmark.csv) contains the test/train cosine-similarity ratio and absolute cosine similarity for every method, cell type, and gene-subsample size

## Limitations

- Existing GRN tools are limited to thousands of pre-selected genes and cells because GRN inference from full transcriptomes in large scRNA-seq datasets is computationally intensive and intractable
- Existing methods show limited accuracy due to high gene sparsity and cell-to-cell heterogeneity in single-cell data
- SCING has many hyperparameters that require selection, and while robustness is shown across a range of hyperparameters, selection can be dataset dependent

## Evidence

- [readme] Gene prediction validation, to determine network overfitting (b). We split data into train and test sets and build a network on the train set.: "Gene prediction validation, to determine network overfitting (b). We split data into train and test sets and build a network on the train set."
- [readme] We then predict the expression of each gene in the test set and determine the distance from the true expression through cosine similarity.: "We then predict the expression of each gene in the test set and determine the distance from the true expression through cosine similarity."
- [other] Load the Morabito control human prefrontal cortex snRNA-seq dataset, subset the oligodendrocyte, astrocyte, and microglia populations, and split each cell type's cells 50/50 into train and test sets.: [evidence span withheld — anchor status: not_found]
- [readme] BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges.: "BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges."
- [readme] First, we select a specific cell type, or use whole visium data. We then cluster the cells using the leiden graph partitioning algorithm and merge subclusters into supercells.: "First, we select a specific cell type, or use whole visium data. We then cluster the cells using the leiden graph partitioning algorithm and merge subclusters into supercells."
