---
name: geo-dataset-retrieval
description: Use when an analysis requires real published HTO datasets — e.g., surveying
  typical dataset quality with an overlap score (ovs) or correlating HTO counts with
  detected genes across datasets — and GEO accessions or 'recently published' HTO
  data must be downloaded.
when_to_use_negative:
- The dataset is already in hand as a local HTO count matrix — skip retrieval and
  proceed directly to model fitting
- The analysis targets simulated HTO data rather than real published experiments;
  simulation replaces GEO retrieval in the demuxmix comparison workflow
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Environment for downloading GEO datasets and loading HTO/RNA count matrices
    for downstream demuxmix analysis
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: demuxmix
  role: Fits negative binomial regression mixture models to the retrieved HTO count
    matrices and computes overlap scores for quality assessment
  repo: https://github.com/huklein/demuxmix
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:430653fe781155ec172db80d7796510303504e65c0dedac02156b5f54bbee748
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# geo-dataset-retrieval

## Summary

Retrieve published HTO-labeled scRNA-seq datasets from GEO (e.g., via accessions like GSM3501446/GSM3501447) to assemble real-data HTO count matrices for quality benchmarking against the demuxmix benchmark and simulated low-quality datasets.

## When to use

Use when an analysis requires real published HTO datasets — e.g., surveying typical dataset quality with an overlap score (ovs) or correlating HTO counts with detected genes across datasets — and GEO accessions or 'recently published' HTO data must be downloaded.

## When NOT to use

- The dataset is already in hand as a local HTO count matrix — skip retrieval and proceed directly to model fitting
- The analysis targets simulated HTO data rather than real published experiments; simulation replaces GEO retrieval in the demuxmix comparison workflow

## Inputs

- GEO accession identifiers (e.g., GSM3501446, GSM3501447) for published HTO-labeled scRNA-seq studies
- Downloaded (HTOs x droplets) HTO count matrices
- Optional (genes x droplets) RNA read count matrices for computing genes detected per droplet

## Outputs

- HTO count matrices per dataset loaded into R
- Per-dataset summary tables such as overlap_scores.tsv and hto_gene_correlations.csv (dataset, sample, HTO type, metric, reference-range match)
- Comparison of real-dataset metrics against benchmark and simulated low-quality reference levels (e.g., Supplementary Fig. S5A/B panels)

## How to apply

Identify GEO accessions from the target study or publication (in the demuxmix paper, the cell line mixture benchmark corresponds to accessions GSM3501446 and GSM3501447). Download and load each dataset's (HTOs x droplets) HTO count matrix into R alongside any RNA count matrices needed for covariates such as genes detected per droplet. In the quality-survey workflow, 10 recently published HTO datasets are combined with 2 benchmark datasets, demuxmix's negative binomial regression mixture models are fit to each HTO count matrix, and the overlap score (ovs) quantifying overlap between fitted components is computed per dataset. Assemble per-dataset metrics (e.g., ovs values, correlation coefficients) into summary tables (overlap_scores.tsv, hto_gene_correlations.csv) and compare against reported reference ranges such as brain tagging r = 0.67-0.86, background r = 0.07-0.13, CSF PBMC r = 0.48, CSF r = 0.18, and category-average ovs of 0.019 (cell line) and 0.002 (human brain).

## Related tools

- **R** (Environment for downloading GEO datasets and loading HTO/RNA count matrices for downstream demuxmix analysis) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **demuxmix** (Fits negative binomial regression mixture models to the retrieved HTO count matrices and computes overlap scores for quality assessment) — https://github.com/huklein/demuxmix

## Examples

```
BiocManager::install("demuxmix"); library(demuxmix); dmm <- demuxmix(hto = hto_counts, rna = numGenes); ovs <- overlapScore(dmm)
```

## Evaluation signals

- Retrieved datasets yield the expected number of HTO count matrices (e.g., 10 published + 2 benchmark datasets for the quality survey)
- Reproduced category-average overlap scores match reported values of 0.019 (cell line) and 0.002 (human brain)
- Per-sample HTO-vs-gene correlations fall within reported ranges: brain tagging r = 0.67-0.86, background r = 0.07-0.13, CSF PBMC r = 0.48, CSF r = 0.18
- Summary tables (overlap_scores.tsv, hto_gene_correlations.csv) contain the required columns: dataset, sample, HTO type, metric value, reference-range match

## Limitations

- Some published HTO datasets exhibit quality comparable to simulated low-quality data (small HTO-signal scaling factor s), so downstream demultiplexing results are method-sensitive
- The real-dataset quality is heterogeneous: while some published datasets match benchmark quality, others fall significantly below it
- Correlation strength varies by tissue and dataset (cell line mixture shows weaker HTO-gene correlation than the brain dataset), so reference ranges are not universally transferable

## Evidence

- [full_text] Download the 10 recently published HTO datasets from GEO and load the HTO count matrices of the two benchmark datasets into R.: [evidence span withheld — anchor status: not_found]
- [full_text] While some published HTO datasets showed quality comparable to the benchmark datasets, others exhibited significantly lower quality, similar to the simulated datasets: [evidence span withheld — anchor status: not_found]
- [full_text] the cell line mixture dataset (GEO accessions GSM3501446 and GSM3501447): [evidence span withheld — anchor status: not_found]
- [readme] Demultiplexing error rates can be estimated, and methods for quality control are provided.: "Demultiplexing error rates can be estimated, and methods for quality control are provided."
