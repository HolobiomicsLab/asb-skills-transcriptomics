---
name: feature-downselection-benchmark-design
description: Use when comparing computational scalability (wall-clock time) of dimension-reduction
  methods on scRNAseq count data, and you need a controlled design that fixes feature
  count (e.g., 1500 features) across benchmark datasets.
when_to_use_negative:
- The goal is clustering-accuracy benchmarking (e.g., ARI against reference labels)
  rather than runtime comparison — this skill only addresses wall-clock scalability
- You need to evaluate the adapted CA variants (CA-FT, binning, etc.) against standard
  CA; this design times standard corral CA versus glmPCA specifically
- Datasets cannot be down-selected to a common feature count, so a controlled equal-size
  timing comparison is not possible
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: corral
  role: Performs standard CA dimension reduction on the down-selected matrices using
    irlba-based partial SVD; its runtime is the CA arm of the comparison
  repo: github.com/laurenhsu1/corral
- name: glmPCA
  role: Computes the glmPCA embeddings on the identical 1500-feature matrices; its
    runtime is the comparator arm
- name: R
  role: Language environment in which the timing scripts and corral/glmPCA calls are
    executed
- name: Bioconductor
  role: Package ecosystem through which corral is installed and interfaces with single-cell
    classes (SingleCellExperiment)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:46afcd32715fb9d4f9898de4d069ba12a739696cd0f6653b17a03a297cad413f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  - glmPCA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# feature-downselection-benchmark-design

## Summary

A benchmark-design pattern for comparing wall-clock runtime of single-cell dimension-reduction methods (standard CA via corral versus glmPCA) by down-selecting ten benchmark scRNAseq datasets to a standardized 1500 features and recording per-dataset elapsed time for each method.

## When to use

Use when comparing computational scalability (wall-clock time) of dimension-reduction methods on scRNAseq count data, and you need a controlled design that fixes feature count (e.g., 1500 features) across benchmark datasets.

## When NOT to use

- The goal is clustering-accuracy benchmarking (e.g., ARI against reference labels) rather than runtime comparison — this skill only addresses wall-clock scalability
- You need to evaluate the adapted CA variants (CA-FT, binning, etc.) against standard CA; this design times standard corral CA versus glmPCA specifically
- Datasets cannot be down-selected to a common feature count, so a controlled equal-size timing comparison is not possible

## Inputs

- Ten benchmark scRNAseq count datasets with original-publication cell labels (Zhengmix4eq, SCMixology, Zhengmix and related benchmarks)
- Down-selected gene-by-cell count matrices standardized to 1500 features

## Outputs

- runtime_comparison.csv: per-method, per-dataset wall-clock timing table
- Per-dataset elapsed-time measurements for corral CA and glmPCA

## How to apply

Load the ten benchmark single-cell count datasets used in the study (Zhengmix4eq, SCMixology, Zhengmix and related benchmarks carrying original-publication cell labels), then down-select each to exactly 1500 features so both methods receive identically sized input matrices. Run standard CA with the corral R/Bioconductor package, which uses an irlba-based partial SVD, and record the elapsed wall-clock time per dataset; run glmPCA on the same down-selected matrices and record its time. Compile the per-dataset timings into a single runtime_comparison.csv with runtime per method per dataset. The rationale is that fixing the feature dimensionality isolates method-level scaling differences (corral's partial SVD versus glmPCA's iterative fitting) from dataset-size confounds.

## Related tools

- **corral** (Performs standard CA dimension reduction on the down-selected matrices using irlba-based partial SVD; its runtime is the CA arm of the comparison) — github.com/laurenhsu1/corral
- **glmPCA** (Computes the glmPCA embeddings on the identical 1500-feature matrices; its runtime is the comparator arm)
- **R** (Language environment in which the timing scripts and corral/glmPCA calls are executed)
- **Bioconductor** (Package ecosystem through which corral is installed and interfaces with single-cell classes (SingleCellExperiment))

## Examples

```
BiocManager::install("laurenhsu1/corral", dependencies = TRUE); sce <- sce[,1:1500]; t0 <- Sys.time(); corral(sce); Sys.time() - t0
```

## Evaluation signals

- Standard CA (corral) completes in under one minute on every down-selected dataset, including datasets with over 20,000 cells
- glmPCA requires over one hour on the equivalent ~1500 features x ~22,000 cells input matrix
- runtime_comparison.csv contains exactly one runtime value per (method, dataset) pair for all ten datasets
- Both methods were run on matrices down-selected to the same 1500 features, confirming the comparison is size-controlled
- Reproduced timings replicate the paper's reported behavior: CA is consistently faster, with glmPCA scaling less favorably

## Limitations

- Timings are wall-clock and hardware-dependent; no tool or package versions are reported for corral or its dependencies in the article, complicating exact reproduction
- The benchmark datasets (SCMixology, Zhengmix) are, by design, simpler than data from complex tissues and lack independently validated cell type annotations, so the design covers scalability but not accuracy on complex tissues
- No accession IDs or links for the SCMixology and Zhengmix benchmarking datasets are provided in the article's discussion

## Evidence

- [abstract] corral CA consistently faster than glmPCA: "we propose five adaptations of CA, which are fast, scalable, and outperform standard CA and glmPCA"
- [abstract] corral interfaces with Bioconductor single-cell classes: "We implement CA for scRNAseq data in corral, an R/Bioconductor package which interfaces directly with single cell classes in Bioconductor."
- [readme] corral purpose for count data: "corral is a package to perform correspondence analysis on count data, particularly for single-cell (e.g., RNA-seq)."
- [discussion] benchmark datasets are simple by design: "Except for SCMixology and Zhengmix (both comprising well-defined cell clusters and by design simpler than data from complex tissues)"
