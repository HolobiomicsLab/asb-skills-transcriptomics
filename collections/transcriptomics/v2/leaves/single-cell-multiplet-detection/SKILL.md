---
name: single-cell-multiplet-detection
description: Use when an (HTOs x droplets) hashtag-oligonucleotide count matrix from
  pooled cells/nuclei must be assigned to samples of origin with error probabilities,
  uncertain-droplet discarding, and multiplet (singlet/doublet) detection.
when_to_use_negative:
- Data are not oligonucleotide-barcoded — e.g., natural genetic multiplexing demultiplexed
  from genotypes with demuxlet; demuxmix requires HTO counts
- Droplets have already been assigned to samples and only downstream per-sample analysis
  is needed
- Complete per-cell transcription profiles are available and contemplated as the covariate
  — the method calls for the total number of genes detected per droplet, not the full
  profile
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2229
tools:
- name: demuxmix
  role: 'Core method: negative binomial regression mixture models for probabilistic
    demultiplexing of HTO counts, multiplet detection, and error-rate estimation'
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: HTODemux
  role: Baseline demultiplexing method run with default settings for benchmark comparison
- name: hashedDrops
  role: Baseline demultiplexing method run with default settings for benchmark comparison
- name: MULTI-seq
  role: Baseline demultiplexing method run with default settings for benchmark comparison
- name: GMM-Demux
  role: Baseline demultiplexing method run with default settings for benchmark comparison
- name: DemuxEM
  role: Baseline demultiplexing method run with default settings for benchmark comparison
- name: demuxlet
  role: Source of high-confidence singlet/doublet (SSD) ground-truth labels used to
    evaluate all methods
- name: R
  role: Runtime environment; demuxmix is an R/Bioconductor package installed via BiocManager
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:c376a71600955dfa181dd64672d0861c8c6a52a44a09df7282b348050b23cb1e
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# single-cell-multiplet-detection

## Summary

Demultiplex oligonucleotide-barcoded single-cell RNA-seq data using demuxmix, which fits negative binomial regression mixture models to HTO counts (leveraging per-droplet gene-detection counts) to probabilistically assign droplets to samples, detect multiplets, and estimate error rates.

## When to use

Use when an (HTOs x droplets) hashtag-oligonucleotide count matrix from pooled cells/nuclei must be assigned to samples of origin with error probabilities, uncertain-droplet discarding, and multiplet (singlet/doublet) detection.

## When NOT to use

- Data are not oligonucleotide-barcoded — e.g., natural genetic multiplexing demultiplexed from genotypes with demuxlet; demuxmix requires HTO counts
- Droplets have already been assigned to samples and only downstream per-sample analysis is needed
- Complete per-cell transcription profiles are available and contemplated as the covariate — the method calls for the total number of genes detected per droplet, not the full profile

## Inputs

- (HTOs x droplets) hashtag oligonucleotide count matrix from oligonucleotide-labeled pooled single-cell data
- Per-droplet total number of detected genes (from the RNA library) used as regression covariate
- Optional ground-truth singlet/doublet (SSD) labels, e.g., demuxlet-derived high-confidence calls, for benchmark evaluation

## Outputs

- Probabilistic per-droplet sample assignments (singlet, multiplet, negative) with error probabilities
- Estimated demultiplexing error rates and QC summaries for discarding uncertain droplets
- Benchmark metrics (average precisionSSD, sensitivitySSD, F-score per method across attenuation conditions), e.g., table1_benchmark_metrics.csv

## How to apply

Load the (HTOs x droplets) HTO count matrix produced by sequencing hashtag oligonucleotides together with the RNA library, and pass the total number of genes detected per droplet alongside it — demuxmix exploits the positive association between detected genes and HTO counts to explain variance in the HTO data; omit this covariate only in naive mode. Fit demuxmix's negative binomial regression mixture models (default settings), which yield per-droplet classifications into singlets, multiplets, and negatives with error probabilities. Use the error probabilities to discard uncertain droplets rather than forcing assignments, and estimate demultiplexing error rates with the package's QC methods. If benchmarking, compare assignments against demuxlet-derived high-confidence SSD ground truth under HTO attenuation (scaling factors s = 1 down to s = 0.1 with 200 merged doublets added), computing precisionSSD, sensitivitySSD, and F-score per condition and averaging across conditions, against baselines HTODemux, hashedDrops, MULTI-seq, GMM-Demux, and DemuxEM. On the simulated human brain snRNA-seq benchmark, demuxmix achieved the highest average precisionSSD and sensitivitySSD while discarding only 6.3% of true SSDs as uncertain, negative, or MSMs.

## Related tools

- **demuxmix** (Core method: negative binomial regression mixture models for probabilistic demultiplexing of HTO counts, multiplet detection, and error-rate estimation) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **HTODemux** (Baseline demultiplexing method run with default settings for benchmark comparison)
- **hashedDrops** (Baseline demultiplexing method run with default settings for benchmark comparison)
- **MULTI-seq** (Baseline demultiplexing method run with default settings for benchmark comparison)
- **GMM-Demux** (Baseline demultiplexing method run with default settings for benchmark comparison)
- **DemuxEM** (Baseline demultiplexing method run with default settings for benchmark comparison)
- **demuxlet** (Source of high-confidence singlet/doublet (SSD) ground-truth labels used to evaluate all methods)
- **R** (Runtime environment; demuxmix is an R/Bioconductor package installed via BiocManager) — https://doi.org/doi:10.18129/B9.bioc.demuxmix

## Examples

```
if (!require("BiocManager")) install.packages("BiocManager"); BiocManager::install("demuxmix"); library(demuxmix)
```

## Evaluation signals

- Average precisionSSD, sensitivitySSD, and F-score per method computed against demuxlet SSD ground truth must match the Table 1 values from the article
- On simulated human brain data, demuxmix achieves the highest average precisionSSD and sensitivitySSD among compared methods
- demuxmix discards only 6.3% of true SSDs as uncertain, negative, or MSMs versus up to 12.8% for other methods
- Every droplet receives a classification with an error probability, and uncertain droplets can be discarded via quality-control methods
- Benchmark spans HTO attenuation scaling factors from s = 1 down to s = 0.1 with 200 merged doublets added at each level

## Limitations

- Performance depends on the HTO signal: benchmark conditions attenuated HTO counts (s = 1 to 0.1), and a fraction of true SSDs (6.3% for demuxmix) are left unassigned as uncertain, negative, or MSMs
- Naive mode omits the gene-count covariate, forgoing the variance-explaining association between detected genes and HTO counts
- Regression mode requires per-droplet gene-detection counts; the full transcription profile should not be passed, only the number of genes detected per droplet

## Evidence

- [abstract] The method, called demuxmix, implements two significant improvements.: "A new demultiplexing method based on negative binomial regression mixture models is introduced. The method, called demuxmix, implements two significant improvements."
- [abstract] demuxmix utilizes the positive association between detected genes in the RNA library and HTO counts to explain variance, improving droplet assignments.: "demuxmix utilizes the positive association between detected genes in the RNA library and HTO counts to explain parts of the variance in the HTO data resulting in improved droplet assignments"
- [intro] The demuxmix package implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models.: "The *demuxmix* package implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models."
- [readme] The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection.: "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection. Demultiplexing error rates can be estimated, and methods"
