---
name: mars-seq-gene-expression-preprocessing
description: Use when a raw single-cell gene-count dataset (e.g., MARS-seq myeloid
  progenitor data with corrupted gene names) must be cleaned and shaped into an N×D
  expression matrix before DTFLOW trajectory or branch analysis.
when_to_use_negative:
- The input is already a pre-processed gene expression matrix with no corrupted gene
  names — re-running Scanpy pre-processing and gene removal would be redundant
- The downstream goal is pseudotime or branch detection itself rather than data preparation
  — use the DTFLOW transition-matrix, RWR-diffusion, and RSKG branch-detection steps
  instead
- The dataset is not a gene-level single-cell expression table (e.g., already PCA-reduced
  embeddings), since the DTFLOW constructor expects raw_df as an N cells × D genes
  count matrix
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DTFLOW
  role: 'Consumes the pre-processed expression matrix: raw_df and time_stages are
    passed to DTFLOW(raw_df, time_stages) to start the pipeline'
  repo: https://github.com/statway/DTFLOW
- name: Scanpy
  role: Performs the pre-processing of the MMP dataset, including removal of the 10
    genes with corrupted names, before DTFLOW analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:1f13ba97257d6d53b922e90871e73c1c5a21042e93b29718fe999af9c478fb8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# mars-seq-gene-expression-preprocessing

## Summary

Prepares a raw MARS-seq single-cell count dataset (e.g., the mouse myeloid progenitor dataset of 2730 cells × 3451 informative genes) into a cleaned cells×genes expression matrix by applying Scanpy pre-processing, including removal of the 10 genes with corrupted names, as the entry step of the DTFLOW pseudotime workflow.

## When to use

Use when a raw single-cell gene-count dataset (e.g., MARS-seq myeloid progenitor data with corrupted gene names) must be cleaned and shaped into an N×D expression matrix before DTFLOW trajectory or branch analysis.

## When NOT to use

- The input is already a pre-processed gene expression matrix with no corrupted gene names — re-running Scanpy pre-processing and gene removal would be redundant
- The downstream goal is pseudotime or branch detection itself rather than data preparation — use the DTFLOW transition-matrix, RWR-diffusion, and RSKG branch-detection steps instead
- The dataset is not a gene-level single-cell expression table (e.g., already PCA-reduced embeddings), since the DTFLOW constructor expects raw_df as an N cells × D genes count matrix

## Inputs

- Raw MARS-seq single-cell gene count dataset (cells × genes), e.g., the MMP dataset with 2730 single cells and 3451 informative genes
- Time stage labels (time_stages) for the single cells, required by the DTFLOW constructor
- List/identification of genes with corrupted names to be dropped (10 genes for the MMP dataset)

## Outputs

- Cleaned gene expression matrix X(N×D) with N cells and D informative genes (2730 × 3451 for MMP)
- Pre-processed matrix ready for DTFLOW object creation and Markov transition matrix computation
- (For large datasets like MH) reduced representation of 40,210 cells with 100 approximate principal components

## How to apply

Load the raw single-cell dataset and format it as a gene expression matrix X(N×D) with N cells and D genes, as required to construct the DTFLOW object via z = DTFLOW(raw_df, time_stages). Apply Scanpy pre-processing to the dataset; for the MMP MARS-seq dataset (2730 cells, 3451 informative genes), this removes the 10 genes with corrupted names, yielding the cleaned 2730×3451 informative-gene matrix used for all downstream steps. The decision point is dataset scale: the very large MH microwell-seq dataset (40,210+ cells) is instead reduced to 100 approximate principal components after pre-processing. The output matrix feeds directly into DTFLOW's Markov transition matrix construction (k=10, the paper's optimal value) and random-walk-with-restart diffusion (restart parameter p=0.9), so any gene removal or reshaping must precede get_transition_matrix(k).

## Related tools

- **DTFLOW** (Consumes the pre-processed expression matrix: raw_df and time_stages are passed to DTFLOW(raw_df, time_stages) to start the pipeline) — https://github.com/statway/DTFLOW
- **Scanpy** (Performs the pre-processing of the MMP dataset, including removal of the 10 genes with corrupted names, before DTFLOW analysis)

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10)
```

## Evaluation signals

- Output matrix dimensions match the paper's specification: 2730 single cells and 3451 informative genes for the MMP MARS-seq dataset
- Exactly the 10 genes with corrupted names are removed; spot-check that no corrupted-name gene survives into the matrix used by DTFLOW
- The cleaned matrix is accepted by z = DTFLOW(raw_df, time_stages) and downstream z.get_transition_matrix(k) with k=10 runs without shape errors
- Downstream pseudotime and RSKG branches reproduce the expected MMP lineage structure (erythroid clusters 1-6, CMP clusters 7-10, DC cluster 11, GMP clusters 12-18, lymphoid cluster 19)
- Matrix row/column ordering is consistent with time_stages so pseudotime rooting and branch labeling remain cell-aligned

## Limitations

- Pre-processing and removal of corrupted-name genes are dataset-specific: the 10 corrupted genes apply to the MMP dataset, while the MH dataset instead uses 100 approximate principal components
- Parameter values (Gaussian kernel, number of neighbors k, restart probability) may vary from dataset to dataset and are not standardized, so the pre-processed matrix may need dataset-specific tuning downstream
- No specific software versions or tool implementations are reported for the pre-processing step, complicating exact reproduction

## Evidence

- [methods] 10 genes with corrupted names are removed from our analysis based on the pre-processing of Scanpy.: "10 genes with corrupted names are removed from our analysis based on the pre-processing of Scanpy."
- [methods] Pre-process single-cell dataset into gene expression matrix: "A. Pre-process a single-cell dataset into a gene expression matrix XN×D with N cells and D genes."
- [methods] The second dataset is the mouse myeloid progenitor (MMP) MARS-seq dataset that contains 2730 single cells and 3451 informative genes [43].: "The second dataset is the mouse myeloid progenitor (MMP) MARS-seq dataset that contains 2730 single cells and 3451 informative genes [43]."
- [readme] README: raw_df is a single cell dataset which contains N cells * D genes, time_stages is the time tags of single cells.: "Here `raw_df` is a single cell dataset which contains N cells * D genes, time_stages is the time tags of single cells."
- [discussion] the values of these parameters may vary from dataset to dataset.: "the values of these parameters may vary from dataset to dataset."
