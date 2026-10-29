---
name: single-cell-differential-expression-testing
description: Use when you have a library-size-normalized, log-transformed scRNA-seq
  expression matrix with cell cluster labels and need per-cluster marker ligand/receptor
  statistics (log2FC, adjusted P) to feed cell-cell communication inference.
when_to_use_negative:
- The expression matrix is raw counts not yet normalized by library-size and log-transformed
  — CommPath requires normalized, log-transformed values as input.
- The goal is differential testing of arbitrary genes or pathways rather than the
  curated ligand and receptor sets feeding LR interaction inference — a general DE
  tool would be more appropriate.
- Only one unlabeled cell population exists with no cluster identity vector, since
  markers are defined per cluster of cells.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CommPath
  role: Provides createCommPath to build the analysis object and findLRmarker to run
    wilcox.test/t.test differential expression testing of ligands and receptors per
    cluster
  repo: https://github.com/yingyonghui/CommPath
- name: R
  role: Runtime environment; CommPath is installed via devtools::install_github and
    depends on Matrix, circlize, ggplot2, dplyr, reshape2 (and GSVA for later pathway
    scoring)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.10.028
    title: 'CommPath: An R package for inference and analysis of pathway-mediated
      cell-cell communication chain from single-cell transcriptomics'
  source_content_hash: sha256:401ee9f085cb12339d9aa2f978d4bb8cc9a686c9b14da3abfebb5565fd431784
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yingyonghui/CommPath
  repo_url: https://github.com/yingyonghui/CommPath
  tools:
  - CommPath
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-differential-expression-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies significantly highly expressed marker ligands and receptors per cell cluster by performing non-parametric differential expression tests (t.test or wilcox.test) on normalized scRNA-seq data, yielding the log2FC and adjusted P values that downstream ligand-receptor interaction scoring depends on.

## When to use

Use when you have a library-size-normalized, log-transformed scRNA-seq expression matrix with cell cluster labels and need per-cluster marker ligand/receptor statistics (log2FC, adjusted P) to feed cell-cell communication inference.

## When NOT to use

- The expression matrix is raw counts not yet normalized by library-size and log-transformed — CommPath requires normalized, log-transformed values as input.
- The goal is differential testing of arbitrary genes or pathways rather than the curated ligand and receptor sets feeding LR interaction inference — a general DE tool would be more appropriate.
- Only one unlabeled cell population exists with no cluster identity vector, since markers are defined per cluster of cells.

## Inputs

- Library-size-normalized, log-transformed expression matrix (gene × cell), e.g. tumor.expr from HCC.tumor.3k.RData
- Vector of cell identity labels matching cell order, e.g. tumor.label
- Species selection (hsapiens, mmusculus, rnorvegicus, drerio, dmelanogaster, celegros databases in CommPath)
- Test method choice: "wilcox.test" or "t.test"

## Outputs

- CommPath object with populated LR.marker slot: dataframe of differential expression test results for ligands and receptors per cluster (log2FC, P, adjusted P)
- Per-cluster marker ligand and receptor lists used by findLRpairs

## How to apply

First construct a CommPath object from the normalized expression matrix (gene × cell, e.g. tumor.expr) and matching cell identity vector (tumor.label) via createCommPath with the correct species (e.g. "hsapiens"), since expression values must already be normalized by library-size and log-transformed. Then run findLRmarker with method = "wilcox.test" (or "t.test") to test ligands and receptors for significant high expression in each cluster; results are stored in the object's LR.marker slot as a dataframe from the differential expression test. The resulting per-cluster log2FC and adjusted P values for each ligand and receptor are the statistical basis for the subsequent findLRpairs step, which combines them into an intensity score (product of log2FC of ligand in sender and receptor in receiver) and a combined p-value, screened with thresholds such as logFC.thre = 0 and p.thre = 0.01. Judgment of correctness: marker results should contain entries for every cluster, and only markers passing the significance screen should propagate into the LR association table.

## Related tools

- **CommPath** (Provides createCommPath to build the analysis object and findLRmarker to run wilcox.test/t.test differential expression testing of ligands and receptors per cluster) — https://github.com/yingyonghui/CommPath
- **R** (Runtime environment; CommPath is installed via devtools::install_github and depends on Matrix, circlize, ggplot2, dplyr, reshape2 (and GSVA for later pathway scoring))

## Examples

```
tumor.obj <- findLRmarker(object = tumor.obj, method = "wilcox.test")
```

## Evaluation signals

- The CommPath object's LR.marker slot is a dataframe and is populated for every cluster in tumor.label after findLRmarker runs without error.
- The subsequent findLRpairs output table (e.g. LR_pairs_statistics.csv) exists and contains the columns log2FC.LR, P.val.LR, and P.val.adj.LR.
- Only ligand-receptor pairs passing logFC.thre = 0 and p.thre = 0.01 appear in the filtered cluster-pair statistics.
- Interaction intensity equals the product of the ligand log2FC in the sender cluster and receptor log2FC in the receiver cluster, and the combined P equals 1 − (1−P_l,A)(1−P_r,B).
- Expression matrix passed to createCommPath is normalized and log-transformed (spot-check that per-cell library sizes are equalized and values are on a log scale).

## Limitations

- GSVA-based pathway scoring downstream is time-consuming on large datasets; CommPath offers the UCell algorithm or average-based assessment as alternatives (relevant when scaling this DE step to large matrices).
- The example analysis is demonstrated on ~3000 cells across the top 5000 highly variable genes; results depend on the pre-filtered gene universe.
- Statistical markers reflect expression profiles only at this stage; LR interactions identified this way are not yet filtered by activated pathways in receiver cells.

## Evidence

- [other] article intensity/p-value formula: [evidence span withheld — anchor status: not_found]
