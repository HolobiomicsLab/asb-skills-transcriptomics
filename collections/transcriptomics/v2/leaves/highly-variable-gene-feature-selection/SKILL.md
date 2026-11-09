---
name: highly-variable-gene-feature-selection
description: Use when preparing normalized single-cell expression matrices (e.g.,
  log-transformed TISCH tumor microenvironment data) as model inputs, and a compact,
  informative gene feature set is needed before training a classifier.
when_to_use_negative:
- Input data has not yet been log-normalized — variance ranking on raw counts is dominated
  by sequencing depth and total-count differences
- The goal is spatial transcriptomics spot annotation with a spatially trained Cancer-Finder
  model, where the training set and feature space differ from the 4572-gene scRNA-seq
  feature set
- Hematologic tumor data, for which the article explicitly does not recommend Cancer-Finder,
  making feature selection for it moot
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Performs 'NormalizeData' + log1p global-scaling normalization that precedes
    variance-based gene ranking
- name: SCANPY
  role: Python-side single-cell preprocessing environment in which the variance ranking/HVG
    selection can be executed
- name: Cancer-Finder
  role: Downstream domain-generalization network whose 4572-neuron input layer is
    sized by the selected highly variable genes
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: PyTorch
  role: Framework used to build the network whose input dimension equals the number
    of selected genes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:eeaf9ae14508609a0e22a7e5b336bc58119159e40507c314de5efba30ae9437b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - PyTorch
  - SCANPY
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Highly-variable gene feature selection

## Summary

Ranks genes by variance across all cells of the normalized training matrix and retains the most variable ones as the input feature vector for Cancer-Finder (4572 features for scRNA-seq), reducing input dimensionality while preserving genes with informative expression variability.

## When to use

Use when preparing normalized single-cell expression matrices (e.g., log-transformed TISCH tumor microenvironment data) as model inputs, and a compact, informative gene feature set is needed before training a classifier.

## When NOT to use

- Input data has not yet been log-normalized — variance ranking on raw counts is dominated by sequencing depth and total-count differences
- The goal is spatial transcriptomics spot annotation with a spatially trained Cancer-Finder model, where the training set and feature space differ from the 4572-gene scRNA-seq feature set
- Hematologic tumor data, for which the article explicitly does not recommend Cancer-Finder, making feature selection for it moot

## Inputs

- Log-transformed global-scaling-normalized gene-by-cell expression matrix (Seurat 'NormalizeData' + log1p) from the 74 TISCH tumor microenvironment datasets
- Class-balanced training split (malignant:non-malignant downsampled 1:1 per tissue; 328,230 cells from 13 tissues)

## Outputs

- Ranked gene list by across-cell variance
- Selected highly variable gene feature matrix with 4572 genes (model input features for the scRNA-seq model)

## How to apply

After log-transformed global-scaling normalization (Seurat 'NormalizeData' followed by log1p), compute the per-gene variance across all cells in the 74-TISCH-dataset training matrix (328,230 cells from 13 tissues, malignant/non-malignant downsampled 1:1 per tissue). Rank genes in descending order of variance and select the highly variable genes as model inputs — the article used 4572 features for the scRNA-seq model. These selected genes form the input layer dimension (4572 neurons) of the two-layer fully connected Cancer-Finder network, so the feature count directly sets the network's input width. Rationale: variance-based filtering keeps genes with discriminative, non-constant expression across domains, which supports the domain-generalization objective of learning a pan-cancer malignant/non-malignant boundary.

## Related tools

- **Seurat** (Performs 'NormalizeData' + log1p global-scaling normalization that precedes variance-based gene ranking)
- **SCANPY** (Python-side single-cell preprocessing environment in which the variance ranking/HVG selection can be executed)
- **Cancer-Finder** (Downstream domain-generalization network whose 4572-neuron input layer is sized by the selected highly variable genes) — https://github.com/Patchouli-M/SequencingCancerFinder
- **PyTorch** (Framework used to build the network whose input dimension equals the number of selected genes)

## Examples

```
import scanpy as sc; sc.pp.highly_variable_genes(adata, flavor='seurat', n_top_genes=4572); adata = adata[:, adata.var.highly_variable]
```

## Evaluation signals

- Selected feature count equals 4572 for the scRNA-seq model, matching the network's 4572-neuron input layer
- Every selected gene has nonzero across-cell variance in the normalized training matrix (constant genes excluded)
- Downstream Cancer-Finder training on the selected features reaches ~95.16% average internal-validation accuracy across the 13 tissues
- Feature selection is recomputed on the normalized, 1:1 downsampled training matrix so malignant/non-malignant classes contribute equally to variance estimates

## Limitations

- Variance-based selection is unsupervised and may retain highly variable genes unrelated to malignancy
- The 4572-gene feature set is specific to the scRNA-seq training data; spatial transcriptomics models are trained by substituting the training set, changing the feature space
- Robustness holds only to a certain percentage of mislabeling in the training set (Supplementary Fig. 11), which also affects downstream feature relevance

## Evidence

- [intro] TISCH training corpus: "We collected 74 human tumor microenvironment datasets from Tumor Immune Single Cell Hub (TISCH)27 as training sets"
- [abstract] Cancer-Finder accuracy target: "rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%"
- [readme] Cancer-Finder description (README): "we present Cancer-Finder, a domain generalization-based deep learning algorithm that can rapidly identify malignant cells/spots in single-cell and spatial transcriptomics data"
