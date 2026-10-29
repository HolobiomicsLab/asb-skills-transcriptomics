---
name: rare-cell-type-multistage-clustering
description: Use when a single-cell RNA-seq clustering run merges a rare annotated
  cell type into a larger cluster and a multi-stage re-clustering could isolate it.
when_to_use_negative:
- The rare cell type is already cleanly separated by the single-stage scCAN run (stage
  two adds no resolution).
- The dataset has fewer than samp.size=5000 cells, so Module 3's train/test split
  and k-NN classification do not apply.
- Labels are ambiguous or the annotation itself is uncertain — F1 evaluation against
  a wrong reference would be meaningless.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCAN
  role: Primary clustering engine run in both stages; autoencoder-based feature selection,
    VAE dimensionality reduction, network fusion, and spectral clustering with k-NN
    classification for n>5000.
  repo: https://cran.r-project.org/package=scCAN
- name: SHARP
  role: Benchmark comparison method cited alongside scCAN for single-cell clustering
    accuracy.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:d616486d253db0de2432668f6633310a627c6b76c2adb267248104b4b76d9067
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SHARP
  - scCAN
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# rare-cell-type-multistage-clustering

## Summary

Two-stage scCAN clustering for detecting rare cell types: run scCAN once to get broad clusters, then re-cluster each stage-one cluster with scCAN so rare populations (e.g. tRBC in Zilionis) separate from the rest.

## When to use

Use when a single-cell RNA-seq clustering run merges a rare annotated cell type into a larger cluster and a multi-stage re-clustering could isolate it.

## When NOT to use

- The rare cell type is already cleanly separated by the single-stage scCAN run (stage two adds no resolution).
- The dataset has fewer than samp.size=5000 cells, so Module 3's train/test split and k-NN classification do not apply.
- Labels are ambiguous or the annotation itself is uncertain — F1 evaluation against a wrong reference would be meaningless.

## Inputs

- Normalized single-cell expression matrix (cells as rows, genes as columns), e.g. Zilionis human lung (GSE127465, 34,558 cells)
- Annotated cell-type labels for the rare type of interest (e.g. tRBC) used for F1 scoring

## Outputs

- Stage-one scCAN cluster assignments for all cells
- Stage-two sub-cluster assignments per stage-one cluster (e.g. cluster 2 split into 2_1 and 2_2)
- F1 score of rare cell type separation against annotated labels

## How to apply

Load a normalized expression matrix with cells as rows and genes as columns (Zilionis lung dataset, 34,558 cells, inDrop). Run scCAN with default parameters: min-max scaling to 0–1, non-negative-kernel autoencoder keeping the top 5000 genes with a bottleneck of 50, stacked VAE with latent dimension d=15; because n>5000, apply Module 3 — randomly select a 5000-cell training set, cluster it with network fusion (k=30 neighbors) plus eigengap-based spectral clustering, and classify remaining cells via k-NN. Treat stage-one clusters as inputs to stage two, re-running scCAN on each cluster (with default samp.size=5000), following the multi-stage strategy the paper recommends for rare cell types. Score rare-type separation with an F1 score of stage-two sub-clusters against annotated labels; in the reported Zilionis run, cluster 2 split into sub-clusters 2_1 and 2_2, fully separating tRBC with F1 = 1.

## Related tools

- **scCAN** (Primary clustering engine run in both stages; autoencoder-based feature selection, VAE dimensionality reduction, network fusion, and spectral clustering with k-NN classification for n>5000.) — https://cran.r-project.org/package=scCAN
- **SHARP** (Benchmark comparison method cited alongside scCAN for single-cell clustering accuracy.)

## Examples

```
Rscript -e 'library(scCAN); data("SCE"); data <- t(SCE$data); if(max(data)>100) data <- log2(data+1); res <- scCAN(data, r.seed=1); print(res$cluster)'
```

## Evaluation signals

- Stage two must produce sub-clusters within at least one stage-one cluster (e.g. cluster 2 → 2_1, 2_2).
- Computed F1 score of the rare type (tRBC) separation should equal the reported value of 1 within numerical tolerance.
- Rare cells should be completely separated from other cell types in a stage-two sub-cluster.
- Number of stage-one clusters should be determined by the eigengap method on the fused similarity network.
- Training set should be exactly 5000 randomly selected cells (samp.size=5000), with remaining cells assigned by k-NN.

## Limitations

- Stage-two re-clustering quality depends on the stage-one partition — if the rare type is split across multiple stage-one clusters, F1 can degrade.
- Random selection of the 5000-cell training set introduces stochasticity; results should be checked with a fixed r.seed.
- Module 3's k-NN classification of the remaining cells may misassign rare cells that are underrepresented in the training sample.

## Evidence

- [methods] Big Data Analysis (Module 3): "Methods > Network Fusion And Spectral Clustering For Cell Segregation (Module 2)"
- [methods] Train/test split for big data: "we split the cells into two sets: a training set of 5000 randomly selected cells and a testing set that consists of the remaining cells"
- [methods] k-NN classification of testing cells: [evidence span withheld — anchor status: not_found]
- [methods] Eigengap method for cluster count: "we use the eigengap method48 to determine the number of clusters"
- [readme] Package install from GitHub: "devtools::install_github('bangtran365/scCAN')"
- [readme] scCAN function call example: "result <- scCAN(data, r.seed = 1)"
- [results] Multi-stage clustering rationale: [evidence span withheld — anchor status: not_found]
