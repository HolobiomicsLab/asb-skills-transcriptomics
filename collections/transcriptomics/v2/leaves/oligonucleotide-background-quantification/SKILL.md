---
name: oligonucleotide-background-quantification
description: Use when you have an (HTOs x droplets) count matrix alongside an RNA
  count matrix and need to quantify tagging vs background HTO signal per droplet,
  or to validate the positive association between HTO counts and detected genes before
  regression-based demultiplexing.
when_to_use_negative:
- The experiment has no hashtag oligonucleotide (HTO) library — only an RNA expression
  matrix with no (HTOs x droplets) count matrix exists
- The goal is cell assignment/demultiplexing itself rather than validating or quantifying
  HTO signal against detected genes — use the full demuxmix classification workflow
  instead
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: demuxmix
  role: R/Bioconductor package providing the regression mixture model framework whose
    detected-genes covariate the HTO-vs-gene correlations justify, and whose HTO count
    inputs this skill quantifies
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Analysis environment used to load HTO and RNA count matrices, compute per-droplet
    detected-gene totals, and calculate tagging/background correlation coefficients
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:3e02401ccee607a2a07cecc08c2df724f62285049cc0fe549b348f91f2d3e0ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# oligonucleotide-background-quantification

## Summary

Quantifies per-droplet HTO oligonucleotide counts, separating the tagging HTO (assigned to each nucleus) from background HTO counts, and correlates them with the number of detected genes per droplet to justify detected genes as an explanatory covariate in demuxmix's negative binomial regression mixture models.

## When to use

Use when you have an (HTOs x droplets) count matrix alongside an RNA count matrix and need to quantify tagging vs background HTO signal per droplet, or to validate the positive association between HTO counts and detected genes before regression-based demultiplexing.

## When NOT to use

- The experiment has no hashtag oligonucleotide (HTO) library — only an RNA expression matrix with no (HTOs x droplets) count matrix exists
- The goal is cell assignment/demultiplexing itself rather than validating or quantifying HTO signal against detected genes — use the full demuxmix classification workflow instead

## Inputs

- HTO count matrix (HTOs x droplets) from oligonucleotide-labeled scRNA-seq experiments
- RNA gene count matrix (genes x droplets) for the same droplets
- Sample/dataset metadata (human brain snRNA-seq, cell line mixture GSM3501446/GSM3501447, CSF dataset)

## Outputs

- Per-droplet detected-gene totals used as the demuxmix regression covariate
- Per-sample correlation coefficients for tagging HTO counts vs detected genes
- Per-sample correlation coefficients for background HTO counts vs detected genes
- hto_gene_correlations.csv with columns: dataset, sample, HTO type (tagging/background), correlation coefficient, reference-range match

## How to apply

In R, load the HTO count matrices and gene count matrices for each dataset (human brain snRNA-seq, cell line mixture from GEO accessions GSM3501446/GSM3501447, and CSF). For each droplet, compute the total number of genes detected, the covariate demuxmix uses alongside HTO counts. For each sample, compute the Pearson correlation between the tagging HTO counts and detected genes, and separately between background HTO counts and detected genes. Expect strong tagging correlations (brain dataset r = 0.67–0.86; CSF PBMC r = 0.48; non-tagged CSF cells r = 0.18) and weak background correlations (brain r = 0.07–0.13), since demuxmix exploits the positive association between detected genes and HTO counts to explain variance. Assemble dataset, sample, HTO type, and correlation coefficient into hto_gene_correlations.csv and compare against these reference ranges.

## Related tools

- **demuxmix** (R/Bioconductor package providing the regression mixture model framework whose detected-genes covariate the HTO-vs-gene correlations justify, and whose HTO count inputs this skill quantifies) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Analysis environment used to load HTO and RNA count matrices, compute per-droplet detected-gene totals, and calculate tagging/background correlation coefficients) — https://doi.org/doi:10.18129/B9.bioc.demuxmix

## Examples

```
BiocManager::install("demuxmix"); library(demuxmix)
```

## Evaluation signals

- Tagging HTO vs detected-gene correlations in the human brain dataset fall in the reported range r = 0.67–0.86 across all samples
- Background HTO vs detected-gene correlations in the brain dataset are weak, within r = 0.07–0.13
- CSF dataset reproduces tagging correlation r = 0.48 for tagged PBMCs and r = 0.18 for non-tagged CSF cells
- Cell line mixture dataset shows a tagging correlation weaker than the brain dataset's range
- hto_gene_correlations.csv contains one row per sample per HTO type with dataset, sample, HTO type, coefficient, and reference-range match columns populated

## Limitations

- The association strength varies by dataset: the cell line mixture shows a weaker correlation than the brain dataset, so correlation magnitudes from one dataset cannot be assumed for another
- Background HTO counts correlate only weakly with detected genes (brain r = 0.07–0.13), so the detected-genes covariate explains variance mainly in tagging, not background, HTO counts
- Non-tagged cells (e.g. CSF cells) show lower correlation (r = 0.18) than tagged populations, limiting the covariate's usefulness in pooled designs with non-labeled cells

## Evidence

- [abstract] demuxmix utilizes the positive association between detected genes and HTO counts to explain parts of the variance in the HTO data: [evidence span withheld — anchor status: not_found]
- [intro] better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts to leverage the positive: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts"
- [intro] These oligonucleotides are termed hashtag oligonucleotides (HTOs) and are sequenced together with the RNA molecules of the cells resulting in an (HTOs x droplets) count matrix: "These oligonucleotides are termed hashtag oligonucleotides (HTOs) and are sequenced together with the RNA molecules of the cells resulting in an (HTOs x droplets) count matrix"
- [readme] The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection.: "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection."
