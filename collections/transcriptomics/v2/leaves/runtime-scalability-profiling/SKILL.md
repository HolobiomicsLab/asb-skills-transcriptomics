---
name: runtime-scalability-profiling
description: Use when choosing an imputation method for a large scRNA-seq matrix (e.g.,
  the 162,490-cell Darrah dataset) and you need to establish which methods complete
  in reasonable time versus failing with out-of-memory errors.
when_to_use_negative:
- Input is a small plate-based dataset (e.g., Goolam) where all methods finish quickly
  — scalability profiling adds no decision value
- The goal is to evaluate imputation accuracy only, without runtime or memory constraints
  — use a pure quality benchmark instead
- Data is bulk RNA-seq, which cannot measure cell-to-cell variability and is out of
  scope for single-cell imputation methods
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scISR
  role: Primary imputation method whose scalability is profiled; also installed from
    CRAN for benchmarking
  repo: https://github.com/duct317/scISR
- name: scImpute
  role: Competing imputer timed or flagged OOM on the same matrix
- name: SAVER
  role: Competing imputer timed or flagged OOM on the same matrix
- name: BISCUIT
  role: Competing imputer timed or flagged OOM on the same matrix
- name: MAGIC
  role: Competing imputer timed or flagged OOM on the same matrix
- name: DrImpute
  role: Competing imputer timed or flagged OOM on the same matrix
- name: scScope
  role: Competing imputer timed or flagged OOM on the same matrix
- name: DCA
  role: Competing imputer timed or flagged OOM on the same matrix
- name: DeepImpute
  role: Competing imputer timed or flagged OOM on the same matrix
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:a60c08f2f78f39613db2712356e52ff0c092967bc8cf10e9e3c55b2ac4f63d28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - BISCUIT
  - DCA
  - DeepImpute
  - DrImpute
  - MAGIC
  - SAVER
  - scISR
  - scImpute
  - scScope
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# runtime-scalability-profiling

## Summary

Benchmarks the wall-clock runtime and memory feasibility of scRNA-seq imputation methods on large datasets, using scISR as the reference implementation and eight competitors (scImpute, SAVER, BISCUIT, MAGIC, DrImpute, scScope, DCA, DeepImpute), and pairs runtime with imputation-quality (ARI) so scalability claims are judged alongside clustering benefit.

## When to use

Use when choosing an imputation method for a large scRNA-seq matrix (e.g., the 162,490-cell Darrah dataset) and you need to establish which methods complete in reasonable time versus failing with out-of-memory errors.

## When NOT to use

- Input is a small plate-based dataset (e.g., Goolam) where all methods finish quickly — scalability profiling adds no decision value
- The goal is to evaluate imputation accuracy only, without runtime or memory constraints — use a pure quality benchmark instead
- Data is bulk RNA-seq, which cannot measure cell-to-cell variability and is out of scope for single-cell imputation methods

## Inputs

- scRNA-seq count matrix with cell-type labels (e.g., Darrah, GSE139598, 162,490 cells)
- List of imputation methods to benchmark (scISR, scImpute, SAVER, BISCUIT, MAGIC, DrImpute, scScope, DCA, DeepImpute)
- True number of cell types k and known labels for ARI computation

## Outputs

- Per-method runtime or out-of-memory status
- Adjusted Rand Index per method (imputed matrices plus raw baseline)
- runtime_ari_comparison.csv with one record per method

## How to apply

Download the target count matrix with known cell-type labels (e.g., Darrah/GSE139598, 162,490 cells), then preprocess by removing genes expressed in fewer than two cells and genes with less than one percent non-zero-valued entries, applying a log2 transformation when the value range exceeds 100. Run scISR and each competitor on the identical preprocessed matrix, wall-clock timing every run and explicitly flagging out-of-memory failures instead of dropping them. To tie runtime to quality, cluster each imputed matrix (and the raw matrix as baseline) with k-means using the true number of cell types k and compute the Adjusted Rand Index against the known labels. Assemble one record per method with runtime (or OOM status) and ARI, written to runtime_ari_comparison.csv, so the fastest method's scalability can be weighed against its clustering gain.

## Related tools

- **scISR** (Primary imputation method whose scalability is profiled; also installed from CRAN for benchmarking) — https://github.com/duct317/scISR
- **scImpute** (Competing imputer timed or flagged OOM on the same matrix)
- **SAVER** (Competing imputer timed or flagged OOM on the same matrix)
- **BISCUIT** (Competing imputer timed or flagged OOM on the same matrix)
- **MAGIC** (Competing imputer timed or flagged OOM on the same matrix)
- **DrImpute** (Competing imputer timed or flagged OOM on the same matrix)
- **scScope** (Competing imputer timed or flagged OOM on the same matrix)
- **DCA** (Competing imputer timed or flagged OOM on the same matrix)
- **DeepImpute** (Competing imputer timed or flagged OOM on the same matrix)

## Examples

```
library(scISR); imputed <- scISR(data = raw)
```

## Evaluation signals

- One CSV record per method containing either a wall-clock runtime or an explicit OOM flag — no silent dropouts
- Identical preprocessed matrix (genes in <2 cells and <1% non-zero entries removed; log2 transform applied if value range >100) fed to every method
- ARI computed consistently: k-means with the true number of cell types k, run on both imputed and raw matrices as baseline
- The largest benchmark dataset (Darrah, 162,490 cells) is the accepted stress case: scISR completing it in 50 minutes while competitors fail is the expected outcome
- Runtime-versus-ARI pairing must be retained per method so a fast method is not mistaken for a good one

## Limitations

- Model-based imputers require excessive computational power, which is exactly why runtime/OOM profiling is needed but also means hardware specs strongly condition results
- Runtimes are wall-clock measurements on one machine; results are not portable across hardware and no software versions were stated for scISR or the R stats package
- A method may be fast yet still need quality verification — speed alone does not guarantee better clusters, hence ARI is recorded alongside runtime

## Evidence

- [results] preprocessing thresholds: "(i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
- [results] log2 rescaling rule: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
- [results] clustering evaluation with true k: "we use k-means to cluster the cells using the true number of cell types k as the number of clusters."
- [intro] why model-based methods are slow: "The quality of data imputed by methods in the first category (model-based methods) is determined by the validity of the assumption of the distribution models."
- [readme] scISR invocation from README: [evidence span withheld — anchor status: external_doc]
