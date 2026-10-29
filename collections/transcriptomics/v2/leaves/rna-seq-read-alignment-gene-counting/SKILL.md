---
name: rna-seq-read-alignment-gene-counting
description: Use when raw scRNA-seq FASTQ data (e.g. ArrayExpress E-MTAB-6854 droplet-prepared
  mouse epithelial libraries) must be converted into a per-gene exonic read-pair count
  matrix for each demultiplexed sample combination.
when_to_use_negative:
- Input is already a gene-by-cell count matrix (e.g. the Nestorowa et al. GSE81682
  count matrices obtained directly from the authors) — skip alignment and proceed
  to downstream modelling
- The dataset is human or another non-mouse organism, since the reference described
  here is specifically the mm10 mouse genome
- The goal is 10x Genomics molecule-level deconvolution, where the molecule-information-file
  based exclusion in MarioniLab/DropletUtils is the appropriate route instead of raw
  read-pair counting
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: R
  role: Implementation language for demultiplexing, mm10 mapping/assignment of exonic
    read pairs per gene, and the subsequent regression of swapped reads on reads available
    for swapping
  repo: https://github.com/MarioniLab/BarcodeSwapping2017
- name: HiSeq 4000
  role: Source instrument whose patterned-flow-cell runs produce the high swapping-fraction
    (~2.5%) read data being aligned and counted
- name: HiSeq 2500
  role: Source instrument for the non-patterned control runs of the same libraries,
    used to verify the lower (0.22 ± 0.01%) swapped fraction
- name: DropletUtils
  role: Bioconductor utilities for droplet scRNA-seq data handling (loading count
    matrices, cell calling, barcode-swapped pseudo-cell removal) in related downstream
    steps
  repo: https://bioconductor.org/packages/DropletUtils
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:c0a7b408dcfaa2a2999657dea519874bffa893b76f6013d2be44a2198ad2ec32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/BarcodeSwapping2017
  tools:
  - DropletUtils
  - HiSeq 2500
  - HiSeq 4000
  - R
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# rna-seq-read-alignment-gene-counting

## Summary

Align single-cell RNA-seq reads to the mouse mm10 genome and count exonic read pairs per gene per sample/barcode combination, producing the gene-by-sample count matrix needed for downstream analyses such as barcode-swapping regression in the Griffiths/Richard 2018 pipeline.

## When to use

Use when raw scRNA-seq FASTQ data (e.g. ArrayExpress E-MTAB-6854 droplet-prepared mouse epithelial libraries) must be converted into a per-gene exonic read-pair count matrix for each demultiplexed sample combination.

## When NOT to use

- Input is already a gene-by-cell count matrix (e.g. the Nestorowa et al. GSE81682 count matrices obtained directly from the authors) — skip alignment and proceed to downstream modelling
- The dataset is human or another non-mouse organism, since the reference described here is specifically the mm10 mouse genome
- The goal is 10x Genomics molecule-level deconvolution, where the molecule-information-file based exclusion in MarioniLab/DropletUtils is the appropriate route instead of raw read-pair counting

## Inputs

- Raw FASTQ/BCL sequencing data from ArrayExpress accession E-MTAB-6854 (droplet-prepared mouse epithelial cell libraries, HiSeq 2500 and HiSeq 4000 runs)
- Sample-index barcode sequences for expected and impossible demultiplexing combinations
- mm10 mouse reference genome

## Outputs

- Count matrix of exonic read pairs per gene for each library/barcode combination (including impossible combinations carrying swapped reads)
- Per-combination read totals used as the 'reads available for swapping' covariate in the linear regression

## How to apply

Demultiplex the sequencing reads allowing assignment to both expected and 'impossible' sample-index barcode combinations so that putatively swapped reads are retained as separate libraries. Map the read pairs to the mm10 genome and count, for each library combination, the read pairs mapping to the exonic region of each gene, yielding a gene-by-library count matrix. In the source workflow this matrix is the direct input to the linear regression of observed swapped reads per impossible combination against reads available for swapping, whose gradient estimates the swapped-read fraction on HiSeq 4000 versus HiSeq 2500 runs. Perform the counting in R as part of the reproducible report in MarioniLab/BarcodeSwapping2017, downloading the raw data with the repo's get_data.sh script before rendering the .Rmd.

## Related tools

- **R** (Implementation language for demultiplexing, mm10 mapping/assignment of exonic read pairs per gene, and the subsequent regression of swapped reads on reads available for swapping) — https://github.com/MarioniLab/BarcodeSwapping2017
- **HiSeq 4000** (Source instrument whose patterned-flow-cell runs produce the high swapping-fraction (~2.5%) read data being aligned and counted)
- **HiSeq 2500** (Source instrument for the non-patterned control runs of the same libraries, used to verify the lower (0.22 ± 0.01%) swapped fraction)
- **DropletUtils** (Bioconductor utilities for droplet scRNA-seq data handling (loading count matrices, cell calling, barcode-swapped pseudo-cell removal) in related downstream steps) — https://bioconductor.org/packages/DropletUtils

## Examples

```
bash get_data.sh; Rscript -e "rmarkdown::render('bcswap_supp.Rmd')"
```

## Evaluation signals

- Count matrix contains counts for both expected and impossible barcode combinations; impossible combinations are non-empty on HiSeq 4000 data but near-zero on HiSeq 2500 controls
- Fitted linear regression of observed swapped reads vs reads available for swapping yields a gradient of approximately 2.5% (2.18 ± 0.08%) for HiSeq 4000 and substantially lower, 0.22 ± 0.01%, for HiSeq 2500
- Each counted read pair maps to the mm10 genome and falls in the exonic region of exactly one gene, so total per-library counts are traceable to the demultiplexed read pool
- Re-running the pipeline via get_data.sh + rmarkdown::render() reproduces the counts supporting the published swapping estimates

## Limitations

- Exonic read-pair counting on mm10 is mouse-specific; other genomes require an equivalent reference and annotation
- Regression-based swapped-fraction estimation details are deferred to Supplementary Note 3, and the electronic supplementary material was unavailable in the indexed document ('No changelog found.'), so some parameterization must be recovered from the BarcodeSwapping2017 report code
- The counting step alone does not remove swapped molecules; deconvolution or exclusion (e.g. cell/molecule exclusion, DropletUtils) is required for cleaned count matrices

## Evidence

- [discussion] Reads mapped to mm10 and exonic read pairs counted per gene: "Reads were mapped to the mm10 genome, and the number of read pairs mapping to the exonic region of each gene were counted."
- [discussion] Demultiplexing retained impossible barcode combinations: [evidence span withheld — anchor status: not_found]
- [discussion] Regression gradient estimates the swapped-read fraction: "We used the gradient of the fitted line to obtain an estimate of the fraction of swapped reads for this experiment"
- [readme] Repo pipeline: download data then render report: [evidence span withheld — anchor status: external_doc]
