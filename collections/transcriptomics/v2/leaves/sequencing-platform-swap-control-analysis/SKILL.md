---
name: sequencing-platform-swap-control-analysis
description: Use when a barcode-swapping fraction has been estimated on a patterned
  flow-cell instrument (HiSeq 4000/X/NovaSeq) and you need a same-libraries control
  on a non-patterned instrument (HiSeq 2500) to attribute swapping to the sequencer
  rather than the libraries.
when_to_use_negative:
- The libraries were prepared for or sequenced only on non-patterned flow-cell instruments
  where swapping is expected to be at background — the control adds no discriminating
  information
- The analysis goal is demultiplexing or molecule-level de-swapping in 10x Genomics
  data (use the DropletUtils cell/molecule exclusion algorithm instead of the regression
  control)
- Sample-index combinations are unique dual indexes where impossible combinations
  cannot arise, so the impossible-combination regression cannot be fitted
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: R
  role: Fitting the simple linear regression of observed swapped reads vs reads available
    for swapping and extracting the gradient as the swapped fraction
  repo: https://github.com/MarioniLab/BarcodeSwapping2017
- name: HiSeq 2500
  role: Non-patterned flow-cell instrument providing the control sequencing runs of
    the same libraries
- name: HiSeq 4000
  role: Patterned flow-cell instrument whose swapping estimate (~2.5%) serves as the
    comparator
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:bfa4b7e9dd63c208ceb44e2875631a99b56046d9eaa8faf8b8389ac5f1e1ebee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/BarcodeSwapping2017
  tools:
  - HiSeq 2500
  - HiSeq 4000
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# sequencing-platform-swap-control-analysis

## Summary

A control procedure that quantifies the barcode-swapped read fraction attributable to a specific sequencing platform by repeating the impossible-combination regression on the same libraries sequenced on an alternative instrument (e.g. HiSeq 2500 vs HiSeq 4000), isolating patterned-flow-cell-specific swapping from library-preparation effects.

## When to use

Use when a barcode-swapping fraction has been estimated on a patterned flow-cell instrument (HiSeq 4000/X/NovaSeq) and you need a same-libraries control on a non-patterned instrument (HiSeq 2500) to attribute swapping to the sequencer rather than the libraries.

## When NOT to use

- The libraries were prepared for or sequenced only on non-patterned flow-cell instruments where swapping is expected to be at background — the control adds no discriminating information
- The analysis goal is demultiplexing or molecule-level de-swapping in 10x Genomics data (use the DropletUtils cell/molecule exclusion algorithm instead of the regression control)
- Sample-index combinations are unique dual indexes where impossible combinations cannot arise, so the impossible-combination regression cannot be fitted

## Inputs

- Raw FASTQ of droplet-prepared mouse epithelial cell libraries (ArrayExpress E-MTAB-6854), HiSeq 2500 runs of the same libraries used in the HiSeq 4000 analysis
- Expected sample-index barcode combinations and the set of 'impossible' combinations per sample pair
- mm10 genome reference for read mapping

## Outputs

- Exonic read-pair count matrix per gene for each library combination (expected and impossible)
- Fitted linear regression of observed swapped reads vs reads available for swapping per impossible combination
- Swapped-read fraction estimate (gradient of fitted line) with uncertainty, e.g. 0.22 ± 0.01% for HiSeq 2500
- Platform comparison showing the HiSeq 2500 control is substantially lower than the HiSeq 4000 estimate (~2.18-2.5%)

## How to apply

Download the same droplet-prepared libraries from ArrayExpress E-MTAB-6854 that were used for the HiSeq 4000 analysis, but use the HiSeq 2500 runs. Demultiplex allowing reads to be assigned to both expected and 'impossible' sample-index barcode combinations, map reads to the mm10 genome, and count exonic read pairs per gene for each library combination. Fit a simple linear regression of observed swapped reads per impossible barcode combination against the number of reads available for swapping, and take the gradient of the fitted line as the swapped-read fraction. The rationale is that barcode swapping is specific to patterned flow-cell Illumina machines, so the same libraries sequenced on the HiSeq 2500 should yield a much lower swapped fraction (0.22 ± 0.01% vs 2.18 ± 0.08% on the HiSeq 4000, ~2.5% paper-level estimate). Compare the HiSeq 2500 gradient against the HiSeq 4000 estimate to confirm the platform-specific mechanism.

## Related tools

- **R** (Fitting the simple linear regression of observed swapped reads vs reads available for swapping and extracting the gradient as the swapped fraction) — https://github.com/MarioniLab/BarcodeSwapping2017
- **HiSeq 2500** (Non-patterned flow-cell instrument providing the control sequencing runs of the same libraries)
- **HiSeq 4000** (Patterned flow-cell instrument whose swapping estimate (~2.5%) serves as the comparator)

## Examples

```
git clone https://github.com/MarioniLab/BarcodeSwapping2017 && ./get_data.sh && Rscript -e "rmarkdown::render('bcswap_supp.Rmd')"
```

## Evaluation signals

- The fitted-regression swapped fraction for the HiSeq 2500 run must be substantially lower than the reproducible ~2.5% HiSeq 4000 estimate (reported control: 0.22 ± 0.01%)
- Regression must be fitted over impossible barcode combinations only, with the gradient (not intercept) used as the swapped-fraction estimate
- Read counts must come from exonic, mm10-mapped read pairs per gene per library combination, matching the HiSeq 4000 pipeline exactly
- Demultiplexing must have allowed assignment to both expected and impossible sample-index combinations; otherwise impossible-combination counts are absent and the regression fails
- The control estimate should be computed on the identical libraries (E-MTAB-6854) as the HiSeq 4000 analysis so the only changing factor is the sequencer

## Limitations

- The estimate depends on demultiplexing tolerating impossible barcode combinations; standard stringent demultiplexing discards the signal needed for the regression
- Precise details of the gradient-based swapping-fraction estimation are deferred to Supplementary Note 3, which is unavailable in the extracted document ('No changelog found.')
- The control validates a platform-specific mechanism only for libraries also run on the patterned instrument; swapping rates may vary with library prep and barcode concentration (no correlation with free barcode concentration was observed, but this was tested on plates)

## Evidence

- [discussion] Demultiplexing allowing assignment to expected and impossible barcode combinations: "Demultiplexing was performed allowing for reads to be assigned to libraries with both expected and “impossible” barcode combinations."
- [discussion] Control: repeated procedure on same libraries sequenced on HiSeq 2500: "As a control, we repeated this procedure with the same libraries sequenced on HiSeq 2500."
- [discussion] Swapping fraction estimated from the gradient of the fitted regression line: "We used the gradient of the fitted line to obtain an estimate of the fraction of swapped reads for this experiment"
- [discussion] Dataset accession for droplet-prepared libraries: "Raw data can be acquired from ArrayExpress accession codes E-MTAB-6843 for plate-prepared mouse T-cells"
- [readme] DropletUtils README context: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells, and downsampling of the count"
