---
name: scatac-seq-peak-calling-benchmarking
description: Use when benchmarking scATAC-seq peak callers against a known set of
  ground-truth peaks (e.g. scReadSim synthetic BAM from the sci-ATAC-seq dataset),
  or when choosing a peak caller for single-cell ATAC data.
when_to_use_negative:
- Benchmarking scRNA-seq UMI deduplication tools — use the scReadSim synthetic UMI
  count matrix ground truth instead
- The tool under test is an isoform-level or alternative-splicing method — scReadSim
  currently provides only gene-level/peak-level ground truths, not isoform abundance
- Input data lacks ground-truth peaks/non-peaks (real data without designed truth)
  — recall/precision/F1 cannot be computed, only qualitative cross-tool overlap
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3168
tools:
- name: scReadSim
  role: Generates synthetic scATAC-seq reads with designed ground-truth peaks and
    non-peaks for benchmarking
  repo: https://github.com/JSB-UCLA/scReadSim
- name: MACS3
  role: Peak caller run in -BAMPE mode with q-value threshold 0.05; top performer
    (F1 81.3%); also used internally by scReadSim via Utility.CallPeak
  repo: https://github.com/macs3-project/MACS
- name: HOMER
  role: Peak caller run in -region mode with -minDist 150; F1 70.0% in benchmark
- name: SEACR
  role: Peak caller run in -stringent mode with control threshold 0.5; F1 14.3% in
    benchmark
- name: HMMRATAC
  role: Peak caller run with default settings; top performer with F1 87.3% in benchmark
- name: Intervene
  role: Python package used to cross-compare the four called peak sets with Venn diagrams
    and upset plots
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:4721749197c042a7315b1cbe04b43544a286bbc1b9f97e50c51342d4eb63bebd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - HMMRATAC
  - HOMER
  - Intervene
  - MACS3
  - SEACR
  - scReadSim
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# scatac-seq-peak-calling-benchmarking

## Summary

Benchmarks scATAC-seq peak-calling tools (MACS3, HOMER, SEACR, HMMRATAC) on scReadSim synthetic reads whose designed ground-truth peaks and non-peaks allow exact recall, precision, and F1 computation. HMMRATAC and MACS3 emerge as top performers in the scReadSim benchmark.

## When to use

Use when benchmarking scATAC-seq peak callers against a known set of ground-truth peaks (e.g. scReadSim synthetic BAM from the sci-ATAC-seq dataset), or when choosing a peak caller for single-cell ATAC data.

## When NOT to use

- Benchmarking scRNA-seq UMI deduplication tools — use the scReadSim synthetic UMI count matrix ground truth instead
- The tool under test is an isoform-level or alternative-splicing method — scReadSim currently provides only gene-level/peak-level ground truths, not isoform abundance
- Input data lacks ground-truth peaks/non-peaks (real data without designed truth) — recall/precision/F1 cannot be computed, only qualitative cross-tool overlap

## Inputs

- scReadSim synthetic scATAC-seq BAM file generated from the mouse sci-ATAC-seq dataset with designed ground-truth peaks
- Ground-truth peak and non-peak region coordinates (BED-style [TSS, TSS+LEN-1] intervals, LEN 250-550 bp, non-overlapping)
- mm10 reference genome and chromosome size file

## Outputs

- Per-tool recall, precision, F1, and mean called-peak lengths aggregated into peak_calling_benchmark_metrics.csv
- ROC and precision-recall curves for each peak caller
- RPKM distribution comparisons (called vs ground-truth peaks and non-peaks) with mean differences
- Venn diagram and upset plot of the four called peak sets (via Intervene)

## How to apply

Simulate synthetic scATAC-seq reads with scReadSim from a real BAM (e.g. the sci-ATAC-seq dataset), with designed ground-truth peaks defined as [TSS, TSS+LEN-1], LEN uniformly sampled from 250 to 550 bp, merged to be non-overlapping. Run each peak caller with its benchmark setting: MACS3 in -BAMPE mode with q-value 0.05, HOMER in -region mode with -minDist 150, SEACR in -stringent mode with control threshold 0.5, and HMMRATAC with defaults. Match called peaks to ground truth by overlap, counting a called peak as correct if it overlaps at least 225 bp of any ground-truth peak (half the shortest ground-truth peak length). Sweep thresholds per tool to build ROC and precision-recall curves, compare RPKM distributions of called vs ground-truth peaks and non-peaks via mean difference, and cross-compare peak sets with Venn/upset plots using the Python package Intervene.

## Related tools

- **scReadSim** (Generates synthetic scATAC-seq reads with designed ground-truth peaks and non-peaks for benchmarking) — https://github.com/JSB-UCLA/scReadSim
- **MACS3** (Peak caller run in -BAMPE mode with q-value threshold 0.05; top performer (F1 81.3%); also used internally by scReadSim via Utility.CallPeak) — https://github.com/macs3-project/MACS
- **HOMER** (Peak caller run in -region mode with -minDist 150; F1 70.0% in benchmark)
- **SEACR** (Peak caller run in -stringent mode with control threshold 0.5; F1 14.3% in benchmark)
- **HMMRATAC** (Peak caller run with default settings; top performer with F1 87.3% in benchmark)
- **Intervene** (Python package used to cross-compare the four called peak sets with Venn diagrams and upset plots)

## Examples

```
macs3 callpeak -t synthetic.bam -f BAMPE -q 0.05 -n macs3_peaks
```

## Evaluation signals

- HMMRATAC F1 ≈ 87.3% and MACS3 F1 ≈ 81.3% should reproduce the Fig. 4e-g benchmark magnitudes, with HOMER ≈ 70.0% and SEACR ≈ 14.3%
- Each called peak overlapping at least 225 bp of a ground-truth peak (half the shortest 250 bp ground-truth peak) is counted as a true positive
- RPKM distributions of called peaks should resemble ground-truth peak distributions (small mean difference); called non-peaks should resemble ground-truth non-peaks
- Precision-recall and ROC curves should be computed by sweeping thresholds over each tool's called peaks
- Cross-tool overlap of the four called peak sets visualized via Intervene Venn/upset plots should show HMMRATAC and MACS3 concordance

## Limitations

- The default implementation of scReadSim may generate intronic synthetic reads when extracting sequences from the reference genome, which can affect benchmarks such as cellranger default mode
- Multi-sample or multi-condition benchmarking requires supplying one BAM file per sample or condition
- Current scReadSim lacks isoform-level ground truths, so benchmarking of isoform/alternative-splicing tools is not supported

## Evidence

- [methods] F1 scores show HMMRATAC outperforms MACS3, HOMER, and SEACR: [evidence span withheld — anchor status: not_found]
- [methods] Ground-truth peak design: [evidence span withheld — anchor status: not_found]
- [methods] Peak matching rule: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract finding: "HMMRATAC and MACS3 achieve the top performance in scATAC-seq peak calling"
- [abstract] Ground-truth availability: "scReadSim provides ground truths, including unique molecular identifier (UMI) counts for"
- [methods] Internal MACS3 usage: "scReadSim utilizes MACS3 through function `Utility.CallPeak` with following arguments"
- [intro] ReadSim objective: "The ground-truth peaks provided by scReadSim allow benchmarking peak-calling tools"
