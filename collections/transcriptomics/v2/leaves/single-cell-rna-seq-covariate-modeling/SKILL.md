---
name: single-cell-rna-seq-covariate-modeling
description: Use when an (HTOs x droplets) count matrix and the total number of genes
  detected per droplet are both available, and more accurate probabilistic droplet
  assignment is needed than a naive HTO-only model provides.
when_to_use_negative:
- No per-droplet detected-gene counts are available and only HTO counts exist — the
  naive demuxmix variant is the appropriate (if weaker) option
- The data contain no hashtag-oligocode labels at all and cells cannot be distinguished
  by oligonucleotide barcodes
- Droplets have already been demultiplexed and only downstream per-sample analysis
  is needed
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: demuxmix
  role: Fits the negative binomial regression mixture model using HTO counts plus
    the detected-genes covariate and returns probabilistic droplet classifications
    with error probabilities
  repo: https://github.com/huklein/demuxmix
- name: R
  role: R/Bioconductor environment in which the demuxmix package is installed and
    run (BiocManager::install("demuxmix"))
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:57bcec8530d538b8c76e0b805177bc9e899188958041e5bc67139a32931cccd7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huklein/demuxmix
  tools:
  - R
  - demuxmix
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# single-cell-rna-seq-covariate-modeling

## Summary

Improves HTO-based demultiplexing of single-cell RNA-seq droplets by fitting demuxmix's negative binomial regression mixture model with the per-droplet number of detected genes as a covariate, exploiting the positive association between RNA content and HTO counts to explain variance and sharpen assignments.

## When to use

Use when an (HTOs x droplets) count matrix and the total number of genes detected per droplet are both available, and more accurate probabilistic droplet assignment is needed than a naive HTO-only model provides.

## When NOT to use

- No per-droplet detected-gene counts are available and only HTO counts exist — the naive demuxmix variant is the appropriate (if weaker) option
- The data contain no hashtag-oligocode labels at all and cells cannot be distinguished by oligonucleotide barcodes
- Droplets have already been demultiplexed and only downstream per-sample analysis is needed

## Inputs

- (HTOs x droplets) hashtag oligonucleotide count matrix from the HTO library
- Per-droplet number of detected genes from the RNA library (used as regression covariate)
- Optional genetic demultiplexing labels (e.g. freemuxlet) for validation

## Outputs

- Probabilistic per-droplet sample assignments with assignment error probabilities
- Set of uncertain/multiplet droplets whose error probability exceeds pAcpt (e.g. 0.99), to be discarded
- Estimated demultiplexing error rates and quality-control summaries (e.g. demuxmix_csf_evaluation.csv metrics)

## How to apply

Load the HTO count matrix together with the per-droplet number of detected genes (not the full transcription profile), and fit demuxmix's negative binomial regression mixture model by passing both inputs so the gene-count association explains variance in the HTO data. Set the acceptance-probability threshold (e.g. pAcpt=0.99) so droplets whose assignment error probability exceeds it are flagged as uncertain/multiplets and discarded. For benchmarking, run the naive demuxmix variant on the same HTO counts without the RNA covariate and compare specificity, sensitivity, and the uncertain rate against a ground truth such as freemuxlet genetic demultiplexing. On the CSF/PBMC one-HTO example, the covariate model reached 0.985 specificity / 0.980 sensitivity with 11.1% removed, versus 0.983 / 0.972 / 13.1% for the naive model.

## Related tools

- **demuxmix** (Fits the negative binomial regression mixture model using HTO counts plus the detected-genes covariate and returns probabilistic droplet classifications with error probabilities) — https://github.com/huklein/demuxmix
- **R** (R/Bioconductor environment in which the demuxmix package is installed and run (BiocManager::install("demuxmix")))

## Examples

```
library(demuxmix); dmm <- demuxmix(htoCounts, genes = nGenes, pAcpt = 0.99)
```

## Evaluation signals

- Against freemuxlet genetic ground truth at pAcpt=0.99, the covariate model should reach specificity ~0.985 and sensitivity ~0.980 on the CSF/PBMC dataset
- The covariate model should outperform the naive variant (0.983 specificity / 0.972 sensitivity / 13.1% removed)
- Uncertain droplet rate should be ~11.1% (261 cells removed) at pAcpt=0.99 for the covariate model
- Every accepted assignment must carry an error probability at or below the acceptance threshold; higher-probability droplets are flagged uncertain
- Per-method metrics are written to demuxmix_csf_evaluation.csv with specificity, sensitivity, and uncertain-rate columns

## Limitations

- Droplets with assignment error probability above pAcpt are discarded rather than resolved — 11.1% of cells were removed as uncertain in the CSF/PBMC evaluation
- Only the total number of genes detected per droplet is used as covariate, not the complete transcription profile; the benefit depends on the strength of the genes–HTO count association
- The naive model without the RNA covariate performs measurably worse (specificity 0.983, sensitivity 0.972), so omitting the covariate costs accuracy

## Evidence

- [abstract] covariate improves assignments: "demuxmix utilizes the positive association between detected genes in the RNA library and HTO counts to explain parts of the variance in the HTO data resulting in improved droplet assignments"
- [intro] which covariate to pass: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts to leverage the positive"
- [readme] probabilistic classification with multiplet detection: "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection. Demultiplexing error rates can be estimated, and methods"
- [abstract] non-labeled pooled-cell designs are in scope: "Finally, the feasibility of accurately demultiplexing experimental designs where non-labeled cells are pooled with labeled cells is demonstrated."
