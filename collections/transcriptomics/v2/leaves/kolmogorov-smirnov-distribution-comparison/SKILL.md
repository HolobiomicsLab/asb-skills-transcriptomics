---
name: kolmogorov-smirnov-distribution-comparison
description: Use when benchmarking a synthetic-data simulator against real single-cell
  data by testing whether distributions of per-peak and per-cell summary statistics
  (mean, variance, CV, zero proportion, library size) deviate from the real matrix.
when_to_use_negative:
- Data are not count matrices over a shared feature space — KS tests here compare
  peak-wise/cell-wise statistic distributions across real and synthetic matrices over
  the same peak set, not raw sequences or per-read qualities
- A single simulated dataset already has paired observations with real cells, where
  paired tests (not two-sample KS) would be appropriate
- The goal is testing statistical significance of differential accessibility rather
  than simulator fidelity assessment
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3173
tools:
- name: scReadSim
  role: Source of synthetic reads (BAM) whose count-matrix distributions are KS-tested
    against real data; trained via scDesign2 on trustworthy peak-by-cell counts
  repo: https://github.com/JSB-UCLA/scReadSim
- name: SCAN-ATAC-Sim
  role: Baseline read simulator (default parameters) whose output BED-derived count
    matrix provides the comparison KS distributions
- name: MACS3
  role: Calls trustworthy peaks at q-value 0.01 (via scReadSim's Utility.CallPeak)
    defining the shared feature space for count matrices being compared
  repo: https://github.com/macs3-project/MACS
- name: scDesign2
  role: Count simulator that scReadSim trains on the trustworthy peak-by-cell count
    matrix to generate realistic synthetic counts before read simulation
  repo: https://github.com/JSB-UCLA/scDesign2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:79ceee84f1c61acf9cc28af720e8241e3886d573a1bf59bca5b8433b3e31e22f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - MACS3
  - SCAN-ATAC-Sim
  - scDesign2
  - scReadSim
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# kolmogorov-smirnov-distribution-comparison

## Summary

Two-sample Kolmogorov-Smirnov (KS) tests quantify how closely synthetic scATAC-seq read-count distributions match a real count matrix, complementing UMAP-based miLISI mixing scores in benchmarking read simulators like scReadSim against SCAN-ATAC-Sim.

## When to use

Use when benchmarking a synthetic-data simulator against real single-cell data by testing whether distributions of per-peak and per-cell summary statistics (mean, variance, CV, zero proportion, library size) deviate from the real matrix.

## When NOT to use

- Data are not count matrices over a shared feature space — KS tests here compare peak-wise/cell-wise statistic distributions across real and synthetic matrices over the same peak set, not raw sequences or per-read qualities
- A single simulated dataset already has paired observations with real cells, where paired tests (not two-sample KS) would be appropriate
- The goal is testing statistical significance of differential accessibility rather than simulator fidelity assessment

## Inputs

- Real peak-by-cell read count matrix (e.g. sci-ATAC-seq, counted over MACS3 q=0.01 trustworthy peaks)
- Synthetic peak-by-cell count matrix derived from scReadSim output BAM file
- Synthetic peak-by-cell count matrix derived from SCAN-ATAC-Sim output BED file
- Trustworthy peak set (MACS3 q=0.01) and non-peak complement (complement of q=0.1 peaks)

## Outputs

- KS statistics for six summary statistics per simulator versus real matrix (e.g. scan_atac_sim_comparison_metrics.csv)
- Supplementary plots comparing summary-statistic distributions (e.g. Supplementary Fig. 6a)
- miLISI mixing scores recorded per simulator alongside KS statistics

## How to apply

Construct peak-by-cell read count matrices for the real sci-ATAC-seq data and each simulator's output (scReadSim's BAM file, SCAN-ATAC-Sim's BED file) by counting reads overlapping MACS3-called trustworthy peaks (q-value 0.01) using the complement of relaxed peaks (q-value 0.1) as background. Compute six summary statistics: peak-wise mean, variance, coefficient of variance, and zero proportion, plus cell-wise library size and zero proportion. Run two-sample KS tests of each synthetic matrix's statistic distributions against the real matrix's distributions; smaller KS statistics indicate better distributional fidelity. Report KS statistics per simulator (e.g. in scan_atac_sim_comparison_metrics.csv) alongside miLISI UMAP mixing scores (range 1–2, perfect mixing = 2) so the simulator that both matches summary statistics and mixes with real cells is judged superior.

## Related tools

- **scReadSim** (Source of synthetic reads (BAM) whose count-matrix distributions are KS-tested against real data; trained via scDesign2 on trustworthy peak-by-cell counts) — https://github.com/JSB-UCLA/scReadSim
- **SCAN-ATAC-Sim** (Baseline read simulator (default parameters) whose output BED-derived count matrix provides the comparison KS distributions)
- **MACS3** (Calls trustworthy peaks at q-value 0.01 (via scReadSim's Utility.CallPeak) defining the shared feature space for count matrices being compared) — https://github.com/macs3-project/MACS
- **scDesign2** (Count simulator that scReadSim trains on the trustworthy peak-by-cell count matrix to generate realistic synthetic counts before read simulation) — https://github.com/JSB-UCLA/scDesign2

## Examples

```
ks_statistic <- ks.test(real_stats$peak_mean, scReadSim_stats$peak_mean)$statistic
```

## Evaluation signals

- scReadSim's six summary-statistic distributions show smaller two-sample KS statistics versus the real matrix than SCAN-ATAC-Sim's (Supplementary Fig. 6a)
- scReadSim's synthetic cells achieve higher miLISI (closer to the perfect-mixing value of 2) than SCAN-ATAC-Sim in the shared UMAP embedding (Fig. 2b)
- All three count matrices (real, scReadSim, SCAN-ATAC-Sim) are computed over the identical trustworthy peak set so KS comparisons are like-for-like
- Synthetic simulator generates the same number of synthetic cells as real cells per cell type before computing statistics
- KS statistics and miLISI values are recorded per simulator in scan_atac_sim_comparison_metrics.csv for reproducibility

## Limitations

- SCAN-ATAC-Sim's poorer KS/miLISI performance is expected because it does not learn from real scATAC-seq data — a fair comparison requires noting this asymmetry
- Trustworthy-peak definition (q=0.01 vs q=0.1 complement) affects which features enter the count matrices and thus the KS statistics
- Current scReadSim version is not recommended for benchmarking isoform or alternative-splicing tools, so KS-based fidelity checks do not transfer to isoform-level distributions

## Evidence

- [other] Compute six summary statistics (peak-wise mean, variance, coefficient of variance, zero proportion; cell-wise library size, zero proportion) and perform two-sample Kolmogorov-Smirnov tests of each synthetic matrix against the real matrix.: [evidence span withheld — anchor status: not_found]
- [results] scReadSim outperforms SCAN-ATAC-Sim in mimicking the sci-ATAC-seq data at both the read-count level (six summary statistics) and in cells' UMAP embeddings (Fig. 2b): [evidence span withheld — anchor status: not_found]
- [methods] Call MACS3 peaks from the sci-ATAC-seq BAM file at the stringent q-value threshold 0.01 for trustworthy peaks and derive trustworthy non-peaks as the complement of peaks called at the relaxed q-value threshold 0.1.: [evidence span withheld — anchor status: not_found]
- [methods] scReadSim is judged to outperform SCAN-ATAC-Sim if its summary-statistic distributions show smaller KS statistics versus the real matrix (Supplementary Fig. 6a) and its cells mix with real cells at a higher miLISI, closer to the perfect-mixing value of 2 (Fig. 2b).: [evidence span withheld — anchor status: not_found]
- [methods] As an overview, scReadSim consists of three major steps: genome segmentation, synthetic count generation, and synthetic read generation.: "scReadSim consists of three major steps: genome segmentation, synthetic count generation, and synthetic read generation"
- [readme] Trained on real data, scReadSim generates synthetic sequencing reads in BAM or FASTQ formats.: [evidence span withheld — anchor status: external_doc]
