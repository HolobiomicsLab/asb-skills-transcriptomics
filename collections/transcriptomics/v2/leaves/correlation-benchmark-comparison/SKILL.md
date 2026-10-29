---
name: correlation-benchmark-comparison
description: Use when you need to reproduce or verify the reported HTO-count vs detected-gene
  correlations (e.g. brain tagging r = 0.67–0.86, background r = 0.07–0.13; CSF PBMC
  r = 0.48, CSF r = 0.18) before feeding the gene covariate into demuxmix.
when_to_use_negative:
- The dataset contains only HTO counts with no RNA gene count matrix, so the detected-genes-per-droplet
  covariate cannot be computed
- The goal is demultiplexing itself rather than validating the gene-count covariate
  — use the demuxmix classification workflow instead
- Cells were not labeled with hashtag oligonucleotides and no HTO count matrix exists
  (no tagging/background HTO distinction is meaningful)
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: demuxmix
  role: R/Bioconductor package whose regression mixture models consume the detected-genes
    covariate; the correlation benchmark justifies passing the total number of genes
    detected per droplet to demuxmix alongside HTO counts
  repo: https://github.com/huklein/demuxmix
- name: R
  role: Analysis environment used to load the HTO and gene count matrices, compute
    per-droplet detected-gene totals, and calculate per-sample correlations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:5ae09a1f4352fba3b28beef618992f3ea0606857714d6fcfb9caa115ff902532
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huklein/demuxmix
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# correlation-benchmark-comparison

## Summary

Quantifies per-sample Pearson correlations between HTO counts (tagging and background) and the number of detected genes per droplet, then benchmarks the computed values against published reference ranges to justify using detected genes as a covariate in demuxmix's regression mixture models.

## When to use

Use when you need to reproduce or verify the reported HTO-count vs detected-gene correlations (e.g. brain tagging r = 0.67–0.86, background r = 0.07–0.13; CSF PBMC r = 0.48, CSF r = 0.18) before feeding the gene covariate into demuxmix.

## When NOT to use

- The dataset contains only HTO counts with no RNA gene count matrix, so the detected-genes-per-droplet covariate cannot be computed
- The goal is demultiplexing itself rather than validating the gene-count covariate — use the demuxmix classification workflow instead
- Cells were not labeled with hashtag oligonucleotides and no HTO count matrix exists (no tagging/background HTO distinction is meaningful)

## Inputs

- HTO count matrix (HTOs x droplets) per dataset (human brain snRNA-seq, cell line mixture GSM3501446/GSM3501447, CSF)
- RNA gene count matrix (genes x droplets) per dataset
- Sample/dataset labels grouping droplets into samples
- Published reference correlation ranges for tagging and background HTOs

## Outputs

- hto_gene_correlations.csv with columns dataset, sample, HTO type (tagging/background), correlation coefficient, and reference-range match
- Per-sample correlation coefficients for tagging HTO counts vs detected genes
- Per-sample correlation coefficients for background HTO counts vs detected genes

## How to apply

Load the HTO count matrices ((HTOs x droplets)) and RNA gene count matrices ((genes x droplets)) in R for each dataset — e.g. the human brain snRNA-seq samples, the cell line mixture (GEO accessions GSM3501446 and GSM3501447), and the CSF dataset. For each droplet compute the total number of genes detected per droplet, the covariate demuxmix uses alongside HTO counts. Within each sample, compute the correlation of the tagging HTO counts with detected genes, and separately the correlation of the background HTO counts with detected genes, since the article reports tagging correlations are far stronger than background ones (0.67–0.86 vs 0.07–0.13 in the brain dataset). Assemble the per-sample tagging and background coefficients into a table and compare each value against the published reference ranges (brain tagging r = 0.67–0.86, brain background r = 0.07–0.13; CSF PBMC r = 0.48, CSF cells r = 0.18) to decide whether the positive association between detected genes and HTO counts holds for your data. The rationale is grounded in the paper's premise that demuxmix leverages the positive association between detected genes and HTO counts to explain variance in HTO data and improve droplet assignments.

## Related tools

- **demuxmix** (R/Bioconductor package whose regression mixture models consume the detected-genes covariate; the correlation benchmark justifies passing the total number of genes detected per droplet to demuxmix alongside HTO counts) — https://github.com/huklein/demuxmix
- **R** (Analysis environment used to load the HTO and gene count matrices, compute per-droplet detected-gene totals, and calculate per-sample correlations)

## Examples

```
Rscript -e 'library(demuxmix); nGenes <- colSums(geneCounts > 0); rTag <- cor(htoCounts[hto, ], nGenes); rBg <- cor(bgHtoCounts[hto, ], nGenes)'
```

## Evaluation signals

- Brain dataset tagging-HTO correlations fall within the reported range r = 0.67–0.86 in every sample
- Brain dataset background-HTO correlations fall within the reported range r = 0.07–0.13, i.e. much weaker than tagging correlations
- CSF dataset reproduces the reported values: tagged PBMC correlation ≈ 0.48 and non-tagged CSF cell correlation ≈ 0.18
- hto_gene_correlations.csv contains one row per sample per HTO type with non-missing correlation coefficients and a reference-range match flag
- Tagging correlations are consistently higher than background correlations within each dataset, matching the paper's reported pattern

## Limitations

- The cell line mixture dataset shows a weaker HTO–gene correlation than the brain dataset, so correlation strength is dataset-dependent and may not justify the gene covariate equally well everywhere
- Background HTO counts are only weakly correlated with detected genes (r ≈ 0.07–0.13), so the association does not generalize to non-tagged/background droplets
- The correlation justification assumes hashtag-labeled pooled cells; the paper demonstrates non-tagged cells pooled with labeled cells as a distinct, separate design

## Evidence

- [abstract] demuxmix utilizes the positive association between detected genes and HTO counts to explain variance in HTO data, improving droplet assignments: [evidence span withheld — anchor status: not_found]
- [intro] Leverage positive association between genes detected per droplet and HTO counts: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts to leverage the positive"
- [intro] HTOs sequenced together with RNA molecules producing an (HTOs x droplets) count matrix alongside the (genes x droplets) RNA matrix: "These oligonucleotides are termed hashtag oligonucleotides (HTOs) and are sequenced together with the RNA molecules of the cells resulting in an (HTOs x droplets) count matrix in addition to the"
- [readme] demuxmix is a package for demultiplexing pooled cells labeled with barcode oligonucleotides, used to load and analyze the HTO and gene count data in R: "demuxmix is a package for demultiplexing single-cell sequencing experiments of pooled cells labeled with barcode oligonucleotides."
