---
name: hiseq-4000-barcode-swapping-characterization
description: Use when raw plate-prepared scRNA-seq FASTQ/BAM data from a HiSeq 4000
  (or other patterned flow-cell) run must be demultiplexed with cross-sample swapped-read
  contamination quantified before downstream cell-level analysis.
when_to_use_negative:
- Data were sequenced on non-patterned flow-cell instruments (e.g. HiSeq 2500 alone),
  where barcode swapping is negligible and the regression would estimate near-zero
  signal.
- The dataset is droplet-based (10x Genomics); use molecule-level exclusion (e.g.
  DropletUtils removal of swapped pseudo-cells) rather than plate-based regression.
- Only processed count matrices are available — impossible barcode combinations cannot
  be tabulated without raw barcode-level reads.
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: R
  role: Statistical environment for fitting the linear regression of swapped reads
    on available-to-swap reads and computing cross-plate means/standard deviations
  repo: https://github.com/MarioniLab/BarcodeSwapping2017
- name: HiSeq 4000
  role: Patterned-flow-cell sequencer whose demultiplexed libraries are analysed;
    source of the ~2.5% swapped-read signal being quantified
- name: DropletUtils
  role: Companion Bioconductor package for removing barcode-swapped pseudo-cells in
    droplet-based (10x Genomics) data, complementary to this plate-based quantification
  repo: https://github.com/MarioniLab/DropletUtils
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:331e33904bf3f4dcb891b9679dbeaf344028a51ba3c466f94409076ee1dbdab2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/BarcodeSwapping2017
  tools:
  - DropletUtils
  - HiSeq 4000
  - R
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# hiseq-4000-barcode-swapping-characterization

## Summary

Quantifies the fraction of barcode-swapped reads in plate-based scRNA-seq libraries sequenced on patterned-flow-cell Illumina machines (HiSeq 4000) by regressing library sizes of impossible barcode combinations against reads available for swapping, and controls the estimate by re-running on the same libraries sequenced on a HiSeq 2500.

## When to use

Use when raw plate-prepared scRNA-seq FASTQ/BAM data from a HiSeq 4000 (or other patterned flow-cell) run must be demultiplexed with cross-sample swapped-read contamination quantified before downstream cell-level analysis.

## When NOT to use

- Data were sequenced on non-patterned flow-cell instruments (e.g. HiSeq 2500 alone), where barcode swapping is negligible and the regression would estimate near-zero signal.
- The dataset is droplet-based (10x Genomics); use molecule-level exclusion (e.g. DropletUtils removal of swapped pseudo-cells) rather than plate-based regression.
- Only processed count matrices are available — impossible barcode combinations cannot be tabulated without raw barcode-level reads.

## Inputs

- Raw plate-prepared scRNA-seq sequencing data (e.g. ArrayExpress E-MTAB-6843) with sample barcodes
- Sample-index barcode whitelist defining expected and impossible barcode combinations
- Control FASTQ/BAM from the same libraries sequenced on a HiSeq 2500 (for the swapping-negative control)

## Outputs

- Estimated swapped-read fraction (gradient of the regression line) with cross-plate standard deviation, e.g. written to swapping_fraction_estimate.txt
- Per-impossible-combination table of observed swapped reads vs reads available for swapping
- Control swapped-read fraction from HiSeq 2500-sequenced replicate libraries

## How to apply

Obtain raw data (e.g. ArrayExpress E-MTAB-6843 for the Richard et al. two-plate mouse T-cell dataset) and demultiplex the HiSeq 4000 libraries allowing reads to be assigned to both expected and 'impossible' sample-barcode combinations. For each impossible combination, tabulate the observed swapped-read count and the number of reads available for swapping, then fit a simple linear regression of observed swapped reads on reads available for swapping; the gradient of the fitted line estimates the swapped-read fraction. Compute the mean and standard deviation of swapping fractions across plates for uncertainty (Richard HiSeq 4000 estimate: 2.18 ± 0.08%, consistent with the reported ~2.5%). Run the identical procedure on the same libraries sequenced on a HiSeq 2500 as a negative control, since non-patterned flow cells exhibit negligible swapping. Map reads to mm10 and count exonic read pairs per gene when cell-level swapped contributions must be traced.

## Related tools

- **R** (Statistical environment for fitting the linear regression of swapped reads on available-to-swap reads and computing cross-plate means/standard deviations) — https://github.com/MarioniLab/BarcodeSwapping2017
- **HiSeq 4000** (Patterned-flow-cell sequencer whose demultiplexed libraries are analysed; source of the ~2.5% swapped-read signal being quantified)
- **DropletUtils** (Companion Bioconductor package for removing barcode-swapped pseudo-cells in droplet-based (10x Genomics) data, complementary to this plate-based quantification) — https://github.com/MarioniLab/DropletUtils

## Examples

```
bash get_data.sh && Rscript -e "rmarkdown::render('bcswap_supp.Rmd')"  # rebuilds the BarcodeSwapping2017 report containing the swapping-fraction regression
```

## Evaluation signals

- Estimated swapped-read fraction for the Richard et al. HiSeq 4000 data should be approximately 2.18 ± 0.08%, consistent with the reported ~2.5% (report the deviation).
- The same regression applied to HiSeq 2500-sequenced control libraries should yield a near-zero swapping fraction.
- The regression intercept should be close to zero if swapped reads scale linearly with reads available for swapping.
- Standard deviation across the two plates should be small relative to the mean estimate, indicating reproducibility.
- No correlation should be observed between the swapped fraction and free barcode concentration (BioAnalyzer 45-70 bp area).

## Limitations

- Key model details for the swapping-fraction regression and per-plate linear models are deferred to Supplementary Notes 3 and 4, which are unavailable in the main text.
- No correlation between swapped fraction and free barcode concentration was observed, so swapping severity cannot be predicted from library-preparation QC traces.
- Cell exclusion (removing all barcodes shared between samples) as an alternative remedy can incorrectly discard genuine cell libraries; the fraction incorrectly discarded was counted in simulations.

## Evidence

- [abstract] approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000, which is lower than previous reports: "approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000, which is lower than previous reports"
- [discussion] Demultiplexing with impossible combinations: [evidence span withheld — anchor status: not_found]
- [discussion] Regression and gradient estimate: "We performed a simple linear regression on the number of observed swapped reads for each impossible combination, with respect to the number of reads available for swapping."
- [discussion] HiSeq 2500 control: "As a control, we repeated this procedure with the same libraries sequenced on HiSeq 2500."
- [discussion] Cross-plate mean and SD: "The mean and standard deviation of swapping fractions were then computed across plates."
- [discussion] Raw data accession: "Raw data can be acquired from ArrayExpress accession codes E-MTAB-6843 for plate-prepared mouse T-cells"
- [readme] Supplementary report repo: [evidence span withheld — anchor status: external_doc]
