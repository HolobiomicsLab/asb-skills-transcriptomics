---
name: hto-demultiplexing-benchmark-evaluation
description: Use when an (HTOs x droplets) HTO count matrix from a pooled, hashtag-oligonucleotide-labeled
  scRNA-seq/sRNA-seq experiment needs demultiplexing with quantified error rates,
  or when comparing demultiplexing methods against a ground-truth labeling.
when_to_use_negative:
- Input is genetic-demultiplexing output (demuxlet/freemuxlet calls) — in this skill's
  workflows those labels serve as ground truth, not as the method being run
- The dataset lacks an HTO count matrix (no barcode oligonucleotide labeling), e.g.
  untagged single-sample libraries — demultiplexing by HTO cannot be performed
- Only total gene expression profiles are available without the number of genes detected
  per droplet, so the regression covariate central to demuxmix's improvement cannot
  be supplied
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3307
tools:
- name: demuxmix
  role: 'Primary demultiplexing method: negative binomial regression mixture models
    over HTO counts with gene-count covariate, probabilistic classification and multiplet
    detection'
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: HTODemux
  role: Competing demultiplexing method run with default settings in the benchmark
    comparison
- name: hashedDrops
  role: Competing demultiplexing method run with default settings in the benchmark
    comparison
- name: MULTI-seq
  role: Competing demultiplexing method run with default settings in the benchmark
    comparison
- name: GMM-Demux
  role: Competing demultiplexing method run with default settings in the benchmark
    comparison
- name: DemuxEM
  role: Competing demultiplexing method run with default settings in the benchmark
    comparison
- name: demuxlet
  role: Genetic demultiplexer whose high-confidence SSD calls provide ground-truth
    labels for benchmark evaluation
- name: R
  role: Implementation environment; demuxmix is an R/Bioconductor package installed
    via BiocManager
  repo: https://bioconductor.org
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:175e03d96fe3b70b9539d99c84b41b65cc583438d3b9bacfde4dca5047ab4e9e
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# hto-demultiplexing-benchmark-evaluation

## Summary

Benchmark and evaluate HTO-based demultiplexing methods (demuxmix, HTODemux, hashedDrops, MULTI-seq, GMM-Demux, DemuxEM) on attenuated HTO count matrices, scoring singlet/doublet classification against genetic (demuxlet/freemuxlet) or RNA-cluster ground truth.

## When to use

Use when an (HTOs x droplets) HTO count matrix from a pooled, hashtag-oligonucleotide-labeled scRNA-seq/sRNA-seq experiment needs demultiplexing with quantified error rates, or when comparing demultiplexing methods against a ground-truth labeling.

## When NOT to use

- Input is genetic-demultiplexing output (demuxlet/freemuxlet calls) — in this skill's workflows those labels serve as ground truth, not as the method being run
- The dataset lacks an HTO count matrix (no barcode oligonucleotide labeling), e.g. untagged single-sample libraries — demultiplexing by HTO cannot be performed
- Only total gene expression profiles are available without the number of genes detected per droplet, so the regression covariate central to demuxmix's improvement cannot be supplied

## Inputs

- HTO count matrix (HTOs x droplets) from hashtag-oligonucleotide-labeled pooled cells
- Per-droplet number of detected genes (RNA covariate) for the regression mixture model
- Ground-truth labels: demuxlet-derived high-confidence singlet/doublet (SSD) calls or RNA-profile cluster labels
- Attenuation scaling factors (s = 1 down to 0.1 or 0.25) and number of simulated doublets (200 or 350)
- Acceptance-probability threshold pAcpt (e.g. 0.99) for discarding uncertain droplets

## Outputs

- Per-method average precisionSSD, sensitivitySSD, and F-score table (e.g. table1_benchmark_metrics.csv, table2_demultiplexing_benchmark.csv)
- Per-method specificity, sensitivity, and uncertain-removal-rate for one-HTO designs (e.g. demuxmix_csf_evaluation.csv)
- Per-droplet demultiplexing assignments with error probabilities for quality control

## How to apply

