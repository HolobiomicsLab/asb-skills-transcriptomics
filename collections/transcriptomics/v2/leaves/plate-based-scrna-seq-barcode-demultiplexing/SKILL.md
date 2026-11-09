---
name: plate-based-scrna-seq-barcode-demultiplexing
description: Use when demultiplexing plate-prepared scRNA-seq FASTQ data from patterned-flowcell
  instruments (HiSeq 4000/HiSeq X/NovaSeq) where index hopping may misassign reads
  between sample libraries, and swapped-read fractions must be estimated or artefactual
  cross-sample libraries identified.
when_to_use_negative:
- Libraries sequenced on non-patterned flow-cell instruments such as the HiSeq 2500,
  where barcode swapping is not elevated (used as the negative control in the article)
- Droplet-based 10x Genomics data, where molecule-level swapped-read removal (DropletUtils)
  is the appropriate approach instead of plate-barcode demultiplexing
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Used to fit the simple linear regression of observed swapped reads on reads
    available for swapping and to compute mean and standard deviation of swapping
    fractions across plates
- name: HiSeq 4000
  role: Patterned flow-cell sequencer whose demultiplexed output contains the swapped
    reads being quantified
- name: DropletUtils
  role: Bioconductor package providing removal of barcode-swapped pseudo-cells for
    droplet data, complementary to the plate-based demultiplexing workflow
  repo: https://github.com/MarioniLab/DropletUtils
- name: BarcodeSwapping2017
  role: Repository containing the supplementary report and code that regenerates the
    demultiplexing and swapping-fraction analyses
  repo: https://github.com/MarioniLab/BarcodeSwapping2017
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:453fb99ac94ef6eb77cf25c397edc739b69e10c99156521936378c4704b9317e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - BarcodeSwapping2017
  - DropletUtils
  - HiSeq 4000
  - R
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# plate-based-scrna-seq-barcode-demultiplexing

## Summary

Demultiplex plate-based scRNA-seq libraries (e.g. Smart-seq2 plate data sequenced on HiSeq 4000/HiSeq X patterned flow-cells) allowing reads to be assigned to both expected and 'impossible' sample-barcode combinations, enabling quantification of barcode-swapped reads between samples.

## When to use

Use when demultiplexing plate-prepared scRNA-seq FASTQ data from patterned-flowcell instruments (HiSeq 4000/HiSeq X/NovaSeq) where index hopping may misassign reads between sample libraries, and swapped-read fractions must be estimated or artefactual cross-sample libraries identified.

## When NOT to use

- Libraries sequenced on non-patterned flow-cell instruments such as the HiSeq 2500, where barcode swapping is not elevated (used as the negative control in the article)
- Droplet-based 10x Genomics data, where molecule-level swapped-read removal (DropletUtils) is the appropriate approach instead of plate-barcode demultiplexing

## Inputs

- Raw BCL/FASTQ data from plate-prepared scRNA-seq libraries sequenced on a HiSeq 4000 (or HiSeq X/NovaSeq), e.g. ArrayExpress accession E-MTAB-6843
- Sample index barcode sequences for each plate-prepared library
- Reference genome (mm10) for downstream read mapping and gene-level counting

## Outputs

- Per-library read counts for expected and 'impossible' barcode combinations
- Swapped-read counts and reads-available-for-swapping per impossible combination
- Estimated swapped-read fraction (regression gradient) with cross-plate standard deviation, e.g. written to swapping_fraction_estimate.txt

## How to apply

Obtain raw plate-prepared data (e.g. ArrayExpress E-MTAB-6843 for the Richard et al. two-plate mouse T-cell dataset) and demultiplex the HiSeq 4000 libraries allowing reads to be assigned to libraries with both expected and 'impossible' barcode combinations. For each impossible combination, tabulate the observed swapped-read count and the number of reads available for swapping, then fit a simple linear regression of observed swapped reads on reads available for swapping across impossible combinations; the gradient of the fitted line estimates the swapped-read fraction. Compute the mean and standard deviation of swapping fractions across plates to report an uncertainty (Richard et al. HiSeq 4000 data yield ~2.18 ± 0.08%, consistent with the ~2.5% reported for HiSeq 4000). Map reads to the mm10 genome and count exonic read pairs per gene for downstream libraries. As a control, repeat the procedure with the same libraries sequenced on a non-patterned instrument such as the HiSeq 2500, where swapping should be negligible.

## Related tools

- **R** (Used to fit the simple linear regression of observed swapped reads on reads available for swapping and to compute mean and standard deviation of swapping fractions across plates)
- **HiSeq 4000** (Patterned flow-cell sequencer whose demultiplexed output contains the swapped reads being quantified)
- **DropletUtils** (Bioconductor package providing removal of barcode-swapped pseudo-cells for droplet data, complementary to the plate-based demultiplexing workflow) — https://github.com/MarioniLab/DropletUtils
- **BarcodeSwapping2017** (Repository containing the supplementary report and code that regenerates the demultiplexing and swapping-fraction analyses) — https://github.com/MarioniLab/BarcodeSwapping2017

## Examples

```
Rscript -e 'fit <- lm(swapped_reads ~ available_reads, data=impossible_combos); swapping_fraction <- coef(fit)[2]'
```

## Evaluation signals

- Estimated swapped-read fraction should be approximately 2.5% (Richard et al. HiSeq 4000 data: 2.18 ± 0.08%); report the deviation from this reference value
- Repeating the procedure on the same libraries sequenced on the HiSeq 2500 should give a near-zero swapping fraction (negative control)
- Impossible barcode combinations must be genuinely absent from the sample design; every counted 'impossible' library arises from swapping, not mis-specified sample sheets
- The regression of observed swapped reads on reads available for swapping should show a positive, approximately proportional relationship (nonzero gradient)
- Mean and standard deviation of swapping fractions should be computed across both plates, not from a single plate

## Limitations

- The swapped fraction applies to patterned flow-cell instruments (HiSeq 4000, HiSeq X, NovaSeq); it should not be extrapolated to non-patterned instruments
- Details of the swapping-fraction estimation via the regression gradient are deferred to Supplementary Note 3, which is unavailable in the extracted text ('No changelog found.')
- Cell exclusion by removing all barcodes shared between samples can incorrectly discard genuine cell libraries, so naive exclusion is lossy

## Evidence

- [discussion] Demultiplexing allowing assignment to expected and impossible barcode combinations: "Demultiplexing was performed allowing for reads to be assigned to libraries with both expected and “impossible” barcode combinations."
- [discussion] Linear regression of observed swapped reads vs reads available for swapping: "We performed a simple linear regression on the number of observed swapped reads for each impossible combination, with respect to the number of reads available for swapping."
- [discussion] HiSeq 2500 negative control: "As a control, we repeated this procedure with the same libraries sequenced on HiSeq 2500."
- [abstract] ~2.5% mislabelled reads on HiSeq 4000: "approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000, which is lower than previous reports"
- [readme] Repository contains supplementary report and code: [evidence span withheld — anchor status: external_doc]
