---
name: tmm-bulk-rnaseq-normalization
description: Use when matched bulk RNA-seq replicates (e.g. GSE52529 Trapnell myoblast
  bulk data with three biological replicates) must be normalized to define true-zero
  / expression-level bins against which scRNA-seq dropout imputation is judged.
when_to_use_negative:
- There are no matched bulk RNA-seq replicates available to define biologically-silent
  (true-zero) genes, e.g. only a single-cell matrix with no ground-truth reference.
- The bulk data are single-end samples without biological replicates, making median-across-replicates
  and composition-bias checking unreliable.
- The goal is standalone differential expression between bulk conditions rather than
  building reference expression bins for evaluating scRNA-seq imputation.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: AutoImpute
  role: Downstream single-cell imputation method whose true-zero retention is evaluated
    against the TMM-normalized bulk expression bins
  repo: github.com/divyanshu-talwar/AutoImpute
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:b458dad486a7b09c03b6ad624619106f83c2ea970799ee5d9b5a093950db29f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  - MAGIC
  - drImpute
  - scImpute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tmm-bulk-rnaseq-normalization

## Summary

Normalize matched bulk RNA-seq replicates with TMM followed by log2 transformation to detect and correct composition bias, so that gene-level bulk expression bins (e.g. biologically-silent zero-expression genes from Trapnell myoblast GSE52529) provide a trustworthy reference for evaluating single-cell zero retention.

## When to use

Use when matched bulk RNA-seq replicates (e.g. GSE52529 Trapnell myoblast bulk data with three biological replicates) must be normalized to define true-zero / expression-level bins against which scRNA-seq dropout imputation is judged.

## When NOT to use

- There are no matched bulk RNA-seq replicates available to define biologically-silent (true-zero) genes, e.g. only a single-cell matrix with no ground-truth reference.
- The bulk data are single-end samples without biological replicates, making median-across-replicates and composition-bias checking unreliable.
- The goal is standalone differential expression between bulk conditions rather than building reference expression bins for evaluating scRNA-seq imputation.

## Inputs

- Bulk RNA-seq count matrix for three biological replicates of undifferentiated myoblasts (GEO GSE52529, Trapnell)
- Preprocessed single-cell RNA-seq count matrix (genes x cells) for matched cells

## Outputs

- TMM-normalized, log2-transformed bulk expression matrix
- Per-gene median bulk expression values across the three replicates
- Gene bins: biologically-silent (zero bulk expression) bin plus bins grouped by median expression level
- true_zero_fractions.csv with per-bin zero fractions (natural-log scale) in the imputed single-cell data

## How to apply

Download the matched bulk RNA-seq replicates (Trapnell myoblast data, GEO accession GSE52529) and process them with TMM normalization followed by log2 transformation, which checks for composition bias arising from library-composition differences between replicates. Next, compute the median gene expression across the three biological replicates of undifferentiated myoblasts and bin genes: genes with zero bulk expression form the biologically-silent (true-zero) bin, and the remaining genes are grouped by median expression level. These bulk-derived bins are then matched to the preprocessed single-cell matrix (gene filtering at >=3 reads in >=3 cells, median library-size normalization, top-1000 high-dispersion genes, log2 with pseudo-count 1) after AutoImpute imputation to compute per-bin zero fractions. The rationale is that only composition-bias-corrected bulk medians reliably distinguish biologically silenced genes from highly expressed genes prone to single-cell dropout.

## Related tools

- **AutoImpute** (Downstream single-cell imputation method whose true-zero retention is evaluated against the TMM-normalized bulk expression bins) — github.com/divyanshu-talwar/AutoImpute

## Evaluation signals

- Log2-transformed, TMM-normalized bulk replicates show no residual composition bias across the three myoblast replicates.
- Median bulk expression is exactly zero for every gene assigned to the biologically-silent bin.
- In the imputed single-cell data, the biologically-silent bin retains the highest fraction of zeros while zero fraction decreases as bulk median expression increases.
- true_zero_fractions.csv reports per-bin zero fractions on the natural-log scale as in the paper.

## Limitations

- Requires matched bulk and single-cell data from the same biological system; the bin definition assumes zero bulk median expression across all three replicates truly reflects biological silencing.
- The skill is specific to bulk RNA-seq normalization for reference-building; it does not itself impute single-cell dropouts (that is AutoImpute's role).

## Evidence

- [other] TMM normalization step from workflow: [evidence span withheld — anchor status: not_found]
- [other] Binning by median bulk expression: [evidence span withheld — anchor status: not_found]
- [other] Verification trend: [evidence span withheld — anchor status: not_found]
- [readme] Imputed matrix input format: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
