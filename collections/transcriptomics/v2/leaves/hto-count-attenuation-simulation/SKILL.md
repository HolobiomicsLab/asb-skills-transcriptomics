---
name: hto-count-attenuation-simulation
description: Use when benchmarking or comparing HTO-based demultiplexing methods on
  snRNA-seq/scRNA-seq data, when real ground truth is unavailable, or when you need
  to simulate reduced oligonucleotide labeling and increased multiplet rates by attenuating
  HTO counts.
when_to_use_negative:
- The goal is demultiplexing a new real experiment rather than method benchmarking
  — use a single demultiplexing method (e.g. demuxmix) directly instead of simulating
  attenuated datasets
- The experiment lacks HTO/oligonucleotide labeling or uses genotype-based demultiplexing
  only (e.g. demuxlet on natural genetic variants), so HTO count attenuation is not
  applicable
- Ground-truth sample-of-origin labels cannot be derived (no demuxlet high-confidence
  SSDs and no clusterable RNA profiles), leaving no reference to compute precisionSSD/sensitivitySSD
  against
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2269
tools:
- name: demuxmix
  role: Reference demultiplexing method (negative binomial regression mixture models,
    default and naïve mode) evaluated on attenuated HTO datasets; also the package
    providing the analysis framework
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: HTODemux
  role: Competing demultiplexing method run with default settings on each simulated
    dataset
- name: hashedDrops
  role: Competing demultiplexing method run with default settings on each simulated
    dataset
- name: MULTI-seq
  role: Competing demultiplexing method run with default settings on each simulated
    dataset
- name: GMM-Demux
  role: Competing demultiplexing method run with default settings on each simulated
    dataset
- name: DemuxEM
  role: Competing demultiplexing method run with default settings on each simulated
    dataset
- name: demuxlet
  role: Source of high-confidence singlet/doublet ground-truth labels for the human
    brain benchmark
- name: R
  role: Implementation environment (R/Bioconductor) in which the attenuation, doublet
    simulation, and metric computation are carried out
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:e7e01389999bd72f437351a859b5c28035d456f8233ed3c023e083d25b2d48ee
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
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# hto-count-attenuation-simulation

## Summary

A benchmarking protocol that simulates challenging HTO demultiplexing scenarios by scaling HTO count matrices with attenuation factors (s = 1 down to 0.1 or 0.25) and injecting artificial doublets, then scoring demultiplexing methods (demuxmix, HTODemux, hashedDrops, MULTI-seq, GMM-Demux, DemuxEM) against ground-truth sample-of-origin labels using precisionSSD, sensitivitySSD, and F-score.

## When to use

Use when benchmarking or comparing HTO-based demultiplexing methods on snRNA-seq/scRNA-seq data, when real ground truth is unavailable, or when you need to simulate reduced oligonucleotide labeling and increased multiplet rates by attenuating HTO counts.

## When NOT to use

- The goal is demultiplexing a new real experiment rather than method benchmarking — use a single demultiplexing method (e.g. demuxmix) directly instead of simulating attenuated datasets
- The experiment lacks HTO/oligonucleotide labeling or uses genotype-based demultiplexing only (e.g. demuxlet on natural genetic variants), so HTO count attenuation is not applicable
- Ground-truth sample-of-origin labels cannot be derived (no demuxlet high-confidence SSDs and no clusterable RNA profiles), leaving no reference to compute precisionSSD/sensitivitySSD against

## Inputs

- (HTOs x droplets) HTO count matrix from a hashtag-oligonucleotide-labeled single-cell experiment
- (genes x droplets) RNA read count matrix or per-droplet number of detected genes
- Ground-truth sample-of-origin labels: demuxlet-derived high-confidence singlet/doublet (SSD) calls (human brain benchmark) or RNA-profile cluster labels (cell line mixture, GSM3501446/GSM3501447)
- Attenuation factor grid (e.g. s = 1 to 0.1 or 1 to 0.25) and target number of artificial doublets (e.g. 200 or 350)

## Outputs

- Per-method, per-attenuation-level precisionSSD, sensitivitySSD, and F-score values
- Average metrics across conditions per method, assembled as a benchmark table (e.g. table1_benchmark_metrics.csv, table2_demultiplexing_benchmark.csv) reproducing Table 1/Table 2 of the demuxmix article

## How to apply

