---
name: demuxlet-ground-truth-matching
description: Use when evaluating or comparing demultiplexing methods (e.g., demuxmix
  vs HTODemux, hashedDrops, MULTI-seq, GMM-Demux, DemuxEM) and genotype-derived ground-truth
  labels from demuxlet are available for the same (HTOs x droplets) count matrix.
when_to_use_negative:
- No genotype data are available to produce demuxlet-based high-confidence SSD ground
  truth for the same droplets
- The experiment is not oligonucleotide-barcoded (no HTO count matrix), so there is
  nothing to demultiplex against SSD labels
- You need de novo demultiplexing of a new sample rather than method benchmarking/evaluation
  against known labels
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: demuxlet
  role: Genotype (SNP)-based demultiplexer providing the high-confidence SSD calls
    used as ground truth for scoring
- name: demuxmix
  role: Method under test; fits negative binomial regression mixture models on HTO
    counts with the gene-count covariate (naïve mode omits it) and discards uncertain
    droplets via error probabilities
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: HTODemux
  role: Comparison demultiplexing method run with default settings on the attenuated
    datasets
- name: hashedDrops
  role: Comparison demultiplexing method run with default settings on the attenuated
    datasets
- name: MULTI-seq
  role: Comparison demultiplexing method run with default settings on the attenuated
    datasets
- name: GMM-Demux
  role: Comparison demultiplexing method run with default settings on the attenuated
    datasets
- name: DemuxEM
  role: Comparison demultiplexing method run with default settings on the attenuated
    datasets
- name: R
  role: Implementation environment (R/Bioconductor) for running demuxmix and computing
    the SSD metrics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:be360ac0cc23d6d52a3b83dfc5a6ecfdf70a4d0c6b210adc129c0f082bc251a5
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# demuxlet-ground-truth-matching

## Summary

Benchmark HTO-based demultiplexing methods by treating demuxlet-derived high-confidence singlet/doublet (SSD) genotype calls as ground truth, then scoring each method's droplet assignments with precisionSSD, sensitivitySSD, and F-score on attenuated human brain snRNA-seq HTO data.

## When to use

Use when evaluating or comparing demultiplexing methods (e.g., demuxmix vs HTODemux, hashedDrops, MULTI-seq, GMM-Demux, DemuxEM) and genotype-derived ground-truth labels from demuxlet are available for the same (HTOs x droplets) count matrix.

## When NOT to use

- No genotype data are available to produce demuxlet-based high-confidence SSD ground truth for the same droplets
- The experiment is not oligonucleotide-barcoded (no HTO count matrix), so there is nothing to demultiplex against SSD labels
- You need de novo demultiplexing of a new sample rather than method benchmarking/evaluation against known labels

## Inputs

- HTO count matrix (HTOs x droplets) from human brain snRNA-seq
- Per-droplet gene-detection counts (total number of genes detected per droplet)
- demuxlet-derived high-confidence singlet/doublet (SSD) calls as ground-truth labels

## Outputs

- Per-method per-condition precisionSSD, sensitivitySSD, and F-score values
- Condition-averaged benchmark metrics table (table1_benchmark_metrics.csv)

## How to apply

Load the (HTOs x droplets) HTO count matrix, the accompanying per-droplet gene-detection counts, and demuxlet-derived high-confidence SSD labels as ground truth. To simulate reduced oligonucleotide labeling, attenuate the HTO counts with scaling factors s from 1 down to 0.1, adding 200 merged doublets at each attenuation level. Run demuxmix (negative binomial regression mixture models) in both default and naïve mode — naïve mode omits the gene-count covariate — plus HTODemux, hashedDrops, MULTI-seq, GMM-Demux, and DemuxEM, all with default settings, on every attenuated dataset. Match each method's per-droplet assignments against the demuxlet SSD labels, computing precisionSSD, sensitivitySSD, and F-score per condition, then average the metrics across conditions per method. The rationale is that demuxlet calls from SNPs are independent of HTO signal, so agreement with them measures how well each HTO method retains true singlets and doublets; demuxmix's use of the positive association between detected genes and HTO counts is expected to yield the highest average precisionSSD and sensitivitySSD while discarding only ~6.3% of true SSDs versus up to 12.8% for other methods.

## Related tools

- **demuxlet** (Genotype (SNP)-based demultiplexer providing the high-confidence SSD calls used as ground truth for scoring)
- **demuxmix** (Method under test; fits negative binomial regression mixture models on HTO counts with the gene-count covariate (naïve mode omits it) and discards uncertain droplets via error probabilities) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **HTODemux** (Comparison demultiplexing method run with default settings on the attenuated datasets)
- **hashedDrops** (Comparison demultiplexing method run with default settings on the attenuated datasets)
- **MULTI-seq** (Comparison demultiplexing method run with default settings on the attenuated datasets)
- **GMM-Demux** (Comparison demultiplexing method run with default settings on the attenuated datasets)
- **DemuxEM** (Comparison demultiplexing method run with default settings on the attenuated datasets)
- **R** (Implementation environment (R/Bioconductor) for running demuxmix and computing the SSD metrics)

## Examples

```
if (!require("BiocManager")) install.packages("BiocManager"); BiocManager::install("demuxmix")
```

## Evaluation signals

- Computed average precisionSSD, sensitivitySSD, and F-score per method match the values reported in Table 1 of the article
- demuxmix attains the highest average precisionSSD and sensitivitySSD among all compared methods
- demuxmix discards only ~6.3% of true SSDs as uncertain, negative, or MSMs, versus up to 12.8% for other methods
- All precision/sensitivity/F-score values fall in [0, 1] and are computed against exactly the demuxlet high-confidence SSD label set
- Each attenuation condition (s = 1 down to s = 0.1, with 200 merged doublets added) is evaluated for every method before averaging

## Limitations

- Ground truth is limited to demuxlet's high-confidence SSDs; uncertain droplets are excluded, so they cannot contribute to sensitivity estimates
- demuxmix's naïve mode omits the gene-count covariate and therefore does not leverage the positive association between detected genes and HTO counts, giving worse assignments
- Performance degrades as HTO signal is attenuated (scaling factors down to s = 0.1), reflecting reduced oligonucleotide labeling; results are specific to attenuated benchmark conditions rather than fully unattenuated real data alone

## Evidence

- [abstract] The improved performance of demuxmix compared with existing demultiplexing methods is assessed using real and simulated data.: "The improved performance of demuxmix compared with existing demultiplexing methods is assessed using real and simulated data."
- [abstract] demuxmix's probabilistic classification framework provides error probabilities for droplet assignments that can be used to discard uncertain droplets: "demuxmix's probabilistic classification framework provides error probabilities for droplet assignments that can be used to discard uncertain droplets"
- [intro] These oligonucleotides are termed hashtag oligonucleotides (HTOs) and are sequenced together with the RNA molecules of the cells resulting in an (HTOs x droplets) count matrix: "These oligonucleotides are termed hashtag oligonucleotides (HTOs) and are sequenced together with the RNA molecules of the cells resulting in an (HTOs x droplets) count matrix"
- [intro] better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts"
- [readme] The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection.: "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection."
