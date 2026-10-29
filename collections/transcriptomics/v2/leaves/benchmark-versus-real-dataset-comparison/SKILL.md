---
name: benchmark-versus-real-dataset-comparison
description: Use when you have HTO count matrices from multiple published or in-house
  datasets plus benchmark/simulated references, and need to know if real data quality
  (component overlap) is comparable to benchmarks or degraded enough that demultiplexing
  results are method-sensitive.
when_to_use_negative:
- The goal is to actually demultiplex droplets into sample assignments rather than
  to quantify HTO data quality via component overlap.
- The input lacks HTO count matrices (only the RNA gene-count matrix is available),
  so no mixture models over HTO counts can be fit.
- No benchmark or simulated reference datasets are available, so ovs values cannot
  be placed on a comparative quality scale.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: demuxmix
  role: R/Bioconductor package used to fit negative binomial regression mixture models
    to HTO counts and compute the overlap score (ovs) quantifying overlap between
    fitted components
  repo: https://github.com/huklein/demuxmix
- name: R
  role: Runtime environment for loading HTO count matrices, fitting demuxmix models,
    averaging ovs by category, and writing the summary table and figure
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:e36ce415363faa8f2dadfd2f13c966bf089fbb0707df4b1c8ed45a0f04724781
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huklein/demuxmix
  tools:
  - R
  - demuxmix
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# benchmark-versus-real-dataset-comparison

## Summary

Quantifies HTO data quality of real published datasets by fitting demuxmix negative binomial regression mixture models and computing the overlap score (ovs), then comparing those ovs values against benchmark datasets and simulated low-quality levels to judge whether demultiplexing method choice will matter.

## When to use

Use when you have HTO count matrices from multiple published or in-house datasets plus benchmark/simulated references, and need to know if real data quality (component overlap) is comparable to benchmarks or degraded enough that demultiplexing results are method-sensitive.

## When NOT to use

- The goal is to actually demultiplex droplets into sample assignments rather than to quantify HTO data quality via component overlap.
- The input lacks HTO count matrices (only the RNA gene-count matrix is available), so no mixture models over HTO counts can be fit.
- No benchmark or simulated reference datasets are available, so ovs values cannot be placed on a comparative quality scale.

## Inputs

- HTO count matrices (HTOs x droplets) from published GEO datasets (e.g., 10 recent datasets)
- HTO count matrices from the two benchmark datasets (cell line and human brain)
- Simulated low-quality HTO datasets generated with a small HTO-signal scaling factor s
- Optionally, per-droplet number of detected genes from the RNA count matrix (genes x droplets)

## Outputs

- Per-dataset overlap score (ovs) values from fitted demuxmix mixture models
- Category-averaged ovs values (e.g., cell line vs human brain)
- Summary table (overlap_scores.tsv)
- Panel figure of ovs per dataset and category averages (s5ab_panels.png)

## How to apply

Download the target HTO datasets (e.g., 10 recently published GEO datasets) and load the HTO count matrices of the two benchmark datasets into R with the demuxmix package (Bioconductor). For each dataset, fit demuxmix's negative binomial regression mixture models to the HTO counts — optionally passing the number of detected genes per droplet alongside HTO counts to leverage their positive association — and compute the overlap score (ovs), which quantifies the overlap between the fitted mixture components: lower ovs means better-separated HTO signals. Average the per-dataset ovs values by dataset category (e.g., cell line vs human brain) and place the real datasets on the same scale as simulated low-quality levels (generated with small HTO-signal scaling factor s). Interpretation: if real datasets fall at or below simulated low-quality levels, the choice of demultiplexing method likely affects results on that data; if ovs matches benchmarks, method choice is less critical. Summarize per-dataset ovs and category averages in a table and reproduce the panel figure showing ovs per dataset and category averages.

## Related tools

- **demuxmix** (R/Bioconductor package used to fit negative binomial regression mixture models to HTO counts and compute the overlap score (ovs) quantifying overlap between fitted components) — https://github.com/huklein/demuxmix
- **R** (Runtime environment for loading HTO count matrices, fitting demuxmix models, averaging ovs by category, and writing the summary table and figure)

## Examples

```
if (!require("BiocManager")) install.packages("BiocManager"); BiocManager::install("demuxmix"); library(demuxmix)
```

## Evaluation signals

- Reproduced category-average ovs values match the reported values: 0.019 for the cell line category and 0.002 for the human brain category.
- Some real published datasets fall below the ovs levels of the simulated low-quality datasets (small HTO-signal scaling factor s), as reported.
- The summary table overlap_scores.tsv contains per-dataset ovs values for all 12 datasets (10 real GEO + 2 benchmark).
- The figure s5ab_panels.png reproduces the Supplementary Fig. S5A/B panel layout showing ovs per dataset and category averages.
- Every ovs value derives from a converged demuxmix negative binomial regression mixture model fit to that dataset's HTO counts.

## Limitations

- Real published HTO datasets vary widely in quality: some match the benchmark datasets while others exhibit significantly lower ovs, similar to simulated low-quality data, so conclusions are dataset-specific.
- On low-quality datasets (ovs comparable to simulated data with small scaling factor s), the choice of demultiplexing method likely affects results, so benchmark-derived recommendations may not transfer.
- The overlap score reflects overlap between fitted mixture components, so it depends on the mixture-model fit, not just raw data properties.

## Evidence

- [intro] demuxmix package fits regression mixture models to HTO counts: "The *demuxmix* package implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models."
- [abstract] performance assessed on real and simulated data: "The improved performance of demuxmix compared with existing demultiplexing methods is assessed using real and simulated data."
- [intro] genes detected per droplet improve HTO modeling: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts to leverage the positive"
- [readme] README describes the package scope: "demuxmix is a package for demultiplexing single-cell sequencing experiments of pooled cells labeled with barcode oligonucleotides."
- [abstract] uncertain droplets can be discarded via error probabilities: "provides error probabilities for droplet assignments that can be used to discard uncertain droplets"