Start from an (HTOs x droplets) count matrix plus per-droplet gene-detection counts. Establish ground truth per dataset type: for human brain snRNA-seq, use demuxlet-derived high-confidence singlet/doublet (SSD) calls; for cell line mixtures, derive sample-of-origin labels from clustering of RNA profiles. Simulate degraded labeling by scaling HTO counts with attenuation factors s from 1 down to 0.1 (brain benchmark) or 0.25 (cell line benchmark), and at each level artificially generate doublets by merging pairs of droplets — 200 merged doublets for the brain benchmark, 350 for the cell line mixture. Run each demultiplexing method (demuxmix in default and naïve mode, HTODemux, hashedDrops, MULTI-seq, GMM-Demux, DemuxEM) with default settings on every simulated dataset. Compare each method's droplet assignments to the ground truth per condition, computing precisionSSD, sensitivitySSD, and F-score, then average metrics across attenuation levels per method. The rationale is that attenuated HTO signal and added multiplets reproduce the failure modes of weak oligonucleotide labeling, letting methods be ranked on how many true SSDs they recover (demuxmix discarded only 6.3% of true SSDs vs up to 12.8% for other methods; it recovered ~85% of SSDs vs 67%–81% for competitors on the cell line data).

## Related tools

- **demuxmix** (Reference demultiplexing method (negative binomial regression mixture models, default and naïve mode) evaluated on attenuated HTO datasets; also the package providing the analysis framework) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **HTODemux** (Competing demultiplexing method run with default settings on each simulated dataset)
- **hashedDrops** (Competing demultiplexing method run with default settings on each simulated dataset)
- **MULTI-seq** (Competing demultiplexing method run with default settings on each simulated dataset)
- **GMM-Demux** (Competing demultiplexing method run with default settings on each simulated dataset)
- **DemuxEM** (Competing demultiplexing method run with default settings on each simulated dataset)
- **demuxlet** (Source of high-confidence singlet/doublet ground-truth labels for the human brain benchmark)
- **R** (Implementation environment (R/Bioconductor) in which the attenuation, doublet simulation, and metric computation are carried out) — https://doi.org/doi:10.18129/B9.bioc.demuxmix

## Examples

```
BiocManager::install("demuxmix"); library(demuxmix); dmm <- demuxmix(htoMatrix = hto_counts, numGenes = genes_per_drop); p <- classify(dmm); table(p[, "predictions"])
```

## Evaluation signals

- Computed average precisionSSD, sensitivitySSD, and F-score per method match the values reported in Table 1 and Table 2 of the demuxmix article
- demuxmix attains approximately 85% average precisionSSD on the cell line mixture benchmark while competing methods fall in the 67%–81% range
- On the human brain benchmark, demuxmix discards only ~6.3% of true SSDs as uncertain, negative, or MSMs versus up to 12.8% for other methods, with the highest average precisionSSD and sensitivitySSD
- Each simulated dataset at attenuation level s contains the specified number of artificial doublets (200 for the brain benchmark, 350 for the cell line benchmark) with HTO counts scaled by the factor s
- Metrics are averaged across all attenuation conditions per method before ranking, not evaluated at a single s value

## Limitations

- Simulated attenuation of HTO counts only approximates real reduced oligonucleotide labeling; artificial doublets are created by merging pairs of droplets rather than arising from actual multiplet events
- Ground-truth quality is contingent: demuxlet SSD calls serve as ground truth for the brain benchmark and RNA-profile cluster labels for the cell line benchmark, so mislabeled references propagate into the evaluation
- Competing methods are run with default settings, so the comparison reflects default-parameter behavior rather than tuned performance

## Evidence

- [other] Attenuate the HTO count matrix by scaling factors from s = 1 down to s = 0.1, adding 200 merged doublets at each attenuation level to simulate reduced oligonucleotide labeling and multiplet presence.: [evidence span withheld — anchor status: not_found]
- [other] Simulate challenging demultiplexing scenarios by scaling the HTO counts with attenuation factors s from 1 down to 0.25 and by artificially generating 350 doublets by combining pairs of droplets.: [evidence span withheld — anchor status: not_found]
- [other] Compare each method's droplet assignments to the demuxlet SSD ground truth, computing precisionSSD, sensitivitySSD, and F-score per condition, then average them across conditions per method.: [evidence span withheld — anchor status: not_found]
- [abstract] demuxmix's probabilistic classification framework provides error probabilities for droplet assignments that can be used to discard uncertain droplets: "demuxmix's probabilistic classification framework provides error probabilities for droplet assignments that can be used to discard uncertain droplets"
- [abstract] The improved performance of demuxmix compared with existing demultiplexing methods is assessed using real and simulated data.: "The improved performance of demuxmix compared with existing demultiplexing methods is assessed using real and simulated data."
- [readme] Demultiplexing error rates can be estimated, and methods for quality control are provided.: "Demultiplexing error rates can be estimated, and methods for quality control are provided."
