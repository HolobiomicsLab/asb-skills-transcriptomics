---
name: precision-sensitivity-fscore-computation
description: 'Use when benchmarking HTO-based demultiplexing methods: you have per-droplet
  sample-of-origin/multiplet calls from one or more methods plus an independent ground
  truth (demuxlet-derived SSDs or RNA-profile cluster labels) and need precisionSSD,
  sensitivitySSD, and F-score averaged across conditions.'
when_to_use_negative:
- The input is a single experimental dataset with no independent ground-truth labels
  — precision/sensitivity cannot be computed without reference calls; use demuxmix's
  estimated demultiplexing error rates and QC methods instead
- The goal is demultiplexing itself rather than benchmarking — run demuxmix directly
  and use its error probabilities to discard uncertain droplets rather than computing
  classification metrics
- Evaluating RNA-level cell type assignment or clustering accuracy rather than singlet/doublet/sample-of-origin
  classification of HTO-labeled droplets
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: demuxmix
  role: Benchmarked method producing probabilistic droplet classifications (including
    naïve mode omitting the gene-count covariate) whose assignments are scored against
    ground truth
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: HTODemux
  role: Competing demultiplexing method run with default settings on the same attenuated
    datasets to obtain comparison assignments
- name: hashedDrops
  role: Competing demultiplexing method run with default settings on the same attenuated
    datasets
- name: MULTI-seq
  role: Competing demultiplexing method run with default settings on the same attenuated
    datasets
- name: GMM-Demux
  role: Competing demultiplexing method run with default settings on the same attenuated
    datasets
- name: DemuxEM
  role: Competing demultiplexing method run with default settings on the same attenuated
    datasets
- name: demuxlet
  role: Provides demuxlet-derived high-confidence SSD calls used as ground truth for
    computing precisionSSD and sensitivitySSD
- name: R
  role: Implementation environment for computing metrics and writing the benchmark
    tables (R/Bioconductor package demuxmix)
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:e2c96d67e4afb4a990bcc578c79807af3c1b6f0b18bf1127f180c7372fe32c04
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DemuxEM
  - GMM-Demux
  - HTODemux
  - MULTI-seq
  - R
  - demuxlet
  - demuxmix
  - hashedDrops
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# precision-sensitivity-fscore-computation

## Summary

Computes precisionSSD, sensitivitySSD, and F-score by comparing per-droplet demultiplexing assignments (singlet/doublet/multiplet calls) against a ground-truth label set, then averages metrics across simulated attenuation conditions to benchmark methods like demuxmix against HTODemux, hashedDrops, MULTI-seq, GMM-Demux, and DemuxEM.

## When to use

Use when benchmarking HTO-based demultiplexing methods: you have per-droplet sample-of-origin/multiplet calls from one or more methods plus an independent ground truth (demuxlet-derived SSDs or RNA-profile cluster labels) and need precisionSSD, sensitivitySSD, and F-score averaged across conditions.

## When NOT to use

- The input is a single experimental dataset with no independent ground-truth labels — precision/sensitivity cannot be computed without reference calls; use demuxmix's estimated demultiplexing error rates and QC methods instead
- The goal is demultiplexing itself rather than benchmarking — run demuxmix directly and use its error probabilities to discard uncertain droplets rather than computing classification metrics
- Evaluating RNA-level cell type assignment or clustering accuracy rather than singlet/doublet/sample-of-origin classification of HTO-labeled droplets

## Inputs

- Per-droplet demultiplexing assignments (sample-of-origin and singlet/doublet/multiplet class) from each method being benchmarked
- Ground-truth per-droplet SSD labels (demuxlet-derived high-confidence singlet/doublet calls, or sample-of-origin labels derived from RNA-profile clustering)
- Attenuated (HTOs x droplets) HTO count matrices at multiple scaling factors s, with artificially merged doublets added

## Outputs

- Per-method, per-condition precisionSSD, sensitivitySSD, and F-score values
- Averaged (across attenuation conditions) benchmark metrics table, e.g., table1_benchmark_metrics.csv / table2_demultiplexing_benchmark.csv

## How to apply

