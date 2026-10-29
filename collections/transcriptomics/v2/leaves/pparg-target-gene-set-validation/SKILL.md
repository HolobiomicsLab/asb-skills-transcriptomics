---
name: pparg-target-gene-set-validation
description: Use when a gene-perturbation microarray analysis must be validated against
  a known transcription-factor target set, e.g. confirming Pparg knockdown effects
  by recovering previously validated PPARG targets among significant DE genes.
when_to_use_negative:
- Input is raw probe-level microarray intensities rather than the processed, batch-corrected
  gene-level SummarizedExperiment object — the skill assumes curation and normalization
  (GEOquery retrieval, collapseRows probe-to-gene collapsing, limma normalization,
  sva batch removal) have already been done.
- The perturbation is not Pparg knockdown in MDI-induced 3T3-L1 cells — the PPARG
  target-gene validation set and the Lpl/Cidec/Cd36 marker expectation are specific
  to this contrast.
- The goal is RNA-seq or ChIP-seq analysis of the same model — use the curatedAdipoRNA
  or curatedAdipoChIP packages instead, whose SummarizedExperiment objects (adipo_counts
  / peak_counts) support different data types.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: curatedAdipoArray
  role: Source of the curated 3T3-L1 microarray SummarizedExperiment data used for
    the Pparg-knockdown differential expression analysis
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: limma
  role: Fits the linear model on normalized gene intensities and computes log2 fold-change,
    moderated p-values and BH FDR for the knockdown-vs-control contrast
- name: ExperimentHub
  role: Provides access to the clean and processed SummarizedExperiment data objects
- name: BiocManager
  role: Installs the curatedAdipoArray package from Bioconductor
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:3afaf0f9b99d02d2cf1f62ac41cd1d99a182726066696a262bba3d49871193f1
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# pparg-target-gene-set-validation

## Summary

Validate PPARG-dependent transcriptional reprogramming by intersecting limma differential-expression results from Pparg-knockdown vs MDI-induced control 3T3-L1 cells (curatedAdipoArray data) with a set of 48 previously validated PPARG target genes, requiring recovery of 29 of them.

## When to use

Use when a gene-perturbation microarray analysis must be validated against a known transcription-factor target set, e.g. confirming Pparg knockdown effects by recovering previously validated PPARG targets among significant DE genes.

## When NOT to use

- Input is raw probe-level microarray intensities rather than the processed, batch-corrected gene-level SummarizedExperiment object — the skill assumes curation and normalization (GEOquery retrieval, collapseRows probe-to-gene collapsing, limma normalization, sva batch removal) have already been done.
- The perturbation is not Pparg knockdown in MDI-induced 3T3-L1 cells — the PPARG target-gene validation set and the Lpl/Cidec/Cd36 marker expectation are specific to this contrast.
- The goal is RNA-seq or ChIP-seq analysis of the same model — use the curatedAdipoRNA or curatedAdipoChIP packages instead, whose SummarizedExperiment objects (adipo_counts / peak_counts) support different data types.

## Inputs

- Processed gene-expression SummarizedExperiment object from the curatedAdipoArray Bioconductor package (accessed via ExperimentHub)
- Pparg-knockdown sample subset (n=27) and MDI-induced control sample subset (n=30)
- List of 48 previously validated PPARG target genes

## Outputs

- Differential expression table written to pparg_kd_differential_expression.csv with log2FC, moderated p-value and BH FDR per gene
- Filtered list of genes down-regulated by >= 1 log2 fold in the knockdown
- Intersection count of recovered PPARG target genes (expected: 29/48)

## How to apply

Load the processed gene-expression SummarizedExperiment object from the curatedAdipoArray package via ExperimentHub and subset samples into the Pparg-knockdown group (n=27) and the MDI-induced control group (n=30). Fit a limma linear model on the normalized, batch-corrected gene intensities (batch effects from data series and platform previously removed with sva) for the knockdown-vs-control contrast, extracting log2 fold-change, moderated p-value, and Benjamini-Hochberg FDR per gene. Filter for genes down-regulated by one or more log2 fold in the knockdown and confirm key lipid metabolism genes Lpl, Cidec, and Cd36 appear among them (|log2FC| >= 1). Intersect this significant down-regulated gene list with the 48 previously validated PPARG target genes; acceptance requires that 29 of the 48 targets are recovered, consistent with failed adipocyte differentiation under Pparg loss. Write the full differential expression table to pparg_kd_differential_expression.csv for downstream reuse.

## Related tools

- **curatedAdipoArray** (Source of the curated 3T3-L1 microarray SummarizedExperiment data used for the Pparg-knockdown differential expression analysis) — https://github.com/MahShaaban/curatedAdipoArray
- **limma** (Fits the linear model on normalized gene intensities and computes log2 fold-change, moderated p-values and BH FDR for the knockdown-vs-control contrast)
- **ExperimentHub** (Provides access to the clean and processed SummarizedExperiment data objects)
- **BiocManager** (Installs the curatedAdipoArray package from Bioconductor)

## Examples

```
BiocManager::install("curatedAdipoArray")
```

## Evaluation signals

- Lpl, Cidec and Cd36 are down-regulated in the knockdown with |log2FC| >= 1
- 29 of the 48 previously validated PPARG target genes are recovered in the significant down-regulated gene list
- Sample grouping sizes match the design: Pparg-knockdown n=27 and MDI-induced control n=30
- The full differential expression table (log2FC, moderated p-value, BH FDR per gene) is written to pparg_kd_differential_expression.csv
- Down-regulated genes at one or more log2 fold are consistent with failed adipocyte differentiation phenotype

## Limitations

- The curated dataset excludes studies with missing probe annotations (two studies excluded) and missing probe intensities (one study excluded), which constrains the underlying data completeness.
- Recovery of PPARG targets depends on prior batch-effect removal (data series and platform via sva) and cross-study normalization (limma); failures in these upstream steps propagate to the intersection result.
- The dataset is microarray-based and limited to MDI-induced 3T3-L1 samples with genetic or pharmacological perturbations; results may not transfer to other cell models or assay platforms.

## Evidence

- [readme] readme: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
- [intro] intro: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
- [abstract] abstract: "Finally, we used a subset of the data to effectively remove batch effects and reproduce biological observations."
