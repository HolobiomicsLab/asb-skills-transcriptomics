---
name: artificial-bulk-rnaseq-construction
description: Use when benchmarking bulk tissue cell type deconvolution with a single-cell
  reference and no gold-standard bulk samples with known cell type proportions are
  available, as in the Segerstolpe pancreatic islet setup.
when_to_use_negative:
- When real bulk RNA-seq samples with independently measured cell type proportions
  are available — those are a better benchmark than artificial bulk built from the
  same study.
- When the goal is to test robustness to cross-study batch effects, since bulk and
  reference derived from the same scRNA-seq study remove that difficulty.
- When input data are normalized values (e.g., TPM) rather than read counts — cell
  type proportions cannot be estimated with TPM as the input, so artificial bulk must
  be built from raw counts.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: MuSiC
  role: bulk_construct builds the artificial bulk ExpressionSet; relative.ab computes
    the true proportions; music_prop and Eval_multi consume the artificial bulk for
    estimation and scoring
  repo: https://github.com/xuranw/MuSiC
- name: BSEQ-sc
  role: alternative deconvolution method applied to the artificial bulk (via bseqsc_basis/bseqsc_proportions)
    for benchmark comparison
- name: CIBERSORT
  role: existing deconvolution baseline benchmarked against MuSiC on the artificial
    bulk data
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:c608b87c2f6bf1f03e699b3fd66a8885bb16981a3e3d58a7fb1c70ac259c016f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/MuSiC
  tools:
  - BSEQ-sc
  - CIBERSORT
  - MuSiC
  - R
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# artificial-bulk-rnaseq-construction

## Summary

Construct an artificial bulk RNA-seq benchmark by summing scRNA-seq counts per subject (bulk_construct) so that true cell type proportions are known, enabling quantitative evaluation of deconvolution methods like MuSiC, NNLS, and BSEQ-sc.

## When to use

Use when benchmarking bulk tissue cell type deconvolution with a single-cell reference and no gold-standard bulk samples with known cell type proportions are available, as in the Segerstolpe pancreatic islet setup.

## When NOT to use

- When real bulk RNA-seq samples with independently measured cell type proportions are available — those are a better benchmark than artificial bulk built from the same study.
- When the goal is to test robustness to cross-study batch effects, since bulk and reference derived from the same scRNA-seq study remove that difficulty.
- When input data are normalized values (e.g., TPM) rather than read counts — cell type proportions cannot be estimated with TPM as the input, so artificial bulk must be built from raw counts.

## Inputs

- Single-cell read-count ExpressionSet object (e.g., XinT2Deset.rds / GSE81608, or EMTABesethealthy.rds / E-MTAB-5061)
- Cell type annotation column ('cellType' with types such as alpha, beta, gamma, delta)
- Subject/sample grouping column ('SubjectName')

## Outputs

- Artificial bulk count matrix (Bulk.counts from bulk_construct) with one column per subject
- Real cell type proportion table (prop.real via relative.ab) used as deconvolution ground truth
- Per-method performance metrics (RMSD, mAD, Pearson R) from Eval_multi, e.g., benchmark_evaluation_metrics.csv

## How to apply

Load the scRNA-seq ExpressionSet (e.g., XinT2Deset.rds from GSE81608 with 39849 genes and 1492 cells) and aggregate read counts per subject using bulk_construct with clusters='cellType' and samples='SubjectName' to emulate a bulk tissue profile where the bulk and reference share the same study. Compute the ground-truth proportions with relative.ab on the per-subject sums (by.col=FALSE) for the target cell types (alpha, beta, gamma, delta). This artificial bulk can then be fed to music_prop (Est.prop.weighted and Est.prop.allgene), bseqsc_basis/bseqsc_proportions, or other deconvolution methods, and estimates are scored against the known proportions with Eval_multi. Because the true proportions are derived from the same single-cell data as the reference, the benchmark isolates method accuracy from cross-dataset batch effects, which is why MuSiC's accuracy here (RMSD 0.09881, mAD 0.06357, R 0.9378) is an upper bound relative to fully cross-study application.

## Related tools

- **MuSiC** (bulk_construct builds the artificial bulk ExpressionSet; relative.ab computes the true proportions; music_prop and Eval_multi consume the artificial bulk for estimation and scoring) — https://github.com/xuranw/MuSiC
- **BSEQ-sc** (alternative deconvolution method applied to the artificial bulk (via bseqsc_basis/bseqsc_proportions) for benchmark comparison)
- **CIBERSORT** (existing deconvolution baseline benchmarked against MuSiC on the artificial bulk data)

## Examples

```
XinT2D.construct.full = bulk_construct(XinT2D.eset, clusters='cellType', samples='SubjectName'); XinT2D.construct.full$prop.real = relative.ab(XinT2D.construct.full$num.real, by.col = FALSE)
```

## Evaluation signals

- MuSiC on the Segerstolpe-based artificial bulk should reproduce the reported metrics: RMSD 0.09881, mAD 0.06357, R 0.9378 (NNLS RMSD 0.17161, R 0.8159; BSEQ-sc RMSD 0.21741, R 0.7880).
- The columns of the artificial bulk matrix must correspond one-to-one with subjects (SubjectName) in the ExpressionSet.
- prop.real from relative.ab must sum to 1 across the selected cell types (alpha, beta, delta, gamma) for each subject.
- Estimated proportions (Est.prop.weighted, Est.prop.allgene) must be non-negative and sum to approximately 1 per bulk sample.
- MuSiC's RMSD/mAD should be lower (and R higher) than NNLS and BSEQ-sc on this same-study benchmark.

## Limitations

- Artificial bulk and reference from the same study (e.g., both Segerstolpe) means the benchmark does not capture cross-study batch effects; accuracy is optimistic.
- scRNA-seq-based bulk construction depends on the reference reflecting cell-type-specific expression, not true tissue composition — most scRNA-seq data do not reflect true cell type proportions in intact tissues.
- Construction must use raw read counts; cell type proportions cannot be estimated with TPM as the input, and RPKM requires external cell-type-specific total RNA abundance estimates.
- Collinearity among closely related cell types limits how well rare cell types can be resolved, even in the artificial bulk setting.

## Evidence

- [methods] The artificial bulk data is constructed through function `bulk_construct`: "The artificial bulk data is constructed through function `bulk_construct`"
- [methods] XinT2D.construct.full$prop.real = relative.ab(XinT2D.construct.full$num.real, by.col = FALSE): "XinT2D.construct.full$prop.real = relative.ab(XinT2D.construct.full$num.real, by.col = FALSE)"
- [methods] Benchmark dataset is constructed by summing up single cell data from `XinT2D.eset`.: "Benchmark dataset is constructed by summing up single cell data from `XinT2D.eset`."
- [methods] There are 39849 genes and 1492 cells.: [evidence span withheld — anchor status: not_found]
- [discussion] However, cell type proportions cannot be estimated with TPM as the input.: "However, cell type proportions cannot be estimated with TPM as the input."