Run each demultiplexing method (demuxmix, HTODemux, hashedDrops, MULTI-seq, GMM-Demux, DemuxEM) on the same (HTOs x droplets) count matrix, then compare each method's droplet assignments droplet-by-droplet to the ground-truth singlet/doublet (SSD) labels. Compute precisionSSD (fraction of predicted SSD-classified droplets that are true SSDs), sensitivitySSD (fraction of true SSDs recovered), and F-score per condition. When simulating reduced oligonucleotide labeling, attenuate the HTO matrix by scaling factors (e.g., s = 1 down to 0.1 or 0.25) and add artificial doublets (e.g., 200 merged doublets or 350 doublet pairs), computing metrics at each attenuation level. Average precisionSSD, sensitivitySSD, and F-score across conditions per method and assemble them into a comparison table (e.g., table1_benchmark_metrics.csv, table2_demultiplexing_benchmark.csv). Validate against reported values — e.g., demuxmix discarding only 6.3% of true SSDs as uncertain/negative/MSMs versus up to 12.8% for other methods, and 85% recovered SSDs versus 67%-81% for competitors.

## Related tools

- **demuxmix** (Benchmarked method producing probabilistic droplet classifications (including naïve mode omitting the gene-count covariate) whose assignments are scored against ground truth) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **HTODemux** (Competing demultiplexing method run with default settings on the same attenuated datasets to obtain comparison assignments)
- **hashedDrops** (Competing demultiplexing method run with default settings on the same attenuated datasets)
- **MULTI-seq** (Competing demultiplexing method run with default settings on the same attenuated datasets)
- **GMM-Demux** (Competing demultiplexing method run with default settings on the same attenuated datasets)
- **DemuxEM** (Competing demultiplexing method run with default settings on the same attenuated datasets)
- **demuxlet** (Provides demuxlet-derived high-confidence SSD calls used as ground truth for computing precisionSSD and sensitivitySSD)
- **R** (Implementation environment for computing metrics and writing the benchmark tables (R/Bioconductor package demuxmix)) — https://doi.org/doi:10.18129/B9.bioc.demuxmix

## Examples

```
BiocManager::install("demuxmix"); library(demuxmix)  # run demuxmix(demuxmix(htoData, rnaData=genesPerDroplet)), compare assignments to demuxlet SSD ground truth, compute precisionSSD/sensitivitySSD/F-score per attenuation level, then average into table1_benchmark_metrics.csv
```

## Evaluation signals

- Reproduced average precisionSSD, sensitivitySSD, and F-score per method must match the values reported in Table 1 of the article (human brain benchmark)
- Reproduced metrics must match Table 2: demuxmix attains approximately 85% average precisionSSD while competing methods fall in the 67%-81% range (cell line mixture benchmark)
- Droplet discard rate check: demuxmix should discard only 6.3% of true SSDs as uncertain, negative, or MSMs versus up to 12.8% for other methods on the simulated human brain data
- Per-condition metrics must be computed at every attenuation level (s = 1 down to 0.1 for human brain; s = 1 down to 0.25 for cell line mixture) before averaging
- Output table schema check: one row per method with averaged precisionSSD, sensitivitySSD, and F-score columns, written as the target CSV

## Limitations

- Metrics depend on ground-truth quality: demuxlet-derived high-confidence SSDs (human brain) or RNA-profile cluster labels (cell line mixture) are used as proxies for truth and may themselves be imperfect
- Simulated attenuation with artificial doublets (200 merged doublets per level; 350 doublet pairs) approximates but does not fully reproduce reduced oligonucleotide labeling and multiplet presence
- Droplets classified as uncertain, negative, or MSMs are discarded before scoring, so discard rates (up to 12.8% for non-demuxmix methods) directly affect apparent sensitivity

## Evidence

- [other] Compare each method's droplet assignments to the demuxlet SSD ground truth, computing precisionSSD, sensitivitySSD, and F-score per condition: [evidence span withheld — anchor status: not_found]
- [other] Table 1 validation: [evidence span withheld — anchor status: not_found]
- [other] Table 2 validation: [evidence span withheld — anchor status: not_found]
- [results] Findings on demuxmix performance: [evidence span withheld — anchor status: not_found]
- [abstract] Error probabilities for QC: "demuxmix's probabilistic classification framework provides error probabilities for droplet assignments that can be used to discard uncertain droplets"
- [readme] README installation: [evidence span withheld — anchor status: not_found]
