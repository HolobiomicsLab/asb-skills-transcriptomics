---
name: bioanalyzer-trace-area-quantification
description: Use when BioAnalyzer Expert traces from plate-based scRNA-seq library
  preparation are available and you need per-sample free-barcode-to-cDNA ratios to
  test their association with estimated barcode-swapping fractions.
when_to_use_negative:
- No BioAnalyzer Expert traces exist for the libraries, so free barcode concentration
  cannot be quantified from trace areas.
- The sequencing instrument is not a patterned flow-cell machine (e.g. HiSeq 4000/HiSeq
  X/NovaSeq), so free-barcode-driven index swapping is not the expected artifact.
- The goal is detecting/removing swapped molecules in 10x Genomics droplet data —
  use the DropletUtils molecule-exclusion algorithm instead.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: BioAnalyzer Expert software
  role: Integrates electropherogram trace areas over 45–70 bp (free barcode) and 400–800
    bp (sequencable cDNA) windows
- name: R
  role: Fits the linear regression of per-plate swapping fractions against free-barcode-to-cDNA
    ratios
  repo: https://github.com/MarioniLab/BarcodeSwapping2017
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:69ed40bc576a7c0a01f785170b1e75afd72d8f605e4394c06bf11d099a916ceb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/BarcodeSwapping2017
  tools:
  - BioAnalyzer Expert software
  - R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# bioanalyzer-trace-area-quantification

## Summary

Quantify free barcode and sequencable cDNA concentrations from BioAnalyzer Expert electropherogram traces collected during scRNA-seq library preparation, by integrating the area under the curve over defined DNA-length windows (45–70 bp for free barcode, 400–800 bp for cDNA). These per-plate ratios are used to test whether free barcode abundance explains barcode-swapping rates on patterned flow-cell instruments such as the HiSeq 4000.

## When to use

Use when BioAnalyzer Expert traces from plate-based scRNA-seq library preparation are available and you need per-sample free-barcode-to-cDNA ratios to test their association with estimated barcode-swapping fractions.

## When NOT to use

- No BioAnalyzer Expert traces exist for the libraries, so free barcode concentration cannot be quantified from trace areas.
- The sequencing instrument is not a patterned flow-cell machine (e.g. HiSeq 4000/HiSeq X/NovaSeq), so free-barcode-driven index swapping is not the expected artifact.
- The goal is detecting/removing swapped molecules in 10x Genomics droplet data — use the DropletUtils molecule-exclusion algorithm instead.

## Inputs

- BioAnalyzer Expert electropherogram traces collected during library preparation for each plate
- Per-plate swapping fraction estimates (regression gradients from HiSeq 2500 vs HiSeq 4000 plate models)
- Nestorowa et al. count matrices (NCBI GEO GSE81682) for context on plate/HiSeq 4000 sample pairing

## Outputs

- Per-plate free barcode concentration (trace area 45–70 bp)
- Per-plate sequencable cDNA concentration (trace area 400–800 bp)
- Per-plate free-barcode-to-cDNA ratios
- Linear regression coefficients and p-value (expected p = 0.129, slope not significantly different from zero)
- swapping_fraction_vs_free_barcode.csv

## How to apply

For each of the 16 Nestorowa plates, open the BioAnalyzer Expert trace and compute the area under the curve between 45 and 70 base pairs to quantify free library barcode concentration, and the area between 400 and 800 base pairs to quantify sequencable cDNA concentration. Form the free-barcode-to-cDNA ratio per plate. Regress the previously estimated per-plate swapping fractions (gradients of linear models quantifying HiSeq 2500 cell contributions to HiSeq 4000 transcriptomes, mean 2.275 ± 0.359%) against these ratios with a linear model in R, and check whether the slope differs significantly from zero. In this analysis the regression was non-significant (p = 0.129), supporting the conclusion that swapping extent is not primarily determined by free barcode amount. Write per-plate swapping fractions, ratios, and regression coefficients/p-values to swapping_fraction_vs_free_barcode.csv.

## Related tools

- **BioAnalyzer Expert software** (Integrates electropherogram trace areas over 45–70 bp (free barcode) and 400–800 bp (sequencable cDNA) windows)
- **R** (Fits the linear regression of per-plate swapping fractions against free-barcode-to-cDNA ratios) — https://github.com/MarioniLab/BarcodeSwapping2017

## Examples

```
ratio <- area_45_70 / area_400_800; fit <- lm(swapping_fraction ~ ratio, data = plates); write.csv(plates, "swapping_fraction_vs_free_barcode.csv")
```

## Evaluation signals

- Per-plate swapping fraction estimates reproduce the reported mean 2.275 ± 0.359% across the 16 Nestorowa plates.
- Trace integration windows are exactly 45–70 bp for free barcode and 400–800 bp for cDNA; areas are positive and finite for all plates.
- Regression of swapping fraction on free-barcode-to-cDNA ratio yields a slope not significantly different from zero (p = 0.129 in the published analysis).
- Output file swapping_fraction_vs_free_barcode.csv contains per-plate swapping fractions, ratios, and regression coefficients/p-values.
- Free barcode concentrations are smaller in magnitude than sequencable cDNA concentrations (short-fragment area is the smaller component of library profiles).

## Limitations

- Key method details for the swapping-fraction estimation are deferred to Supplementary Notes 3–4, which are absent from the main text, so exact regression implementations may need the BarcodeSwapping2017 supplementary report.
- The analysis is specific to plate-based Smart-seq2 libraries sequenced on both HiSeq 2500 and HiSeq 4000; results do not transfer directly to droplet-based protocols.
- Cell exclusion by removing barcodes shared between samples can incorrectly discard genuine cell libraries, which affects upstream swapping-fraction estimates used in the regression.

## Evidence

- [discussion] Free barcode area window: "we used the BioAnalyzer Expert software on the traces collected during library preparation, considering the area under the curve between DNA lengths 45 to 70 base pairs"
- [discussion] cDNA area window: "To quantify the concentration of sequencable cDNA, we considered the area under the curve between DNA lengths 400 to 800 base pairs."
- [abstract] No correlation finding: "We observed no correlation between the swapped fraction of reads and the concentration of free barcode across plates."
- [discussion] Data provenance: "We obtained count matrices from the authors of the Nestorowa et al. study10 (also available at NCBI GEO accession GSE81682)."
- [readme] Repo provides supplementary report with models and results: "A Github repository (https://github.com/MarioniLab/BarcodeSwapping2017) contains a detailed report that expands on the analyses described herein, describing models and showing results."
