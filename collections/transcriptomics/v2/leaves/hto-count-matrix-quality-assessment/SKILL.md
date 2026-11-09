---
name: hto-count-matrix-quality-assessment
description: Use when an (HTOs x droplets) count matrix from oligonucleotide-labeled
  pooled scRNA-seq needs to be quality-checked before demultiplexing, or when comparing
  real HTO datasets against simulated low-quality levels to anticipate demultiplexing
  ambiguity.
when_to_use_negative:
- The input is an RNA (genes x droplets) expression matrix without HTO counts — demuxmix
  classification is based on HTO counts, not the transcription profile
- The HTO data already has high-quality, well-separated signals and only a point estimate
  of sample assignment is needed — the ovs survey is about anticipating method sensitivity,
  not required routine QC
- The experiment does not use hashtag oligonucleotide (HTO) labeling (e.g., genotype-based
  or cell hashing-free demultiplexing)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: demuxmix
  role: Fits negative binomial regression mixture models to HTO counts and computes
    the overlap score (ovs) used to assess dataset quality
  repo: https://github.com/huklein/demuxmix
- name: R
  role: Runtime environment for loading HTO count matrices and running demuxmix via
    the R/Bioconductor package
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:68861698f2ea9c51896304ce8d4996e98f9111cccdc318b689e0da885ab8aaaa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huklein/demuxmix
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# hto-count-matrix-quality-assessment

## Summary

Quantify the quality of an HTO count matrix (HTOs x droplets) by fitting demuxmix's negative binomial regression mixture models and computing the overlap score (ovs), which measures the overlap between fitted mixture components; low ovs values indicate HTO data of poor quality where demultiplexing method choice will strongly affect results.

## When to use

Use when an (HTOs x droplets) count matrix from oligonucleotide-labeled pooled scRNA-seq needs to be quality-checked before demultiplexing, or when comparing real HTO datasets against simulated low-quality levels to anticipate demultiplexing ambiguity.

## When NOT to use

- The input is an RNA (genes x droplets) expression matrix without HTO counts — demuxmix classification is based on HTO counts, not the transcription profile
- The HTO data already has high-quality, well-separated signals and only a point estimate of sample assignment is needed — the ovs survey is about anticipating method sensitivity, not required routine QC
- The experiment does not use hashtag oligonucleotide (HTO) labeling (e.g., genotype-based or cell hashing-free demultiplexing)

## Inputs

- HTO count matrix (HTOs x droplets) from an oligonucleotide-labeled pooled single-cell RNA-seq experiment (e.g., from GEO)
- Optional: number of genes detected per droplet (RNA library), passed alongside HTO counts to improve the fit
- Benchmark HTO count matrices and/or simulated low-quality HTO data (with HTO-signal scaling factor s) as comparison reference

## Outputs

- Per-dataset overlap score (ovs) values quantifying fitted-component overlap
- Category-averaged ovs summary table (e.g., overlap_scores.tsv)
- Panel figure comparing ovs per dataset and category averages (e.g., s5ab_panels.png)

## How to apply

Load the HTO count matrix (and optionally the number of genes detected per droplet, since demuxmix leverages the positive association between detected genes and HTO counts) into R and fit demuxmix's negative binomial regression mixture models to the HTO counts. Compute the overlap score (ovs) per dataset, which quantifies the overlap between the fitted mixture components — smaller ovs means better-separated HTO signals. To contextualize a real dataset, average the ovs values by dataset category (e.g., cell line vs human brain) and compare against benchmark datasets and simulated low-quality datasets generated with a small HTO-signal scaling factor s. If a real dataset's ovs falls at or below the simulated low-quality levels, expect that the choice of demultiplexing method will materially affect the results. In the source survey, category-average ovs was 0.019 for cell line and 0.002 for human brain datasets, with some published GEO datasets falling below even the simulated low-quality levels.

## Related tools

- **demuxmix** (Fits negative binomial regression mixture models to HTO counts and computes the overlap score (ovs) used to assess dataset quality) — https://github.com/huklein/demuxmix
- **R** (Runtime environment for loading HTO count matrices and running demuxmix via the R/Bioconductor package) — https://doi.org/doi:10.18129/B9.bioc.demuxmix

## Examples

```
Rscript -e 'library(demuxmix); dmm <- demuxmix(hto, genes=ngenes); ovs <- ovs(dmm)'
```

## Evaluation signals

- Reproduced category-average ovs matches the reported values (0.019 for cell line, 0.002 for human brain datasets)
- Some real published GEO HTO datasets fall below the ovs levels of simulated low-quality datasets (small HTO-signal scaling factor s)
- ovs values are computed per dataset after fitting demuxmix negative binomial regression mixture models, and are averaged by dataset category
- Summary table written with expected columns/datasets (overlap_scores.tsv) and figure panels reproduce the Supplementary Fig. S5A/B layout
- Fits converge on all 12 datasets (10 GEO + 2 benchmark) without errors before ovs aggregation

## Limitations

- Low ovs on real data implies the choice of demultiplexing method likely affects results — the score does not itself rescue poor-quality HTO data
- Survey depends on availability and quality of published GEO datasets; findings are specific to the 10 recently published HTO datasets plus the two benchmarks
- The overlap score is a model-based summary of fitted mixture components; results depend on the negative binomial regression fit and the optional gene-count covariate

## Evidence

- [methods] Fit demuxmix's negative binomial regression mixture models to the HTO counts and compute the overlap score (ovs) quantifying the overlap between fitted components: "The *demuxmix* package implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models."
- [results] Real GEO datasets can fall below simulated low-quality levels: [evidence span withheld — anchor status: not_found]
- [intro] Gene counts passed alongside HTO counts to improve fit: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts"
- [intro] HTO data is an (HTOs x droplets) count matrix: "resulting in an (HTOs x droplets) count matrix in addition to the"
- [readme] Install via Bioconductor: "BiocManager::install("demuxmix")"
