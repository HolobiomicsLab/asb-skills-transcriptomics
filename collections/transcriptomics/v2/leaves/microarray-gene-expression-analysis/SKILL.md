---
name: microarray-gene-expression-analysis
description: Use when the goal is differential gene expression (e.g. knockdown-vs-control
  contrasts) on MDI-induced 3T3-L1 microarray samples with curated, normalized gene-level
  intensities available as a SummarizedExperiment object.
when_to_use_negative:
- Input is raw RNA-Seq or ChIP-Seq data — use curatedAdipoRNA/curatedAdipoChIP count-based
  workflows instead of microarray intensity models
- The dataset has not been normalized across studies or batch-corrected (sva removal
  of data series and platform effects) — limma results on uncorrected probe intensities
  will be confounded
- Goal is probe-level analysis — the pipeline collapses probe intensities to gene
  symbols, so gene-level inference is the intended unit
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: curatedAdipoArray
  role: Source of the curated microarray dataset and processed SummarizedExperiment
    objects for downstream differential expression
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: limma
  role: Log-transform and normalize gene intensities across studies and fit the linear
    model for the knockdown-vs-control contrast
- name: sva
  role: Remove known batch effects (data series and platform) from the gene intensities
    before modeling
- name: ExperimentHub
  role: Access to the clean and the processed data SummarizedExperiment objects
- name: SummarizedExperiment
  role: Container format for the processed datasets (probe/gene intensities, colData
    metadata)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:3f4562425626a777422882869eaf6fdead7a66d07cdceca647cafe800527535f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - ExperimentHub
  - R
  - R/Bioconductor
  - SummarizedExperiment
  - curatedAdipoArray
  - limma
  - sva
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# microarray-gene-expression-analysis

## Summary

Differential expression analysis on curated MDI-induced 3T3-L1 microarray data using limma linear models on normalized, batch-corrected gene intensities from SummarizedExperiment objects distributed via curatedAdipoArray/ExperimentHub.

## When to use

Use when the goal is differential gene expression (e.g. knockdown-vs-control contrasts) on MDI-induced 3T3-L1 microarray samples with curated, normalized gene-level intensities available as a SummarizedExperiment object.

## When NOT to use

- Input is raw RNA-Seq or ChIP-Seq data — use curatedAdipoRNA/curatedAdipoChIP count-based workflows instead of microarray intensity models
- The dataset has not been normalized across studies or batch-corrected (sva removal of data series and platform effects) — limma results on uncorrected probe intensities will be confounded
- Goal is probe-level analysis — the pipeline collapses probe intensities to gene symbols, so gene-level inference is the intended unit

## Inputs

- Processed SummarizedExperiment object of gene intensities from curatedAdipoArray (accessed via ExperimentHub)
- Sample metadata (colData) classifying samples as none/genetically/pharmacologically modified
- Optional gene set (e.g. 48 validated PPARG target genes) for recovery checks

## Outputs

- Differential expression table with log2 fold-change, moderated p-value, and Benjamini-Hochberg FDR per gene (e.g. pparg_kd_differential_expression.csv)
- Filtered list of genes down-regulated by >= 1 log2 fold in the knockdown contrast

## How to apply

Load the processed SummarizedExperiment object from the curatedAdipoArray package through ExperimentHub and subset colData into the perturbation group (e.g. Pparg-knockdown, n=27) and the MDI-induced control group (n=30). Fit a limma linear model on the normalized, batch-corrected gene intensities (previously log-transformed, normalized across studies with limma, and corrected for data series and platform batch effects with sva) for the knockdown-vs-control contrast. Extract per-gene log2 fold-change, moderated p-value, and Benjamini-Hochberg FDR. Filter for genes down-regulated by one or more log2 fold and check recovery of known adipogenesis markers (Lpl, Cidec, Cd36) and validated PPARG targets. Write the full differential expression table to CSV.

## Related tools

- **curatedAdipoArray** (Source of the curated microarray dataset and processed SummarizedExperiment objects for downstream differential expression) — https://github.com/MahShaaban/curatedAdipoArray
- **limma** (Log-transform and normalize gene intensities across studies and fit the linear model for the knockdown-vs-control contrast)
- **sva** (Remove known batch effects (data series and platform) from the gene intensities before modeling)
- **ExperimentHub** (Access to the clean and the processed data SummarizedExperiment objects)
- **SummarizedExperiment** (Container format for the processed datasets (probe/gene intensities, colData metadata))

## Examples

```
BiocManager::install("curatedAdipoArray"); se <- curatedAdipoArray::adipose2018(); fit <- limma::lmFit(assay(se)[, se$group %in% c("pparg_kd","control")], model.matrix(~group)); fit <- limma::eBayes(fit); topTable(fit)
```

## Evaluation signals

- Lpl, Cidec and Cd36 are down-regulated with |log2FC| >= 1 in the Pparg-knockdown vs MDI-induced control contrast
- 29 of 48 previously validated PPARG target genes are recovered among significant down-regulated genes
- FDR values are Benjamini-Hochberg adjusted moderated p-values from the limma model, not raw p-values
- The analysis operates on gene-level (not probe-level) intensities after collapseRows probe-to-symbol mapping
- Batch effects (data series and platform) are absent from the modeled intensities (sva correction applied upstream)

## Limitations

- Two published studies/datasets were excluded for missing probe intensities and probe annotations, so not all GEO 3T3-L1 studies can be analyzed
- Batch effects could only be removed using a subset of the data; residual confounding across data series and platforms may persist
- Microarray data support differential expression and gene set enrichment, but not transcript-level or splice-aware analyses

## Evidence

- [results] In Pparg-knockdown cells, key lipid metabolism genes Lpl, Cidec and Cd36 were down-regulated by one or more log2 fold relative to controls: [evidence span withheld — anchor status: not_found]
- [intro] missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`.: "the gene intensities were log transformed and normalized across studies using `limma`."
- [intro] the known batch effects (data series and platform) were removed using `sva`.: "the known batch effects (data series and platform) were removed using `sva`."
- [intro] Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`.: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
- [readme] The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment.: "The data contained in the package can be used in any number of downstream
analysis such as differential expression and gene set enrichment."
