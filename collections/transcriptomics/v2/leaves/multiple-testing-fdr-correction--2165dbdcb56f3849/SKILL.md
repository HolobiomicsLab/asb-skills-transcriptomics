---
name: multiple-testing-fdr-correction
description: Use when a differential expression test has been run across many genes
  (e.g. limma on a SummarizedExperiment of microarray intensities) and per-gene p-values
  must be converted to genome-wide FDR before thresholding.
when_to_use_negative:
- When only a single hypothesis or a pre-specified small gene set is tested and no
  multiplicity correction is needed
- When input is raw probe-level intensities rather than the normalized, batch-corrected
  SummarizedExperiment produced by the curatedAdipoArray pipeline
- When the analysis is differential peak binding or RNA-seq count-based testing outside
  the limma/microarray framework described here
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: limma
  role: Fits the linear model, computes moderated p-values and normalizes across studies;
    the BH FDR is extracted from the limma differential expression table
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: curatedAdipoArray
  role: Supplies the processed SummarizedExperiment gene-intensity object (via ExperimentHub)
    that serves as input to the limma fit and FDR correction
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: R/Bioconductor
  role: Runtime environment in which the limma model and BH FDR extraction are performed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:a57fb889a21ffc54623f8be0e2b6b26aaebb7195c3ef4ba334514ec33a94681d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - ExperimentHub
  - R
  - R/Bioconductor
  - curatedAdipoArray
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multiple-testing-fdr-correction

## Summary

Apply Benjamini-Hochberg false discovery rate correction to moderated p-values from a limma differential expression fit (here, Pparg-knockdown vs MDI-induced control 3T3-L1 cells) so that per-gene calls remain valid across thousands of tested genes.

## When to use

Use when a differential expression test has been run across many genes (e.g. limma on a SummarizedExperiment of microarray intensities) and per-gene p-values must be converted to genome-wide FDR before thresholding.

## When NOT to use

- When only a single hypothesis or a pre-specified small gene set is tested and no multiplicity correction is needed
- When input is raw probe-level intensities rather than the normalized, batch-corrected SummarizedExperiment produced by the curatedAdipoArray pipeline
- When the analysis is differential peak binding or RNA-seq count-based testing outside the limma/microarray framework described here

## Inputs

- Normalized, batch-corrected gene-intensity matrix from the curatedAdipoArray SummarizedExperiment object (Pparg-knockdown n=27 vs MDI-induced control n=30)
- Limma linear-model fit for the knockdown-vs-control contrast with per-gene moderated p-values

## Outputs

- Per-gene differential expression table with log2 fold-change, moderated p-value and Benjamini-Hochberg FDR (written to pparg_kd_differential_expression.csv)
- Filtered list of significant down-regulated genes (|log2FC| >= 1) used for target-gene overlap checks

## How to apply

Fit a limma linear model on the normalized, batch-corrected gene intensities for the knockdown-vs-control contrast and extract, for every gene, the log2 fold-change, moderated p-value and Benjamini-Hochberg FDR. Use the FDR column (not raw p-values) when filtering genes, e.g. requiring down-regulation of one or more log2 fold in the knockdown to detect genes such as Lpl, Cidec and Cd36. Intersect the FDR-filtered down-regulated list with a validated target set (here the 48 known PPARG targets, 29 of which were recovered) as a biological sanity check. Report the full table including log2FC, moderated p-value and BH FDR so downstream users can re-threshold without refitting the model.

## Related tools

- **limma** (Fits the linear model, computes moderated p-values and normalizes across studies; the BH FDR is extracted from the limma differential expression table) — https://github.com/MahShaaban/curatedAdipoArray
- **curatedAdipoArray** (Supplies the processed SummarizedExperiment gene-intensity object (via ExperimentHub) that serves as input to the limma fit and FDR correction) — https://github.com/MahShaaban/curatedAdipoArray
- **R/Bioconductor** (Runtime environment in which the limma model and BH FDR extraction are performed)

## Examples

```
BiocManager::install("curatedAdipoArray"); fit <- limma::lmFit(assay(se[, se$group %in% c("Pparg_kd","MDI_control")]), design); fit <- limma::eBayes(fit); tab <- limma::topTable(fit, number = Inf, adjust.method = "BH")
```

## Evaluation signals

- The output table contains a BH FDR column alongside log2FC and moderated p-value for every gene
- Lpl, Cidec and Cd36 are down-regulated in the knockdown with |log2FC| >= 1
- 29 of the 48 previously validated PPARG target genes are recovered among the significant down-regulated genes
- No raw p-value is used for gene filtering without a corresponding FDR threshold
- Full differential expression table written to pparg_kd_differential_expression.csv

## Limitations

- Batch effects (data series and platform) must be removed (e.g. with sva) before the limma fit, otherwise FDR calls are confounded
- FDR results depend on the curated metadata and normalization performed across studies; two datasets were excluded for missing probe annotations and one for missing probe intensities
- Interpretation is limited to MDI-induced 3T3-L1 microarray data as curated in curatedAdipoArray; RNA-seq or ChIP-seq contrasts require the companion packages

## Evidence

- [intro] The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment.: "The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment."
- [intro] missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`.: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
- [intro] the known batch effects (data series and platform) were removed using `sva`.: "the known batch effects (data series and platform) were removed using `sva`."
- [readme] The package contains two things: 1. Scripts for documenting and reproducing the dataset in `inst/scripts`. 2. Access to the clean and the processed data: "Access to the clean and the processed data `SummarizedExperiment` objects
through `ExperimentHub`."
- [full_text] 2. Fit a limma linear model on the normalized, batch-corrected gene intensities for the knockdown-vs-control contrast and extract log2 fold-change, moderated p-value and Benjamini-Hochberg FDR: [evidence span withheld — anchor status: not_found]
