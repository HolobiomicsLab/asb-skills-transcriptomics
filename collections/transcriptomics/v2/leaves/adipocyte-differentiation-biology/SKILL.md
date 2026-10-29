---
name: adipocyte-differentiation-biology
description: Use when the analysis goal is differential gene expression of genetically
  or pharmacologically perturbed MDI-induced 3T3-L1 adipocytes — e.g. Pparg knockdown
  vs control — or gene set enrichment over differentiation time points.
when_to_use_negative:
- The input is raw RNA-Seq read counts for 3T3-L1 differentiation — use the curatedAdipoRNA
  RangedSummarizedExperiment instead of the microarray curatedAdipoArray data
- The question concerns ChIP-Seq peak binding of transcription factors or histone
  modifications — use the curatedAdipoChIP peak_counts object
- The input dataset lacks probe-to-gene-symbol annotation or probe intensities (studies
  missing these were excluded during curation)
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: curatedAdipoArray
  role: Bioconductor experimental data package supplying the curated, batch-corrected
    3T3-L1 microarray SummarizedExperiment data used for differential expression
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: ExperimentHub
  role: Distribution channel through which the clean and processed SummarizedExperiment
    objects are accessed
- name: limma
  role: Fits the linear model for the knockdown-vs-control contrast; also used for
    log transformation and cross-study normalization of gene intensities
- name: SummarizedExperiment
  role: Container format of the processed datasets (intensities plus colData metadata)
    that is subset into the two contrast groups
- name: BiocManager
  role: Installs the curatedAdipoArray package from Bioconductor
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:26e2b4631ebfea97d828798ea2ed0d5cd198a85c575909bef560002dd3b3ef7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - BiocManager
  - ExperimentHub
  - R
  - R/Bioconductor
  - SummarizedExperiment
  - curatedAdipoArray
  - limma
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# adipocyte-differentiation-biology

## Summary

Domain skill for reproducing adipocyte differentiation biology from the curated MDI-induced 3T3-L1 microarray resource (curatedAdipoArray), e.g. quantifying the Pparg-knockdown vs MDI-induced control contrast with limma and verifying that lipid metabolism genes (Lpl, Cidec, Cd36) are down-regulated by >= 1 log2 fold.

## When to use

Use when the analysis goal is differential gene expression of genetically or pharmacologically perturbed MDI-induced 3T3-L1 adipocytes — e.g. Pparg knockdown vs control — or gene set enrichment over differentiation time points.

## When NOT to use

- The input is raw RNA-Seq read counts for 3T3-L1 differentiation — use the curatedAdipoRNA RangedSummarizedExperiment instead of the microarray curatedAdipoArray data
- The question concerns ChIP-Seq peak binding of transcription factors or histone modifications — use the curatedAdipoChIP peak_counts object
- The input dataset lacks probe-to-gene-symbol annotation or probe intensities (studies missing these were excluded during curation)

## Inputs

- Processed gene-expression SummarizedExperiment object from the curatedAdipoArray Bioconductor package (accessed via ExperimentHub)
- Sample metadata (colData) classifying samples as genetically modified (Pparg knockdown, n=27) vs MDI-induced control (n=30)
- List of 48 previously validated PPARG target genes for validation of the down-regulated set

## Outputs

- Differential expression table (CSV) with per-gene log2 fold-change, moderated p-value and Benjamini-Hochberg FDR for the Pparg-knockdown vs control contrast
- Filtered list of genes down-regulated by >= 1 log2 fold in the knockdown
- Validation report of recovered PPARG targets (29/48 expected)

## How to apply

Load the processed gene-expression SummarizedExperiment object provided by curatedAdipoArray through ExperimentHub and subset samples into the Pparg-knockdown group (n=27) and the MDI-induced control group (n=30). Fit a limma linear model on the normalized, batch-corrected gene intensities (intensities were log transformed, normalized across studies with limma, and batch effects of data series and platform removed with sva) for the knockdown-vs-control contrast, extracting log2 fold-change, moderated p-value and Benjamini-Hochberg FDR per gene. Filter for genes down-regulated by one or more log2 fold in the knockdown and confirm the lipid metabolism genes Lpl, Cidec and Cd36 appear among them, consistent with failed adipocyte differentiation. As a biological validation, intersect the significant down-regulated list with the 48 previously validated PPARG target genes and confirm 29 are recovered. Write the full differential expression table (e.g. pparg_kd_differential_expression.csv) for downstream reuse.

## Related tools

- **curatedAdipoArray** (Bioconductor experimental data package supplying the curated, batch-corrected 3T3-L1 microarray SummarizedExperiment data used for differential expression) — https://github.com/MahShaaban/curatedAdipoArray
- **ExperimentHub** (Distribution channel through which the clean and processed SummarizedExperiment objects are accessed)
- **limma** (Fits the linear model for the knockdown-vs-control contrast; also used for log transformation and cross-study normalization of gene intensities)
- **SummarizedExperiment** (Container format of the processed datasets (intensities plus colData metadata) that is subset into the two contrast groups)
- **BiocManager** (Installs the curatedAdipoArray package from Bioconductor)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("curatedAdipoArray")
```

## Evaluation signals

- Lpl, Cidec and Cd36 are each down-regulated with |log2FC| >= 1 in the Pparg-knockdown vs control contrast
- 29 of the 48 previously validated PPARG target genes are recovered in the significant down-regulated gene list
- The output table contains log2 fold-change, moderated p-value and Benjamini-Hochberg FDR columns for every gene
- Group sizes match the curated metadata: 27 Pparg-knockdown and 30 MDI-induced control samples
- Differential expression results reproduce the published observation that batch effects were removed and biological findings reproduced using a subset of the data

## Limitations

- The resource covers microarray datasets of MDI-induced 3T3-L1 sampled at differentiation time points under genetic or pharmacological perturbation; only 43 published studies / 47 datasets were curated and two datasets were excluded for missing probe annotations
- Batch effects of data series and platform were removed with sva, so results depend on the quality of that correction; the processed data are gene-level collapsed intensities, not raw probes
- The package contains no R functions — it only distributes curated SummarizedExperiment objects and the inst/scripts used to generate them, so all analysis code must be supplied by the user

## Evidence

- [other] In Pparg-knockdown cells, key lipid metabolism genes Lpl, Cidec and Cd36 were down-regulated by one or more log2 fold relative to controls: [evidence span withheld — anchor status: not_found]
- [abstract] batch effects reproduced: "Finally, we used a subset of the data to effectively remove batch effects and reproduce biological observations."
- [intro] ExperimentHub access: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
- [readme] downstream use cases: "The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment."
- [intro] limma normalization: "Then the gene intensities were log transformed and normalized across studies using `limma`."
