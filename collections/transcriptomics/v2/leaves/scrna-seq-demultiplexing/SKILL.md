---
name: scrna-seq-demultiplexing
description: Use when pooled scRNA-seq libraries were labeled with hashtag oligonucleotides
  and each droplet in an (HTOs x droplets) HTO count matrix must be assigned to its
  sample of origin (including multiplet detection), with uncertain droplets discarded
  via error probabilities.
when_to_use_negative:
- The library contains no sequenced hashtag oligonucleotides — without an (HTOs x
  droplets) HTO count matrix there is nothing for the negative binomial mixture models
  to fit.
- Input is already a per-droplet sample-assignment table; this skill performs the
  probabilistic assignment step itself, not downstream integration of already-demultiplexed
  cells.
- Sample-of-origin must be inferred from genotype/SNP data rather than oligonucleotide
  barcodes — the paper's scope is demultiplexing pooled cells labeled with sample-specific
  barcode oligonucleotides.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: demuxmix
  role: 'R/Bioconductor package that runs the whole workflow: per-HTO preprocessing,
    EM fitting of naive and negative binomial regression mixture models, model selection
    by expected classification error, and posterior-probability classification of
    droplets including multiplet detection'
  repo: https://github.com/huklein/demuxmix
- name: R
  role: Runtime environment in which the HTO count matrix and genes-detected vector
    are loaded and the demuxmix package is executed (library(demuxmix))
- name: BiocManager
  role: Installs the demuxmix package from Bioconductor before running the demultiplexing
    workflow (BiocManager::install("demuxmix"))
  repo: https://bioconductor.org
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:9e79502dafef98d4fdf49d2d81ba72ea6dc0b44ed86777ffdfb8f4f526f19445
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huklein/demuxmix
  tools:
  - BiocManager
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# scrna-seq-demultiplexing

## Summary

Demultiplex pooled, hashtag-oligonucleotide (HTO)-barcoded single-cell RNA-seq by fitting per-HTO negative binomial mixture models (naive and genes-detected regression variants) to the (HTOs x droplets) count matrix and classifying each droplet to its sample of origin or a multiplet by posterior probability, discarding droplets below the acceptance threshold.

## When to use

Use when pooled scRNA-seq libraries were labeled with hashtag oligonucleotides and each droplet in an (HTOs x droplets) HTO count matrix must be assigned to its sample of origin (including multiplet detection), with uncertain droplets discarded via error probabilities.

## When NOT to use

- The library contains no sequenced hashtag oligonucleotides — without an (HTOs x droplets) HTO count matrix there is nothing for the negative binomial mixture models to fit.
- Input is already a per-droplet sample-assignment table; this skill performs the probabilistic assignment step itself, not downstream integration of already-demultiplexed cells.
- Sample-of-origin must be inferred from genotype/SNP data rather than oligonucleotide barcodes — the paper's scope is demultiplexing pooled cells labeled with sample-specific barcode oligonucleotides.

## Inputs

- HTO count matrix with hashtag oligonucleotides as rows and droplets as columns (HTOs x droplets)
- Number of genes detected per droplet (optional regression covariate, derived from the (genes x droplets) RNA read count matrix)
- Number of HTOs in the experiment (n), which sets the default acceptance threshold pAcpt = 0.9^n

## Outputs

- Per-droplet demultiplexing assignments to sample of origin or multiplet class, each with a posterior probability (e.g., written to demuxmix_results.csv)
- Estimated demultiplexing error rates and quality-control plots from the demuxmix package

## How to apply

Load the (HTOs x droplets) HTO count matrix and the number of genes detected per droplet into R, then run demuxmix's per-HTO preprocessing on each hashtag oligonucleotide's count distribution. For each HTO, fit negative binomial mixture models via the EM algorithm — both the naive model and the regression model using genes-detected-per-droplet as covariate — and select the model minimizing the expected classification error; the regression variant exploits the positive association between detected genes and HTO counts to explain variance in HTO data and improve assignments. Classify each droplet to its sample of origin or a multiplet by posterior probability, applying the acceptance threshold pAcpt (default 0.9^n, where n is the number of HTOs) so uncertain droplets are discarded rather than force-assigned. Write per-droplet assignments with posterior probabilities (e.g., demuxmix_results.csv), then use demuxmix's error-rate estimation and quality-control methods to judge whether the demultiplexing succeeded.

## Related tools

- **demuxmix** (R/Bioconductor package that runs the whole workflow: per-HTO preprocessing, EM fitting of naive and negative binomial regression mixture models, model selection by expected classification error, and posterior-probability classification of droplets including multiplet detection) — https://github.com/huklein/demuxmix
- **R** (Runtime environment in which the HTO count matrix and genes-detected vector are loaded and the demuxmix package is executed (library(demuxmix)))
- **BiocManager** (Installs the demuxmix package from Bioconductor before running the demultiplexing workflow (BiocManager::install("demuxmix"))) — https://bioconductor.org

## Examples

```
library(demuxmix); dmm <- demuxmix(htoCounts, numGenes = nGenesDetected); res <- dmmClassify(dmm)
```

## Evaluation signals

- Every droplet (column of the HTOs x droplets matrix) appears exactly once in the output with either a sample-of-origin/multiplet label plus posterior probability or a discarded/uncertain status.
- For every HTO, both the naive and the genes-detected regression negative binomial mixture models were fit via EM, and the selected model records the lower expected classification error.
- No droplet with posterior probability below the acceptance threshold pAcpt (default 0.9^n, n = number of HTOs) carries a hard sample assignment — uncertain droplets are discarded instead.
- Estimated demultiplexing error rates and QC plots show well-separated negative/positive HTO count distributions per hashtag; on simulated or real data with known sample origins, assignment accuracy should match the improved performance demuxmix reports over existing demultiplexing methods.

## Limitations

- Droplets whose posterior probability falls below the acceptance threshold pAcpt (default 0.9^n, n = number of HTOs) are discarded, so uncertain droplets receive no sample assignment.
- The regression mixture model requires the total number of genes detected per droplet as covariate; better results are often achieved only when this is supplied alongside HTO counts, otherwise only the naive model can be fit and assignments may be worse.
- The method targets pooled cells labeled with sample-specific barcode oligonucleotides; designs pooling non-labeled cells with labeled cells are supported but were demonstrated (not guaranteed) in the paper's feasibility assessment.

## Evidence

- [abstract] Uncertain droplets can be discarded using assignment error probabilities: "provides error probabilities for droplet assignments that can be used to discard uncertain droplets"
- [abstract] Performance benchmarked on real and simulated data against existing demultiplexing methods: "The improved performance of demuxmix compared with existing demultiplexing methods is assessed using real and simulated data."
- [intro] Method core: demultiplex droplets from HTO counts via negative binomial regression mixture models: "The *demuxmix* package implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models."
- [intro] HTO oligonucleotides sequenced with RNA yield an (HTOs x droplets) count matrix: "These oligonucleotides are termed hashtag oligonucleotides (HTOs) and are sequenced together with the RNA molecules of the cells"
- [intro] Genes detected per droplet improve assignments as a regression covariate: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts"
- [methods] Skill scope: demultiplexing oligonucleotide-labeled scRNA-seq data: "Demultiplexing oligonucleotide-labeled scRNA-seq data with demuxmix"
- [readme] Package estimates demultiplexing error rates and provides QC methods: "Demultiplexing error rates can be estimated, and methods for quality control are provided."
