---
name: hto-gene-count-correlation-analysis
description: Use when preparing HTO-based demultiplexing with demuxmix and you need
  to verify the positive association between tagging HTO counts and detected genes
  per droplet before passing gene counts as a covariate.
when_to_use_negative:
- Input data lacks paired HTO and RNA count matrices for the same droplets (e.g.,
  HTO-only assays), since detected genes cannot be computed
- The goal is demultiplexing itself rather than validating the gene-count covariate;
  use demuxmix's regression mixture model fitting directly
- Droplets have not been segmented per sample, so tagging vs background HTO identity
  cannot be distinguished
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: demuxmix
  role: R/Bioconductor package whose regression mixture models consume the detected-genes
    covariate; supplies the HTO/RNA data structures this correlation analysis validates
  repo: https://github.com/huklein/demuxmix
- name: R
  role: Environment in which the HTO and gene count matrices are loaded, per-droplet
    detected gene counts are computed, and correlations are calculated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:79a6911b20663330d3864d81addbda6a4d03aa5781766673c8fb59fbc5331010
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huklein/demuxmix
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# hto-gene-count-correlation-analysis

## Summary

Quantifies per-sample correlations between hashtag oligonucleotide (HTO) counts and the number of detected genes per droplet across datasets, to justify using detected genes as the covariate in demuxmix's negative binomial regression mixture models.

## When to use

Use when preparing HTO-based demultiplexing with demuxmix and you need to verify the positive association between tagging HTO counts and detected genes per droplet before passing gene counts as a covariate.

## When NOT to use

- Input data lacks paired HTO and RNA count matrices for the same droplets (e.g., HTO-only assays), since detected genes cannot be computed
- The goal is demultiplexing itself rather than validating the gene-count covariate; use demuxmix's regression mixture model fitting directly
- Droplets have not been segmented per sample, so tagging vs background HTO identity cannot be distinguished

## Inputs

- HTO count matrix (HTOs x droplets) per dataset/sample
- RNA gene count matrix (genes x droplets) per dataset/sample
- Dataset/sample metadata (e.g., GEO accessions GSM3501446 and GSM3501447 for the cell line mixture)

## Outputs

- Per-sample correlation table (hto_gene_correlations.csv) with columns: dataset, sample, HTO type (tagging/background), correlation coefficient, reference-range match

## How to apply

Load the (HTOs x droplets) HTO count matrix and the (genes x droplets) RNA count matrix in R for each dataset (e.g., human brain snRNA-seq, cell line mixture from GEO GSM3501446/GSM3501447, and CSF). For each droplet, compute the total number of genes detected — the covariate demuxmix uses alongside HTO counts. Then, per sample, compute the Pearson correlation between the tagging HTO counts and detected genes, and separately between background HTO counts and detected genes. Assemble these into a table and compare against the reported reference ranges (brain tagging r = 0.67-0.86, background r = 0.07-0.13; CSF PBMC r = 0.48, CSF r = 0.18) to confirm that tagging HTOs, not background, track RNA content, which is the rationale for passing gene counts to demuxmix.

## Related tools

- **demuxmix** (R/Bioconductor package whose regression mixture models consume the detected-genes covariate; supplies the HTO/RNA data structures this correlation analysis validates) — https://github.com/huklein/demuxmix
- **R** (Environment in which the HTO and gene count matrices are loaded, per-droplet detected gene counts are computed, and correlations are calculated)

## Examples

```
library(demuxmix); genesDetected <- colSums(rnaCounts > 0); cor per sample: cor(htoCounts[g, ], genesDetected)
```

## Evaluation signals

- Per-sample tagging HTO vs detected-gene correlations in the human brain dataset fall within the reported r = 0.67-0.86 range
- Per-sample background HTO vs detected-gene correlations in the brain dataset fall within the reported weak r = 0.07-0.13 range
- CSF dataset reproduces the reported tagging correlations (PBMC r = 0.48; non-tagged CSF cells r = 0.18)
- hto_gene_correlations.csv contains all required columns (dataset, sample, HTO type, correlation coefficient, reference-range match) with one row per sample per HTO type
- Tagging correlations are substantially larger than background correlations within the same dataset, confirming the positive association demuxmix exploits

## Limitations

- Correlation strength is dataset-dependent: the cell line mixture shows a weaker correlation than the brain dataset, so reference ranges from one dataset do not transfer directly
- Background HTO counts correlate only weakly with detected genes (r = 0.07-0.13 in brain), so gene counts explain variance mainly for tagging HTOs, not background HTOs

## Evidence

- [intro] detected genes as demuxmix covariate justified by correlation: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts to leverage the positive"
- [abstract] demuxmix uses gene-HTO association: "demuxmix utilizes the positive association between detected genes in the RNA library and HTO counts to explain parts of the variance in the HTO data resulting in improved droplet assignments"
- [intro] HTO and gene count matrix structures: "sequenced together with the RNA molecules of the cells resulting in an (HTOs x droplets) count matrix in addition to the"
- [readme] installing the package in R: "BiocManager::install("demuxmix")"
- [readme] package scope includes error estimation and QC: "Demultiplexing error rates can be estimated, and methods for quality control are provided."
