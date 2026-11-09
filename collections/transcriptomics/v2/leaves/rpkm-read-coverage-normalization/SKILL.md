---
name: rpkm-read-coverage-normalization
description: Use when benchmarking scATAC-seq peak callers on scReadSim synthetic
  reads and you need to compare read enrichment of called peaks vs ground-truth peaks
  (or non-peaks) in a length-normalized way via RPKM distributions.
when_to_use_negative:
- The task is isoform-level or alternative-splicing benchmarking — the article states
  the current scReadSim provides gene-level ground truths only and is not recommended
  for such tools
- Peak sets are already length- and depth-normalized (e.g., RPKM/CPM columns exist)
  — re-normalizing would double-correct
- No ground-truth peak/non-peak intervals are available, so the called-vs-ground-truth
  RPKM mean-difference comparison cannot be constructed
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: scReadSim
  role: Generates the synthetic scATAC-seq BAM reads and the designed ground-truth
    peaks/non-peaks against which RPKM distributions are compared
  repo: https://github.com/JSB-UCLA/scReadSim
- name: MACS3
  role: Peak caller run in -BAMPE mode (q 0.05); also invoked internally by scReadSim
    via Utility.CallPeak
  repo: https://github.com/macs3-project/MACS
- name: HOMER
  role: Peak caller run in -region mode with -minDist 150 producing called peaks for
    RPKM/threshold evaluation
- name: SEACR
  role: Peak caller run in -stringent mode with control threshold 0.5 producing called
    peaks for RPKM/threshold evaluation
- name: HMMRATAC
  role: Peak caller run with default settings; top performer (F1 87.3%) in the RPKM-based
    benchmark
- name: Intervene
  role: Python package used for Venn diagram and upset plot cross-comparison of the
    four called peak sets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:603037a22c5b0971361c096744a1f5718cc7bd525f1ba1f9784c393a8951098c
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# rpkm-read-coverage-normalization

## Summary

Length- and library-size-normalized read coverage (RPKM) computation for comparing called peak sets against scReadSim-designed ground-truth peaks/non-peaks in the scATAC-seq peak-calling benchmark. It enables the mean-difference RPKM comparison (Fig. 4) that distinguishes top performers (HMMRATAC, MACS3) from weak ones (SEACR).

## When to use

Use when benchmarking scATAC-seq peak callers on scReadSim synthetic reads and you need to compare read enrichment of called peaks vs ground-truth peaks (or non-peaks) in a length-normalized way via RPKM distributions.

## When NOT to use

- The task is isoform-level or alternative-splicing benchmarking — the article states the current scReadSim provides gene-level ground truths only and is not recommended for such tools
- Peak sets are already length- and depth-normalized (e.g., RPKM/CPM columns exist) — re-normalizing would double-correct
- No ground-truth peak/non-peak intervals are available, so the called-vs-ground-truth RPKM mean-difference comparison cannot be constructed

## Inputs

- scReadSim synthetic scATAC-seq BAM file generated from the mouse sci-ATAC-seq dataset (mm10 reference)
- Ground-truth peak BED intervals ([TSS, TSS+LEN-1], 250-550 bp, merged non-overlapping) and ground-truth non-peak intervals
- Called peak BED files from MACS3, HOMER, SEACR, and HMMRATAC
- Chromosome size file (embedded in scReadSim for mm10)

## Outputs

- RPKM values per called peak, ground-truth peak, called non-peak, and ground-truth non-peak
- Mean RPKM difference between called and ground-truth (non-)peak distributions
- ROC and precision-recall curves from threshold sweeps (TPR, FPR, precision, recall)
- peak_calling_benchmark_metrics.csv with per-tool recall, precision, F1, and mean called-peak lengths

## How to apply

After each peak caller (MACS3 in -BAMPE mode with q-value 0.05, HOMER -region with -minDist 150, SEACR stringent with control threshold 0.5, HMMRATAC default) is run on the scReadSim synthetic BAM mimicking the sci-ATAC-seq dataset, count reads overlapping each called peak and each ground-truth peak ([TSS, TSS+LEN-1], LEN ~ Uniform(250, 550), non-overlapping). Convert counts to RPKM by dividing by peak length in kb and by total mapped reads per million, so peaks of unequal length and libraries of unequal depth are comparable. Compute the RPKM distribution separately for called peaks vs ground-truth peaks and for called non-peaks vs ground-truth non-peaks, then compute the mean difference between the distributions as the enrichment signal. Pair this with threshold sweeps on each tool's peaks to derive true positive rate, false positive rate, precision, and recall, scoring a called peak correct if it overlaps at least 225 bp (half the shortest 250 bp ground-truth peak) of any ground-truth peak. Aggregate per-tool recall, precision, F1, and mean called-peak lengths into peak_calling_benchmark_metrics.csv for validation against Fig. 4e-g.

## Related tools

- **scReadSim** (Generates the synthetic scATAC-seq BAM reads and the designed ground-truth peaks/non-peaks against which RPKM distributions are compared) — https://github.com/JSB-UCLA/scReadSim
- **MACS3** (Peak caller run in -BAMPE mode (q 0.05); also invoked internally by scReadSim via Utility.CallPeak) — https://github.com/macs3-project/MACS
- **HOMER** (Peak caller run in -region mode with -minDist 150 producing called peaks for RPKM/threshold evaluation)
- **SEACR** (Peak caller run in -stringent mode with control threshold 0.5 producing called peaks for RPKM/threshold evaluation)
- **HMMRATAC** (Peak caller run with default settings; top performer (F1 87.3%) in the RPKM-based benchmark)
- **Intervene** (Python package used for Venn diagram and upset plot cross-comparison of the four called peak sets)

## Examples

```
RPKM = peak_counts / (peak_len_bp / 1000) / (total_mapped_reads / 1e6); mean_diff = called_RPKM.mean() - truth_RPKM.mean()
```

## Evaluation signals

- Mean RPKM of called peaks should closely match the mean RPKM of ground-truth peaks for accurate callers; mean called non-peak RPKM should approach ground-truth non-peak RPKM
- F1 magnitudes should reproduce Fig. 4e-g ordering: HMMRATAC 87.3% > MACS3 81.3% > HOMER 70.0% > SEACR 14.3%
- Every called peak scored correct must overlap >= 225 bp of at least one ground-truth peak (half of the shortest 250 bp ground-truth peak)
- peak_calling_benchmark_metrics.csv contains non-null recall, precision, F1, and mean called-peak length for all four tools
- Ground-truth peak lengths fall within the 250-550 bp design range and are non-overlapping after merging

## Limitations

- SEACR performed poorly on this benchmark (F1 14.3%), so its called-peak RPKM distributions deviate strongly from ground truth even at matched thresholds
- Default scReadSim may generate intronic synthetic reads when extracting sequences from the reference genome, which can affect some benchmark configurations
- Called peak 'correctness' is defined by a 225 bp overlap cutoff, which is sensitive to the 250-550 bp ground-truth length distribution; shorter ground-truth peaks are harder to match

## Evidence

- [methods] RPKM distribution mean-difference validation: [evidence span withheld — anchor status: not_found]
- [results] F1 benchmark ordering: [evidence span withheld — anchor status: not_found]
- [methods] Peak-matching cutoff: [evidence span withheld — anchor status: not_found]
- [abstract] Top-performer finding: "HMMRATAC and MACS3 achieve the top performance in scATAC-seq peak calling"
