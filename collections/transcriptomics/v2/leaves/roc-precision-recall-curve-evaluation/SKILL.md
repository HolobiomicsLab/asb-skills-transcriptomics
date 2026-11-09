---
name: roc-precision-recall-curve-evaluation
description: Use when benchmarking scATAC-seq peak callers (e.g. MACS3, HOMER, SEACR,
  HMMRATAC) on synthetic reads that come with designed ground-truth peaks, so that
  varying thresholds can sweep TPR/FPR and precision/recall.
when_to_use_negative:
- Benchmarking scRNA-seq UMI deduplication tools — that benchmark uses the synthetic
  UMI count matrix as ground truth, not peak overlaps
- Evaluating isoform or alternative splicing tools — scReadSim explicitly does not
  support isoform-level ground truths
- Datasets lacking designed ground-truth peaks and non-peaks; the overlap-based TP/FP
  classification cannot be computed
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: MACS3
  role: Peak caller benchmarked in -BAMPE mode with q-value 0.05; also invoked by
    scReadSim via Utility.CallPeak
  repo: https://github.com/macs3-project/MACS
- name: HOMER
  role: Peak caller benchmarked in -region mode with -minDist 150
- name: SEACR
  role: Peak caller benchmarked in -stringent mode with control threshold 0.5
- name: HMMRATAC
  role: Peak caller benchmarked with default settings; top performer (F1 87.3%)
- name: Intervene
  role: Python package used to cross-compare the four called peak sets with Venn diagrams
    and upset plots
- name: scReadSim
  role: Generates the synthetic scATAC-seq reads and designed ground-truth peaks that
    serve as the benchmark target
  repo: https://github.com/JSB-UCLA/scReadSim
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:aecdc77c5b56ca694e31bac88626de51f2354a6e40e834f3c496a41c5c9b2765
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/macs3-project/MACS
  tools:
  - HMMRATAC
  - HOMER
  - Intervene
  - MACS3
  - SEACR
  - scReadSim
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# roc-precision-recall-curve-evaluation

## Summary

Benchmarks scATAC-seq peak-calling tools by sweeping score thresholds to build ROC and precision-recall curves against scReadSim's designed ground-truth peaks, then aggregating recall, precision, F1, and peak-length metrics. This turns qualitative 'top performer' claims into reproducible quantitative evidence.

## When to use

Use when benchmarking scATAC-seq peak callers (e.g. MACS3, HOMER, SEACR, HMMRATAC) on synthetic reads that come with designed ground-truth peaks, so that varying thresholds can sweep TPR/FPR and precision/recall.

## When NOT to use

- Benchmarking scRNA-seq UMI deduplication tools — that benchmark uses the synthetic UMI count matrix as ground truth, not peak overlaps
- Evaluating isoform or alternative splicing tools — scReadSim explicitly does not support isoform-level ground truths
- Datasets lacking designed ground-truth peaks and non-peaks; the overlap-based TP/FP classification cannot be computed

## Inputs

- scReadSim synthetic scATAC-seq BAM file generated from the mouse sci-ATAC-seq dataset with designed ground-truth peaks
- Ground-truth peak BED interval set (non-overlapping [TSS, TSS+LEN-1], LEN 250–550 bp)
- Called peak BED files from MACS3 (-BAMPE, q 0.05), HOMER (-region, -minDist 150), SEACR (-stringent, control threshold 0.5), and HMMRATAC (defaults)
- Per-tool peak scores/thresholds for sweeping TPR, FPR, precision, and recall

## Outputs

- ROC curves and precision-recall curves per peak caller
- peak_calling_benchmark_metrics.csv with per-tool recall, precision, F1, and mean called-peak lengths
- Venn diagram and upset plot cross-comparing the four called peak sets (Intervene)
- RPKM distribution comparisons of called peaks vs ground-truth peaks and called non-peaks vs ground-truth non-peaks

## How to apply

Run each peak caller under its benchmark setting — MACS3 in -BAMPE mode with q-value 0.05, HOMER in -region mode with -minDist 150, SEACR in -stringent mode with control threshold 0.5, HMMRATAC with defaults — on the scReadSim synthetic BAM built from the sci-ATAC-seq dataset (ground-truth peaks spanning [TSS, TSS+LEN-1] with LEN uniform 250–550 bp, merged non-overlapping). Match each called peak to ground truth by overlap, counting a hit only if it covers ≥225 bp of a ground-truth peak (half the shortest ground-truth peak length). Vary each tool's peak-score threshold to compute true positive rate, false positive rate, precision, and recall, and plot ROC and precision-recall curves. Complement the curves with RPKM-distribution comparisons (called peaks vs ground-truth peaks, called non-peaks vs ground-truth non-peaks) and cross-compare the four called peak sets with Venn/upset plots via Intervene. Aggregate per-tool recall, precision, F1, and mean called-peak lengths into peak_calling_benchmark_metrics.csv.

## Related tools

- **MACS3** (Peak caller benchmarked in -BAMPE mode with q-value 0.05; also invoked by scReadSim via Utility.CallPeak) — https://github.com/macs3-project/MACS
- **HOMER** (Peak caller benchmarked in -region mode with -minDist 150)
- **SEACR** (Peak caller benchmarked in -stringent mode with control threshold 0.5)
- **HMMRATAC** (Peak caller benchmarked with default settings; top performer (F1 87.3%))
- **Intervene** (Python package used to cross-compare the four called peak sets with Venn diagrams and upset plots)
- **scReadSim** (Generates the synthetic scATAC-seq reads and designed ground-truth peaks that serve as the benchmark target) — https://github.com/JSB-UCLA/scReadSim

## Examples

```
macs3 callpeak -t synthetic_scATAC.bam -f BAMPE -q 0.05
```

## Evaluation signals

- F1 magnitudes reproduce the benchmark: HMMRATAC 87.3%, MACS3 81.3%, HOMER 70.0%, SEACR 14.3% (Fig. 4e-g)
- HMMRATAC and MACS3 emerge as the top performers, consistent with the article's conclusion
- Every called peak counted as a true positive overlaps ≥225 bp of a ground-truth peak (half the shortest ground-truth peak length)
- ROC and precision-recall curves are monotone in the swept threshold and derived from per-threshold TPR, FPR, precision, and recall values
- peak_calling_benchmark_metrics.csv contains recall, precision, F1, and mean called-peak lengths for all four tools

## Limitations

- SEACR performs poorly (F1 14.3%) on this synthetic benchmark, so rankings are specific to the scReadSim synthetic design and the 225 bp overlap criterion
- scReadSim's default implementation may generate intronic synthetic reads, which can affect downstream benchmarks
- Multi-sample or multi-condition benchmarking requires supplying one BAM file per sample or condition

## Evidence

- [abstract] top performers: "HMMRATAC and MACS3 achieve the top performance in scATAC-seq peak calling"
- [intro] ground truth: "The ground-truth peaks provided by scReadSim allow benchmarking peak-calling tools"
- [methods] synthetic reads: "The third step generates synthetic reads based on the synthetic count matrices, the input BAM file, and the reference genome"
- [readme] read simulator: [evidence span withheld — anchor status: external_doc]