Load the (HTOs x droplets) count matrix plus per-droplet gene-detection counts, and simulate challenging scenarios by attenuating HTO counts with scaling factors s from 1 down to 0.1 (human brain snRNA-seq) or 0.25 (cell line mixture, GSM3501446/GSM3501447), adding 200 or 350 merged doublets by combining droplet pairs. Run demuxmix's negative binomial regression mixture model, passing the number of detected genes per droplet as a covariate to leverage the positive gene-count/HTO association, alongside competing methods (HTODemux, hashedDrops, MULTI-seq, GMM-Demux, DemuxEM) with default settings. Classify droplets probabilistically and discard those whose assignment error probability exceeds the acceptance threshold (e.g. pAcpt=0.99) as uncertain/multiplets. Compare assignments to ground truth — demuxlet-derived high-confidence SSDs or RNA-profile cluster labels — computing precisionSSD, sensitivitySSD, and F-score per attenuation level and averaging across conditions; for one-HTO designs (CSF/PBMC), compute specificity, sensitivity, and the uncertain rate against freemuxlet labels.

## Related tools

- **demuxmix** (Primary demultiplexing method: negative binomial regression mixture models over HTO counts with gene-count covariate, probabilistic classification and multiplet detection) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **HTODemux** (Competing demultiplexing method run with default settings in the benchmark comparison)
- **hashedDrops** (Competing demultiplexing method run with default settings in the benchmark comparison)
- **MULTI-seq** (Competing demultiplexing method run with default settings in the benchmark comparison)
- **GMM-Demux** (Competing demultiplexing method run with default settings in the benchmark comparison)
- **DemuxEM** (Competing demultiplexing method run with default settings in the benchmark comparison)
- **demuxlet** (Genetic demultiplexer whose high-confidence SSD calls provide ground-truth labels for benchmark evaluation)
- **R** (Implementation environment; demuxmix is an R/Bioconductor package installed via BiocManager) — https://bioconductor.org

## Examples

```
if (!require("BiocManager")) install.packages("BiocManager"); BiocManager::install("demuxmix"); library(demuxmix)
```

## Evaluation signals

- On simulated human brain benchmark data, demuxmix must show the highest average precisionSSD and sensitivitySSD, discarding only ~6.3% of true SSDs versus up to 12.8% for other methods (Table 1 reproduction)
- On simulated cell line mixture data, demuxmix should recover ~85% of SSDs while competing methods fall in the 67%-81% range (Table 2)
- On the CSF/PBMC one-HTO dataset at pAcpt=0.99, demuxmix must yield specificity 0.985, sensitivity 0.980, and 11.1% (261 cells) removed as uncertain; the naive variant 0.983/0.972/13.1% (308 cells)
- Per-droplet assignments must contain error probabilities, and droplets exceeding the 0.99 acceptance threshold must be flagged uncertain/multiplet rather than assigned
- Computed average metrics must match the values reported in the article's Table 1/Table 2 when written to the output CSVs

## Limitations

- Performance degrades as HTO signal is attenuated (s = 1 down to 0.1 or 0.25 with simulated doublets), so metrics are condition-dependent averages rather than absolute guarantees
- Omitting the gene-count covariate (naive mode) measurably worsens results (13.1% vs 11.1% removed as uncertain on the CSF data), so the RNA covariate should be passed whenever available
- A substantial fraction of droplets (e.g. ~11%) may be discarded as uncertain at pAcpt=0.99, trading yield for classification confidence

## Evidence

- [intro] The *demuxmix* package implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models.: "The *demuxmix* package implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models."
- [abstract] The improved performance of demuxmix compared with existing demultiplexing methods is assessed using real and simulated data.: "The improved performance of demuxmix compared with existing demultiplexing methods is assessed using real and simulated data."
- [intro] better results are often achieved if the total number of genes detected per droplet is passed to the method along with the HTO counts: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts"
- [readme] Demultiplexing error rates can be estimated, and methods for quality control are provided.: "Demultiplexing error rates can be estimated, and methods for quality control are provided."
