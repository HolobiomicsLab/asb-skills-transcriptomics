---
name: hto-based-cell-demultiplexing
description: Use when a pooled scRNA-seq experiment has an (HTOs x droplets) count
  matrix from hashtag-oligonucleotide labeling (including one-HTO designs with unlabeled
  cells) and each droplet must be assigned to its sample of origin or flagged as a
  multiplet.
when_to_use_negative:
- Cells were not labeled with hashtag oligonucleotides and no HTO count matrix exists
  (use genetic or other demultiplexing approaches instead)
- Full transcription profiles per cell are available but only HTO counts are needed
  — passing the complete RNA profile is unnecessary; only the number of detected genes
  per droplet should accompany HTO counts
- The input is already demultiplexed (e.g. cells already carry sample-of-origin labels),
  so re-running classification on HTO counts would be redundant
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: demuxmix
  role: Fits negative binomial regression mixture models to HTO counts (optionally
    regressing on detected genes) to produce probabilistic cell/multiplet assignments
    at a chosen pAcpt threshold
  repo: https://github.com/huklein/demuxmix
- name: R
  role: Runtime environment; demuxmix is an R/Bioconductor package installed via BiocManager
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:e497032d188f3dc788e079dc4832bad0984b5fc7021e16566004a7c8695ec4ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huklein/demuxmix
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# hto-based-cell-demultiplexing

## Summary

Demultiplex droplets in oligonucleotide-barcoded single-cell RNA-seq experiments by probabilistically classifying cells from their HTO count matrix with demuxmix's negative binomial regression mixture models, optionally leveraging the number of detected genes as an RNA covariate.

## When to use

Use when a pooled scRNA-seq experiment has an (HTOs x droplets) count matrix from hashtag-oligonucleotide labeling (including one-HTO designs with unlabeled cells) and each droplet must be assigned to its sample of origin or flagged as a multiplet.

## When NOT to use

- Cells were not labeled with hashtag oligonucleotides and no HTO count matrix exists (use genetic or other demultiplexing approaches instead)
- Full transcription profiles per cell are available but only HTO counts are needed — passing the complete RNA profile is unnecessary; only the number of detected genes per droplet should accompany HTO counts
- The input is already demultiplexed (e.g. cells already carry sample-of-origin labels), so re-running classification on HTO counts would be redundant

## Inputs

- (HTOs x droplets) hashtag-oligonucleotide count matrix
- Per-droplet number of detected genes (RNA covariate) from the gene x droplet RNA count matrix
- Optional genetic-demultiplexing labels (e.g. freemuxlet) as ground truth for evaluation
- Acceptance-probability threshold pAcpt (e.g. 0.99)

## Outputs

- Probabilistic per-droplet sample assignments with assignment error probabilities
- Set of uncertain droplets / multiplets whose error probability exceeds pAcpt
- Estimated demultiplexing error rates and quality-control summaries
- Evaluation metrics (specificity, sensitivity, uncertain rate) when ground-truth labels are available, e.g. written to demuxmix_csf_evaluation.csv

## How to apply

Load the HTO count matrix (and, when available, the per-droplet number of detected genes) and call demuxmix, setting the acceptance-probability threshold pAcpt (e.g. 0.99) so droplets whose assignment error probability exceeds it are discarded as uncertain/multiplets. Prefer passing the total number of genes detected per droplet — not the full transcription profile — so the negative binomial regression mixture model can exploit the positive association between detected genes and HTO counts; running the naive variant without the RNA covariate serves as a baseline comparison and was shown to perform worse (e.g. on the CSF/PBMC one-HTO dataset with pAcpt=0.99, specificity 0.983 vs 0.985 and 13.1% vs 11.1% uncertain). For designs pooling non-labeled rare cells (e.g. CSF) with HTO-labeled cells (e.g. PBMCs), a single HTO suffices to separate negative (untagged) from positive (tagged) cells. Validate against an independent ground truth such as genetic demultiplexing (freemuxlet) by computing specificity, sensitivity, and the uncertain-droplet rate.

## Related tools

- **demuxmix** (Fits negative binomial regression mixture models to HTO counts (optionally regressing on detected genes) to produce probabilistic cell/multiplet assignments at a chosen pAcpt threshold) — https://github.com/huklein/demuxmix
- **R** (Runtime environment; demuxmix is an R/Bioconductor package installed via BiocManager) — https://doi.org/doi:10.18129/B9.bioc.demuxmix

## Examples

```
library(demuxmix); dmm <- demuxmix(hto = htoCounts, rna = numGenes, pAcpt = 0.99); classifyCells(dmm)
```

## Evaluation signals

- Against a freemuxlet genetic-demultiplexing ground truth, expect specificity ≈ 0.985 and sensitivity ≈ 0.980 on the CSF/PBMC one-HTO dataset at pAcpt=0.99
- The uncertain/multiplet fraction at pAcpt=0.99 should be ≈ 11.1% of droplets on the CSF/PBMC example (naive variant ≈ 13.1%)
- Metrics from the regression (genes-covariate) model should dominate the naive mixture model on specificity, sensitivity, and uncertain rate
- Every classified droplet carries an error probability; exactly the droplets exceeding the pAcpt threshold are removed as uncertain
- Package loads cleanly in R via library(demuxmix) after BiocManager::install("demuxmix")

## Limitations

- Droplets whose assignment error probability exceeds the acceptance threshold are discarded as uncertain — on the CSF/PBMC example this removed 261 cells (11.1%) at pAcpt=0.99, so a fraction of data is always lost
- The naive mixture model variant that ignores the RNA covariate performs measurably worse, so omitting the detected-genes input degrades results
- Performance was assessed against genetic demultiplexing (freemuxlet); accuracy depends on the quality of that ground truth in evaluation settings

## Evidence

- [abstract] Improved assignments via detected-genes association: "demuxmix utilizes the positive association between detected genes in the RNA library and HTO counts to explain parts of the variance in the HTO data"
- [intro] Use per-droplet detected gene counts, not full profiles: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts"
- [readme] Package purpose and probabilistic classification: "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection."
- [abstract] Benchmarked against existing methods: "The improved performance of demuxmix compared with existing demultiplexing methods is assessed using real and simulated data."
