---
name: simulated-doublet-data-generation
description: Use when benchmarking HTO-based demultiplexing methods against a known
  ground truth is needed — e.g. the input is a cell line mixture experiment with HTO
  and RNA count matrices and you must simulate weak HTO signal and known multiplets.
when_to_use_negative:
- The experiment has verified sample-of-origin labels or HTO calls of known quality,
  so no simulated ground truth is needed
- The droplets lack RNA profiles, making RNA-cluster ground-truth labels impossible
  to derive
- The goal is demultiplexing a real pooled sample rather than benchmarking method
  performance
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: demuxmix
  role: Target method evaluated on the simulated datasets; fits negative binomial
    regression mixture models on HTO counts plus number of detected genes for probabilistic
    classification and multiplet detection
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Environment in which the simulation (HTO attenuation and artificial doublet
    construction) and the demuxmix benchmark are implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:95557cdad3fa513d6851a7b5f67541660f5636376b25eec2e8f53b74b0a55b93
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - demuxmix
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# simulated-doublet-data-generation

## Summary

Generates synthetic benchmark scRNA-seq demultiplexing datasets by attenuating hashtag oligonucleotide (HTO) counts (s = 1 down to 0.25) and injecting artificial doublets from pairs of droplets, using RNA-profile clusters as ground truth to compare demultiplexing methods.

## When to use

Use when benchmarking HTO-based demultiplexing methods against a known ground truth is needed — e.g. the input is a cell line mixture experiment with HTO and RNA count matrices and you must simulate weak HTO signal and known multiplets.

## When NOT to use

- The experiment has verified sample-of-origin labels or HTO calls of known quality, so no simulated ground truth is needed
- The droplets lack RNA profiles, making RNA-cluster ground-truth labels impossible to derive
- The goal is demultiplexing a real pooled sample rather than benchmarking method performance

## Inputs

- HTO count matrix (HTOs x droplets) from GEO accession GSM3501446/GSM3501447 (cell line mixture experiment)
- RNA count matrix (genes x droplets) for the same experiment
- RNA-profile cluster labels used as ground-truth sample-of-origin assignments
- Grid of HTO attenuation factors s (1 down to 0.25)
- Number of artificial doublets to inject (350), built by combining pairs of droplets

## Outputs

- Simulated demultiplexing benchmark datasets at each attenuation level with known ground-truth labels
- table2_demultiplexing_benchmark.csv tabulating per-method precisionSSD and sensitivitySSD per attenuation level (Table 2 format)

## How to apply

Download the HTO count matrix (HTOs x droplets) and RNA count matrix (genes x droplets) for the cell line mixture experiment (GSM3501446/GSM3501447) from GEO. Simulate challenging scenarios by scaling HTO counts with attenuation factors s from 1 down to 0.25, mimicking weak hashtag signal, and artificially generate 350 doublets by combining the count vectors of pairs of droplets. Derive ground-truth sample-of-origin labels for every droplet from clustering of the RNA profiles — not from the HTO signal — so the labels are independent of the attenuated data being evaluated. Run demuxmix (negative binomial regression mixture models on HTO counts plus the number of detected genes per droplet, leveraging the positive association between detected genes and HTO counts) and each competing method on every simulated dataset. Compute average precision for singlet/doublet classification (precisionSSD) and sensitivity (sensitivitySSD) per method and attenuation level, and tabulate the metrics in the format of Table 2.

## Related tools

- **demuxmix** (Target method evaluated on the simulated datasets; fits negative binomial regression mixture models on HTO counts plus number of detected genes for probabilistic classification and multiplet detection) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Environment in which the simulation (HTO attenuation and artificial doublet construction) and the demuxmix benchmark are implemented)

## Examples

```
if (!require("BiocManager")) install.packages("BiocManager"); BiocManager::install("demuxmix")
```

## Evaluation signals

- demuxmix attains approximately 85% average precisionSSD on the simulated cell line mixture benchmark
- Competing demultiplexing methods fall in the 67%-81% average precisionSSD range, matching the reported Table 2 comparison
- Ground-truth labels are derived from RNA-profile clustering, independent of the attenuated HTO signal
- The attenuation series spans s = 1 down to 0.25 with metrics computed at each level
- Exactly 350 artificial doublets are generated per simulated dataset by combining pairs of droplets

## Limitations

- Ground truth is only as accurate as the RNA-profile clustering, so misclustering propagates into benchmark labels
- Artificial doublets are constructed by combining pairs of droplets and may not capture all real multiplet biology
- Simulated HTO attenuation is a simple multiplicative scaling; real weak-hashtag experiments may degrade differently

## Evidence

- [abstract] improved performance of demuxmix compared with existing methods: "The improved performance of demuxmix compared with existing demultiplexing methods is assessed using real and simulated data."
- [intro] HTO count matrix structure: "sequenced together with the RNA molecules of the cells resulting in an (HTOs x droplets) count matrix in addition to the"
- [intro] genes-detected covariate used in the regression models: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method"
- [readme] package capabilities: "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection."
