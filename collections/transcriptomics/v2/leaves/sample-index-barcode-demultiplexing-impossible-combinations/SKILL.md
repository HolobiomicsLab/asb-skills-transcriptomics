---
name: sample-index-barcode-demultiplexing-impossible-combinations
description: Use when FASTQ data from multiplexed single-cell libraries on a patterned
  flow-cell machine (HiSeq 4000/HiSeq X/NovaSeq) must be checked for swapped read
  fraction, especially with a same-library HiSeq 2500 control available.
when_to_use_negative:
- Data sequenced only on non-patterned flow-cell instruments (e.g. HiSeq 2500) where
  minimal swapping is expected and no patterned-flow-cell comparison exists
- Unmultiplexed libraries where reads cannot be attributed to sample-index combinations
- Droplet data where the goal is molecule-level removal of already-identified swapped
  molecules (use the DropletUtils cell/molecule exclusion algorithm instead)
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: HiSeq 4000
  role: Patterned flow-cell sequencer whose swapped read fraction is being estimated
    (~2.5% of reads mislabelled)
- name: HiSeq 2500
  role: Non-patterned control sequencer used to re-run the identical regression procedure
    on the same libraries
- name: R
  role: Environment in which the linear regression of swapped reads vs reads available
    for swapping is fitted and the gradient estimated (see MarioniLab/BarcodeSwapping2017
    report)
  repo: https://github.com/MarioniLab/BarcodeSwapping2017
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:6705b6bddc412152d3b63ad852673e3c388fbc20af8a2833ad9fb2da917a4112
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/BarcodeSwapping2017
  tools:
  - HiSeq 2500
  - HiSeq 4000
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# sample-index-barcode-demultiplexing-impossible-combinations

## Summary

A regression-based quantification of sample-index barcode swapping on patterned flow-cell Illumina sequencers (HiSeq 4000), where reads are demultiplexed to both expected and 'impossible' sample-index combinations and the swapped-read fraction is taken from the gradient of a linear regression.

## When to use

Use when FASTQ data from multiplexed single-cell libraries on a patterned flow-cell machine (HiSeq 4000/HiSeq X/NovaSeq) must be checked for swapped read fraction, especially with a same-library HiSeq 2500 control available.

## When NOT to use

- Data sequenced only on non-patterned flow-cell instruments (e.g. HiSeq 2500) where minimal swapping is expected and no patterned-flow-cell comparison exists
- Unmultiplexed libraries where reads cannot be attributed to sample-index combinations
- Droplet data where the goal is molecule-level removal of already-identified swapped molecules (use the DropletUtils cell/molecule exclusion algorithm instead)

## Inputs

- Raw BCL/FASTQ from the same libraries sequenced on both HiSeq 4000 and HiSeq 2500 (e.g. ArrayExpress E-MTAB-6854 droplet-prepared mouse epithelial cells)
- Sample-index barcode sequences used to prepare each library
- mm10 reference genome for read mapping

## Outputs

- Swapped-read fraction estimate per sequencer (regression gradient): ~2.5% (2.18 ± 0.08%) for HiSeq 4000 and ~0.22 ± 0.01% for HiSeq 2500
- Per-impossible-combination counts of observed swapped reads and reads available for swapping feeding the regression
- Gene-level count matrices of exonic read pairs per library combination

## How to apply

Obtain raw multiplexed droplet-prepared libraries (e.g. mouse epithelial cells from ArrayExpress E-MTAB-6854, sequenced on both HiSeq 4000 and HiSeq 2500). Demultiplex the reads allowing assignment to libraries with both expected and 'impossible' sample-index barcode combinations; impossible combinations can only arise from barcode swapping. Map reads to the mm10 genome and count exonic read pairs per gene for each library combination. Fit a simple linear regression of observed swapped reads per impossible combination against the number of reads available for swapping, and take the gradient of the fitted line as the swapped-read fraction. Repeat the identical procedure on the HiSeq 2500 runs of the same libraries as a control: swapping is expected to be specific to patterned flow cells, so the HiSeq 2500 gradient should be far lower (~0.22 ± 0.01% vs ~2.5% on the HiSeq 4000).

## Related tools

- **HiSeq 4000** (Patterned flow-cell sequencer whose swapped read fraction is being estimated (~2.5% of reads mislabelled))
- **HiSeq 2500** (Non-patterned control sequencer used to re-run the identical regression procedure on the same libraries)
- **R** (Environment in which the linear regression of swapped reads vs reads available for swapping is fitted and the gradient estimated (see MarioniLab/BarcodeSwapping2017 report)) — https://github.com/MarioniLab/BarcodeSwapping2017

## Evaluation signals

- The fitted regression gradient for the HiSeq 4000 run should be ~2.5% (reproducibly ~2.18 ± 0.08%) and substantially higher than previous reports
- The identical procedure on HiSeq 2500 runs of the same libraries must yield a much lower gradient (~0.22 ± 0.01%), confirming the patterned-flow-cell mechanism
- Swapped reads must be observed only for 'impossible' sample-index combinations absent from library preparation, not expected combinations
- Mean and standard deviation of swapping fractions should be computed across plates for reproducibility

## Limitations

- The swapped-fraction estimate depends on the regression-gradient method whose details are deferred to Supplementary Note 3, which may be unavailable from the main text alone
- Estimates are specific to the flow-cell chemistry and sequencer; swapping may generate complex but artefactual cell libraries in droplet-based scRNA-seq that require downstream exclusion methods
- No correlation was observed between swapped fraction and free barcode concentration, so barcode concentration cannot be used to predict or tune the effect

## Evidence

- [discussion] Demultiplexing allowing assignment to expected and impossible barcode combinations: "Demultiplexing was performed allowing for reads to be assigned to libraries with both expected and “impossible” barcode combinations."
- [discussion] Regression gradient as swapped-fraction estimate: "We used the gradient of the fitted line to obtain an estimate of the fraction of swapped reads for this experiment"
- [discussion] HiSeq 2500 control on same libraries: "As a control, we repeated this procedure with the same libraries sequenced on HiSeq 2500."
- [discussion] Mapping and counting step: "Reads were mapped to the mm10 genome, and the number of read pairs mapping to the exonic region of each gene were counted."
- [discussion] Reproducible ~2.5% on HiSeq 4000: "we have reproducibly estimated the fraction of barcode-swapped reads on the HiSeq 4000 to be approximately 2.5%, which is lower than previously reported1"
- [readme] DropletUtils offers barcode-swap removal utilities: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
