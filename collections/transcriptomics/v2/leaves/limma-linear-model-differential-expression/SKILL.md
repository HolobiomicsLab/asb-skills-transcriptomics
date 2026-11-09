---
name: limma-linear-model-differential-expression
description: Use when you need differential expression between a genetic/pharmacological
  perturbation group and MDI-induced control samples from the curated 3T3-L1 microarray
  data, with per-gene log2FC, moderated p and BH-FDR.
when_to_use_negative:
- Input is raw probe-level microarray data straight from GEO — the pipeline must first
  collapse probes to gene symbols and remove missing/low-intensity genes before fitting
  the model
- Input is RNA-Seq count data (curatedAdipoRNA RangedSummarizedExperiment) or ChIP-Seq
  peak counts (curatedAdipoChIP), where a count-based model rather than this microarray-intensity
  limma workflow applies
- Goal is gene set enrichment or batch-effect exploration rather than a two-group
  differential expression contrast
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: limma
  role: Log transform and normalize gene intensities across studies and fit the linear
    model for the knockdown-vs-control contrast, producing moderated statistics
- name: sva
  role: Remove known batch effects (data series and platform) from gene intensities
    prior to differential expression modeling
- name: curatedAdipoArray
  role: Supplies the processed, batch-corrected SummarizedExperiment microarray dataset
    consumed by the limma model
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: ExperimentHub
  role: Access point for the clean and processed SummarizedExperiment objects
- name: BiocManager
  role: Install the curatedAdipoArray package from Bioconductor
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:036574f04c7519ca1088981e70e4003bdaf773c42c0342a4ce3d7b4a5d1528c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - BiocManager
  - ExperimentHub
  - R
  - R/Bioconductor
  - curatedAdipoArray
  - limma
  - sva
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# limma-linear-model-differential-expression

## Summary

Fit a limma linear model on normalized, batch-corrected gene intensities from the curatedAdipoArray SummarizedExperiment object to extract log2 fold-change, moderated p-value and Benjamini-Hochberg FDR for a perturbation-vs-control contrast (e.g. Pparg knockdown vs MDI-induced controls) in differentiating 3T3-L1 cells.

## When to use

Use when you need differential expression between a genetic/pharmacological perturbation group and MDI-induced control samples from the curated 3T3-L1 microarray data, with per-gene log2FC, moderated p and BH-FDR.

## When NOT to use

- Input is raw probe-level microarray data straight from GEO — the pipeline must first collapse probes to gene symbols and remove missing/low-intensity genes before fitting the model
- Input is RNA-Seq count data (curatedAdipoRNA RangedSummarizedExperiment) or ChIP-Seq peak counts (curatedAdipoChIP), where a count-based model rather than this microarray-intensity limma workflow applies
- Goal is gene set enrichment or batch-effect exploration rather than a two-group differential expression contrast

## Inputs

- Processed gene-expression SummarizedExperiment object from the curatedAdipoArray Bioconductor package (retrieved through ExperimentHub)
- Sample metadata (colData) classifying samples as none, genetically or pharmacologically modified, used to define knockdown vs MDI-control groups
- List of 48 previously validated PPARG target genes for the recovery check

## Outputs

- Differential expression table (gene, log2FC, moderated p-value, Benjamini-Hochberg FDR) written to pparg_kd_differential_expression.csv
- Filtered list of genes down-regulated by >= 1 log2 fold in the knockdown
- Count of recovered validated PPARG targets (expected 29/48)

## How to apply

Load the processed gene-expression SummarizedExperiment object from curatedAdipoArray via ExperimentHub and subset samples into the perturbation group (e.g. Pparg knockdown, n=27) and MDI-induced controls (n=30). Fit a limma linear model on the normalized, batch-corrected gene intensities for the knockdown-vs-control contrast; the upstream pipeline had already removed missing and low-intensity genes, log-transformed and normalized intensities across studies with limma, and removed known batch effects (data series and platform) with sva. Extract log2 fold-change, moderated p-value and Benjamini-Hochberg FDR per gene, then filter for genes changed by one or more log2 fold. Validate biologically: lipid metabolism genes Lpl, Cidec and Cd36 should be down-regulated at |log2FC| >= 1, and the significant down-regulated set should recover 29 of 48 validated PPARG target genes. Write the full differential expression table to CSV for reuse.

## Related tools

- **limma** (Log transform and normalize gene intensities across studies and fit the linear model for the knockdown-vs-control contrast, producing moderated statistics)
- **sva** (Remove known batch effects (data series and platform) from gene intensities prior to differential expression modeling)
- **curatedAdipoArray** (Supplies the processed, batch-corrected SummarizedExperiment microarray dataset consumed by the limma model) — https://github.com/MahShaaban/curatedAdipoArray
- **ExperimentHub** (Access point for the clean and processed SummarizedExperiment objects)
- **BiocManager** (Install the curatedAdipoArray package from Bioconductor)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("curatedAdipoArray")
```

## Evaluation signals

- Lpl, Cidec and Cd36 appear among genes down-regulated at |log2FC| >= 1 in the Pparg knockdown contrast
- 29 of the 48 previously validated PPARG target genes are recovered in the significant down-regulated gene list
- Output CSV contains log2 fold-change, moderated p-value and Benjamini-Hochberg FDR for every gene
- Input gene intensities are log-transformed, cross-study normalized and free of known batch effects (data series and platform, removed with sva) before model fitting
- Contrast groups are correctly sized (e.g. Pparg knockdown n=27 vs MDI control n=30) per the SummarizedExperiment colData

## Limitations

- Differential expression on the curated data depends on prior removal of missing and low-intensity genes and cross-study normalization; skipping these steps invalidates the contrast
- Batch effects must already be removed (data series and platform via sva); residual batch structure would confound the limma model
- The dataset covers MDI-induced 3T3-L1 microarray samples under the specific genetic/pharmacological perturbations curated from 43 published studies; contrasts outside these perturbation types are not supported

## Evidence

- [intro] missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`.: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
- [intro] the known batch effects (data series and platform) were removed using `sva`.: "the known batch effects (data series and platform) were removed using `sva`."
- [intro] The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment.: "The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment."
- [readme] Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`.: "Access to the clean and the processed data `SummarizedExperiment` objects
through `ExperimentHub`."
