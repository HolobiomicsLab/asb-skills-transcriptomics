---
name: overlap-score-computation-hto-datasets
description: Use when you need a numeric, per-dataset quality metric for HTO demultiplexing
  data — e.g., to compare newly published GEO HTO datasets against benchmarks or simulated
  low-quality levels before choosing a demultiplexing method.
when_to_use_negative:
- The input is RNA gene-by-droplet count data without an accompanying HTO count matrix
  — the overlap score is defined on HTO counts only
- You need cell-to-sample assignments (demultiplexing calls) rather than a dataset-level
  quality metric — use demuxmix's classification instead of computing ovs
- The HTO data has already been demultiplexed and filtered, since ovs is computed
  from fitted mixture components on the raw HTO counts
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3542
- http://edamontology.org/topic_6390
tools:
- name: demuxmix
  role: Fits negative binomial regression mixture models to HTO counts and computes
    the overlap score (ovs) quantifying overlap between fitted components
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Runtime environment for loading HTO count matrices, fitting models, and averaging
    ovs values by dataset category
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:2196f9436cac3ccbfb62c62752de5b8ae997ae6052fe319b083d400fa70260f0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3542
  - http://edamontology.org/topic_6390
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# overlap-score-computation-hto-datasets

## Summary

Quantify the quality of hashtag oligonucleotide (HTO) count matrices by fitting demuxmix's negative binomial regression mixture models and computing the overlap score (ovs) between fitted components, enabling comparison of real datasets against benchmark or simulated quality levels.

## When to use

Use when you need a numeric, per-dataset quality metric for HTO demultiplexing data — e.g., to compare newly published GEO HTO datasets against benchmarks or simulated low-quality levels before choosing a demultiplexing method.

## When NOT to use

- The input is RNA gene-by-droplet count data without an accompanying HTO count matrix — the overlap score is defined on HTO counts only
- You need cell-to-sample assignments (demultiplexing calls) rather than a dataset-level quality metric — use demuxmix's classification instead of computing ovs
- The HTO data has already been demultiplexed and filtered, since ovs is computed from fitted mixture components on the raw HTO counts

## Inputs

- HTO count matrices (HTOs x droplets) from two benchmark datasets
- Downloaded HTO count matrices from 10 recently published GEO HTO datasets
- Simulated HTO count matrices generated at low HTO-signal scaling factor s levels

## Outputs

- Per-dataset overlap score (ovs) values
- Category-averaged ovs summary table (overlap_scores.tsv)
- Panel figure of ovs per dataset and category averages (s5ab_panels.png, reproducing Supplementary Fig. S5A/B)

## How to apply

Load each (HTOs x droplets) count matrix into R and, using the demuxmix package, fit negative binomial regression mixture models to the HTO counts, optionally passing the total number of detected genes per droplet alongside the HTO counts to leverage their positive association. From each fitted mixture, compute the overlap score (ovs), which quantifies the overlap between the fitted components (e.g., the positive/negative HTO signal components); higher overlap indicates lower dataset quality. Average the ovs values within dataset categories (e.g., cell line vs. human brain) and compare them against benchmark datasets and simulated datasets generated with a reduced HTO-signal scaling factor s. Write per-dataset ovs values to a summary table (e.g., overlap_scores.tsv) and reproduce panel figures (e.g., Supplementary Fig. S5A/B) to visualize ovs per dataset and category averages. Correct application is judged by whether category averages match reported reference values (0.019 cell line, 0.002 human brain) and whether some real datasets fall below simulated low-quality levels.

## Related tools

- **demuxmix** (Fits negative binomial regression mixture models to HTO counts and computes the overlap score (ovs) quantifying overlap between fitted components) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Runtime environment for loading HTO count matrices, fitting models, and averaging ovs values by dataset category)

## Examples

```
Rscript -e 'library(demuxmix); dmm <- demuxmix(hto = htoCounts, rna = numGenes); ovs <- ovsPeaks(dmm)'
```

## Evaluation signals

- Reproduced category-average ovs values match the reported references: 0.019 for the cell line category and 0.002 for the human brain category
- Some real published datasets exhibit ovs values below the levels of the simulated low-quality datasets (small HTO-signal scaling factor s), consistent with the reported finding
- ovs is computed for all 12 datasets (10 GEO + 2 benchmark) with no missing per-dataset values in overlap_scores.tsv
- Fitted mixture models converge and produce defined components per HTO so the component overlap is computable

## Limitations

- Published datasets vary widely in quality: some match the benchmarks while others are as poor as simulated low-quality data, so ovs-derived conclusions about method choice may not generalize uniformly
- The overlap score is a dataset-level quality summary; it does not by itself provide per-droplet classification or error probabilities
- Computing ovs requires fitting demuxmix's negative binomial regression mixture models, which depends on the HTO count distribution and benefits from gene-count covariate data being available

## Evidence

- [other] fit demuxmix's negative binomial regression mixture models to the HTO counts and compute the overlap score (ovs) quantifying the overlap between fitted components: [evidence span withheld — anchor status: not_found]
- [abstract] demuxmix utilizes the positive association between detected genes in the RNA library and HTO counts to explain parts of the variance in the HTO data resulting in improved droplet assignments: "demuxmix utilizes the positive association between detected genes in the RNA library and HTO counts to explain parts of the variance in the HTO data resulting in improved droplet assignments"
- [readme] The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection. Demultiplexing error rates can be estimated, and methods: "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection. Demultiplexing error rates can be estimated, and methods"
- [intro] better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts"
